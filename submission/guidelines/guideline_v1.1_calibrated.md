# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603038 - Nguyen Thanh Tai

> Chỉ ghi phần bổ sung/diễn giải so với v1.0. Các quyết định dưới đây khớp với `submission/qc/calibration_decisions.csv`. Câu hỏi 1 còn mở; không suy đoán khi chưa có ví dụ phù hợp.

## 1. Câu hỏi mở đã chốt hoặc còn mở (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện hay xe máy điện nhỏ? | **Chưa chốt.** Trong các ca đã rà soát chưa có ảnh được xác nhận rõ để phân biệt xe đạp trợ lực với xe điện kiểu xe máy. Không suy đoán lớp; gắn `can_review` nếu gặp cho đến khi có ví dụ phù hợp. Đây là guideline gap còn lại. | Chưa có ví dụ đã xác nhận | — |
| 2 | Khối motorbike đỗ dày gần camera | Gán từng xe nếu phân biệt được cá thể; dùng `ignore_region` khi không tách được từng xe; trường hợp lưỡng lự quy tắc thì gắn `can_review`. Không đặt ngưỡng số học mới khi chưa có bằng chứng. | `img_006` | `CAL-04` |
| 3 | Hàng nhô ra khỏi xe: box và mask | Box giữ mục 4.1.5 của v1.0: loại hàng khi nhô hơn khoảng 20% chiều dài xe. Giỏ/thùng gắn cố định thuộc xe. Với mask, loại hàng rời nằm ngoài đường bao phương tiện; không loại giỏ/thùng gắn cố định. | `img_003` | `CAL-03` |
| 4 | Người ngồi trong xe ba bánh có mui | Giữ mốc nhìn thấy ≥20% làm quy tắc vận hành. Ca sát ngưỡng gắn `can_review`; bài này chỉ có một người rà soát nên chưa đo độ bất đồng giữa hai người. | `img_013` | `CAL-05` |
| 5 | Kính chắn gió/cửa sổ trong mask | Kính thuộc mask xe theo mục 4.2.3 v1.0; người nhìn thấy qua kính vẫn là instance `person` riêng. | `img_013` | `CAL-06` |
| 6 | Box chồng nhau trong cảnh tối | Không gộp hai phương tiện khác nhau chỉ vì box chồng. Chỉ khử trùng khi xác nhận các box cùng chỉ một phương tiện; nếu điều kiện ánh sáng khiến không xác định được thì gắn `can_review`. Quy tắc này chưa được kiểm chứng riêng trên ảnh ban đêm. | `img_006` (cảnh tối/dày; không kết luận là ảnh ban đêm) | `CAL-08` |
| 7 | Conf tối thiểu để chấp nhận box không sửa | Không dùng conf đơn lẻ để chấp nhận box. Conf chỉ giúp xếp thứ tự duyệt; mọi box vẫn phải đối chiếu ảnh và guideline. | `img_003` | `CAL-07` |

## 2. Ca khó mới từ lỗi máy và bẫy lọt

| Ảnh | Tình huống | Bằng chứng / quyết định | Quy tắc bổ sung |
|---|---|---|---|
| `img_003` | Nhiều box pre-label chồng lên cùng xe ba gác | Báo cáo Part 4 ghi hai box lớn có conf 0.90 và 0.80; nhãn sau review giữ một box. Log không gắn nguồn model cho riêng từng box nên không quy lỗi cho một model cụ thể. (`CAL-01`) | Một phương tiện vật lý là một instance. Chỉ bỏ box sau khi xác nhận trùng cùng vật; các phương tiện riêng vẫn là các instance riêng. Conf không quyết định box nào đúng. |
| `img_018` | Bẫy `missing` bị lọt | Nguyên nhân cá nhân chưa được xác định; không suy ra bước checklist nào đã bị bỏ. Kết quả bắt bẫy ban đầu là 8/12; 11/12 là sau khi danh sách bẫy lọt đã được xem và sửa. (`CAL-02`) | Lượt review sau quét toàn ảnh và đếm vật trước khi xem pre-label, rồi mới đối chiếu từng box. Ghi riêng kết quả ban đầu với kết quả sau khi lộ key. |
| `img_003` | Hàng rời trên xe ba gác | Chính sách box hiện có được giữ; mask phân biệt hàng rời với giỏ/thùng gắn cố định. (`CAL-03`) | Áp dụng quyết định ở câu hỏi mở 3, mục 1. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ khử box trùng, xử lý khối xe khó tách, giữ mốc người nhìn thấy 20% ở mức vận hành, chốt kính/conf và tách hàng rời khỏi mask; bổ sung bước quét toàn ảnh trước pre-label. Câu hỏi xe điện giữ mở vì chưa có ví dụ được xác nhận. | `CAL-01`–`CAL-08`; bẫy `img_018` |
