# IT209 - Session 05 - Bài tập 3: Xử lý Xung đột Phức tạp trong Quá trình Rebase

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Lớp / Học phần:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Đường dẫn nộp bài:** `homework/session_05/ex3/`

---

## Tóm tắt nội dung thực hiện

1. **Khởi tạo và tạo nhánh phân kỳ:**
   - Nhánh `main`: 2 commit mới (`update port on main`, `add env config`).
   - Nhánh `feature-api`: 2 commit mới (`feat: change port`, `feat: enable debug`).
2. **Khởi chạy Rebase:**
   - Lệnh: `git rebase main` trên nhánh `feature-api`.
3. **Xử lý xung đột Chặng 1 (Commit `feat: change port`):**
   - Sửa thủ công `config.json` giữ `"port": 9000` và `"env": "production"`.
   - Chạy `git add config.json` và `git rebase --continue`.
4. **Xử lý xung đột Chặng 2 (Commit `feat: enable debug`):**
   - Sửa thủ công `config.json` giữ `"port": 9000`, `"debug": true` và `"env": "production"`.
   - Chạy `git add config.json` và `git rebase --continue`.
5. **Kiểm tra và nghiệm thu:**
   - Lệnh `git log --graph --oneline` hiển thị đồ thị tuyến tính thẳng tắp 100%.

---

> 📄 **Xem báo cáo chi tiết đầy đủ:** [IT209_Session05_BT3_PhamThanhPhong_N24DTCN120.md](./IT209_Session05_BT3_PhamThanhPhong_N24DTCN120.md)
