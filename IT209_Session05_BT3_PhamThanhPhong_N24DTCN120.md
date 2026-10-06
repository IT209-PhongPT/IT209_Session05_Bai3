# Báo cáo Bài tập 3: Xử lý Xung đột Phức tạp trong Quá trình Rebase

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 05 - Quản trị nâng cao & Khôi phục sự cố mã nguồn Git
- **Bài tập:** Bài tập 3 (ex3)

---

## 1. Mục tiêu & Bối cảnh bài toán

### 1.1. Bối cảnh (Context)
Trong môi trường phát triển phần mềm doanh nghiệp, khi một nhánh tính năng (`feature-api`) kéo dài nhiều ngày, nhánh chính (`main`) liên tục tiếp nhận các commit mới từ các thành viên khác trong nhóm. Để giữ cho lịch sử Git luôn phẳng và tuyến tính (**Linear History**), thay vì sử dụng `git merge` (sinh ra các commit gộp rác và đồ thị rẽ nhánh chằng chịt), nhóm quy định sử dụng `git rebase main`.

Tuy nhiên, nếu cả hai nhánh cùng chỉnh sửa tệp tin cấu hình `config.json` qua nhiều commit khác nhau, Git sẽ "phát lại" (replay) từng commit một lên đỉnh của nhánh `main`. Quá trình này sẽ làm phát sinh xung đột liên tiếp qua từng chặng commit đòi hỏi kỹ sư phải nắm vững quy trình xử lý xung đột từng bước (**Step-by-Step Conflict Resolution**).

### 1.2. Mục tiêu kỹ thuật
- Phân biệt sự khác biệt bản chất giữa giải quyết xung đột khi Merge (xử lý 1 lần duy nhất trên merge commit) và khi Rebase (xử lý lặp qua từng commit bị áp dụng lại).
- Làm chủ quy trình điều khiển luồng Rebase khi gặp xung đột: sửa conflict thủ công -> `git add <file>` -> `git rebase --continue` (hoặc `git rebase --abort` nếu muốn hủy an toàn).
- Hoàn thành quy trình rebase 2 chặng liên tiếp đối với tệp tin `config.json`.
- Đạt được lịch sử đồ thị Git thẳng tắp (`Linear Graph`) qua `git log --graph --oneline`.

### 1.3. Ràng buộc kỹ thuật
- Bắt buộc giải quyết xung đột thủ công ở từng chặng commit (Chặng 1: `feat: change port`, Chặng 2: `feat: enable debug`).
- Tuyệt đối không sử dụng lệnh merge phụ, không làm mất cấu hình `env: production` mới cập nhật của nhánh chính.
- Lịch sử Git sau cùng phải thẳng hàng, các commit của nhánh `feature-api` được gắn nối tiếp ngay trên commit mới nhất của `main`.

---

## 2. Cơ sở lý thuyết: So sánh Merge Conflict vs Rebase Conflict

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                     So sánh Giải quyết Xung đột: Merge vs Rebase                       │
├────────────────────────────────────────┬───────────────────────────────────────────────┤
│            Xung đột khi Merge          │             Xung đột khi Rebase               │
├────────────────────────────────────────┼───────────────────────────────────────────────┤
│ • Git so sánh toàn diện 1 lần giữa     │ • Git "bóc tách" từng commit của feature và   │
│   đỉnh của 2 nhánh (3-Way Merge).      │   "chạy lại" (replay) lần lượt lên đỉnh main. │
│ • Người dùng chỉ cần sửa xung đột      │ • Xung đột có thể xuất hiện LIÊN TIẾP ở từng  │
│   ĐÚNG 1 LẦN DUY NHẤT.                 │   commit trong danh sách rebase.              │
│ • Tạo ra một Merge Commit mới với 2 cha│ • KHÔNG TẠO merge commit, sinh ra các commit  │
│   (Đồ thị rẽ nhánh và hội tụ).         │   mới với mã băm mới (Đồ thị thẳng tắp).      │
└────────────────────────────────────────┴───────────────────────────────────────────────┘
```

---

## 3. Quy trình thực hiện chi tiết

### Bước 1: Khởi tạo và Commit ban đầu trên nhánh `main`
```bash
git init -b main
git config --local user.name "Pham Thanh Phong"
git config --local user.email "thanhphong102006@gmail.com"

# Tạo tệp cấu hình config.json ban đầu
cat << 'EOF' > config.json
{
  "port": 8080,
  "debug": false
}
EOF

git add config.json
git commit -m "init config"
```

---

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit
```bash
git checkout -b feature-api

# Commit 1: Đổi cổng sang 9000
# Sửa "port": 9000
git add config.json
git commit -m "feat: change port"

# Commit 2: Kích hoạt chế độ debug
# Sửa "debug": true
git add config.json
git commit -m "feat: enable debug"
```

---

### Bước 3: Chuyển về nhánh `main` và thực hiện 2 commit mới
```bash
git checkout main

# Commit 1 trên main: Đổi cổng sang 8081
# Sửa "port": 8081
git add config.json
git commit -m "update port on main"

# Commit 2 trên main: Bổ sung môi trường production
# Thêm "env": "production"
git add config.json
git commit -m "add env config"
```

Kiểm tra đồ thị phân nhánh trước khi Rebase:
```bash
git log --graph --oneline --all
```
```text
* 61c3d66 (main) add env config
* 2c5406a update port on main
| * ed206b7 (HEAD -> feature-api) feat: enable debug
| * 77f324e feat: change port
|/  
* a35bcbc init config
```

---

### Bước 4: Khởi chạy Rebase và Xử lý xung đột Chặng 1 (`feat: change port`)
Chuyển về nhánh `feature-api` và thực hiện rebase:
```bash
git checkout feature-api
git rebase main
```
*Git thông báo xung đột tại commit 1:*
```text
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
Rebasing (1/2) error: could not apply 77f324e... feat: change port
```

**Thẻ xung đột trong `config.json`:**
```json
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> 77f324e (feat: change port)
}
```

**Cách giải quyết:** Giữ lại cổng `9000` của feature đồng thời tích hợp trường `"env": "production"` từ `main`:
```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

**Lệnh xác nhận và tiếp tục:**
```bash
git add config.json
git rebase --continue
```

---

### Bước 5: Xử lý xung đột Chặng 2 (`feat: enable debug`)
*Git áp dụng commit thứ 2 và thông báo xung đột tiếp theo:*
```text
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
Rebasing (2/2) error: could not apply ed206b7... feat: enable debug
```

**Thẻ xung đột trong `config.json`:**
```json
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> ed206b7 (feat: enable debug)
}
```

**Cách giải quyết:** Giữ nguyên cổng `9000`, kích hoạt `"debug": true` và duy trì `"env": "production"`:
```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

**Lệnh hoàn tất Rebase:**
```bash
git add config.json
git rebase --continue
```
*Git phản hồi thành công:* `Successfully rebased and updated refs/heads/feature-api.`

---

## 4. Nhật ký thực thi & Minh chứng nghiệm thu (Verification Logs)

### 4.1. Minh chứng lịch sử đồ thị tuyến tính thẳng hàng (`git log --graph --oneline`)
```bash
git log --graph --oneline
```
**Output thực tế:**
```text
* 52b49fc feat: enable debug
* 9d21a96 feat: change port
* 61c3d66 add env config
* 2c5406a update port on main
* a35bcbc init config
```
*(Lịch sử hoàn toàn thẳng hàng, không có phân nhánh và không có merge commit rác).*

---

### 4.2. Minh chứng trạng thái sạch của Working Tree (`git status`)
```bash
git status
```
**Output thực tế:**
```text
On branch feature-api
nothing to commit, working tree clean
```

---

### 4.3. Minh chứng nội dung hợp nhất hoàn chỉnh của `config.json`
```bash
cat config.json
```
**Output thực tế:**
```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```
*(Bảo toàn đầy đủ tất cả cấu hình của cả nhánh main lẫn tính năng nhánh feature-api).*

---

## 5. Hướng dẫn nộp bài & Cấu trúc kho lưu trữ

- **Đường dẫn trên GitHub:** `homework/session_05/ex3/`
- **Tệp báo cáo:** `README.md` (hoặc `IT209_Session05_BT3_PhamThanhPhong_N24DTCN120.md`)
- **Cấu trúc nộp bài chuẩn:**
```text
homework/session_05/ex3/
├── .git/
├── config.json
├── IT209_Session05_BT3_PhamThanhPhong_N24DTCN120.md
└── README.md
```

---

## 6. Tổng kết & Kinh nghiệm quản trị Rebase

1. **Kiểm soát từng chặng commit:** Khi Rebase qua nhiều commit xung đột, hãy bình tĩnh đọc kỹ thẻ conflict markers ở từng bước, không vội vàng bỏ qua (`--skip`) nếu chưa chắc chắn.
2. **Kỹ năng thoát hiểm an toàn:** Nếu phát hiện thao tác giải quyết xung đột bị sai hoặc nhầm lẫn, luôn có thể chạy `git rebase --abort` để đưa repository trở về trạng thái nguyên vẹn trước khi bắt đầu rebase.
3. **Giá trị của Linear History:** Lịch sử Git thẳng hàng giúp việc tìm lỗi tự động bằng `git bisect`, tạo Release Notes và theo dõi dòng chảy phát triển phần mềm trở nên cực kỳ trong sáng và hiệu quả.
