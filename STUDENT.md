# Thông tin học viên

- Họ và tên: Nguyễn Thành Tài
- MSHV: 2A202603038
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: NTT19
- Repo bài nộp (URL): https://github.com/NTT19/K5-L23-DAY08-202603038-NguyenThanhTai-AIPrelabel (học viên xác nhận giảng viên cho phép tên hiện tại)
- Commit chốt bài (hash): sẽ điền sau commit Part 6
- Máy: Windows 10 Pro; GPU Intel Iris Xe Graphics; các lần inference đã ghi log chạy CPU
- CVAT (Docker): v2.74.1 (`cvat/server:v2.74.1`, kiểm tra image đang chạy)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 421 / 38 / 49 (mẫu số: 519 box máy); added: 62 (mẫu số: 521 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 42/74 = 56.8% / 42/73 = 57.5% (4 ảnh gold đã bị nhiễm; xem `submission/REPORT.md` mục 8)
- Bắt bẫy: 11/12 = 91.7% sau khi đã xem bẫy lọt; lượt đầu 8/12 = 66.7%
- Thời gian: gán tay 20/4 = 5 phút/ảnh; duyệt có pre-label 60/24 = 2.5 phút/ảnh
- Lỗi QC được ghi: 0/5 ảnh = 0.0%; 0/108 đối tượng = 0.0%. Mẫu nhỏ có cận trên Wilson 95% theo ảnh là 43%.
- **Quyết định:** HOLD — còn một câu hỏi xe điện trong guideline chưa chốt; cần làm rõ và đánh giá thêm.
