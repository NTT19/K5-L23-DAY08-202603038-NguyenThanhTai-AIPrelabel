# Báo cáo Day 08 — 2A202603038 - Nguyễn Thành Tài

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | Fallback conf=0.30 (truyền rõ ở Part 4) | Không ngưỡng 0.10–0.50 đạt recall 90%; cao nhất 25/34 = 73.5% ở 0.10 (precision 25/53 = 47.2%). Cần review các vật bị bỏ sót. |
| bicycle | YOLO26m | Fallback conf=0.30 (truyền rõ ở Part 4) | Chỉ có 2 vật gold, dưới mức 5 để chọn ngưỡng riêng; không kết luận ngưỡng từ mẫu này. |
| motorbike | YOLO26m | Fallback conf=0.30 (truyền rõ ở Part 4) | Không ngưỡng 0.10–0.50 đạt recall 90%; cao nhất 13/22 = 59.1% ở 0.10 (precision 13/31 = 41.9%), còn 9/22 vật gold chưa match. |
| car | YOLO26m | Fallback conf=0.30 (truyền rõ ở Part 4) | Recall tối đa 7/8 = 87.5%, vẫn thiếu mục tiêu 90%; một vật gold không được match ở mọi ngưỡng 0.10–0.50. |
| bus / truck | YOLO26m | Fallback conf=0.30 (truyền rõ ở Part 4) | Không có vật gold của hai lớp trong 4 ảnh; chưa thể ước lượng recall hay chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s; prompt `tuk-tuk` / `cyclo` / `cargo tricycle` | 0.15 | Đạt recall 7/7 = 100% với precision 7/9 = 77.8% (2 box thừa); đây là gợi ý của sweep trên 7 vật gold. |

- Part 4 đã có `work/prelabel/merged_review/pre.coco.json` với 28 ảnh và 606 box; metadata ghi nguồn YOLOE trước YOLO26. Tái tạo phép gộp từ hai nguồn đó với `--thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold"` cho danh sách ảnh và annotation giống tệp đã có. `--conf 0.3` là fallback cho các lớp không có ngưỡng riêng; khi box chồng nhau với IoU ≥ 0.6, file YOLOE được ưu tiên trước YOLO26.
- **Phiên bản SAM chọn cho Part 5:** SAM 2.1 base (`sam2.1_b.pt`, run r05). Checkpoint chạy được trên CPU và tạo 16 mask ở 2 ảnh, 14.82 giây/ảnh; chưa so với bản khác nên đây là lựa chọn thực dụng, chưa phải kết luận bản tốt nhất.
- Lỗi **không sửa được bằng đổi ngưỡng:** YOLO26 không có lớp `three_wheeler`; tuk-tuk ở img_002 không được gán lớp này và xe ba bánh ở img_032 bị gọi `bicycle`. Dùng YOLOE cho lớp `three_wheeler`, sau đó merge và review nhãn.
- Box **điểm thấp mà có vẻ đúng:** ở img_032, YOLOE dự đoán `three_wheeler` 0.27 trên xe ba bánh phía phải; cần kiểm bằng guideline nhưng không nên loại chỉ vì score thấp (r04). Box ngưỡng cao cũng không tự bảo đảm đúng lớp.
- `conf` là điểm lọc của model, không phải phần trăm xác suất đúng: ở ngưỡng `three_wheeler=0.15`, sweep có 7 TP và 2 FP (precision 7/9 = 77.8%) dù recall là 7/7 = 100%.
- Kết quả Part 3 lấy từ `qc/threshold_sweep.md` và `qc/threshold_sweep.csv`: chỉ `three_wheeler` đạt recall mục tiêu. Với `person`, `motorbike`, `car`, dùng fallback conf 0.30 theo sweep và review thủ công vì lần lượt chỉ đạt recall tối đa 25/34, 13/22, 7/8 trong dải 0.10–0.50; `bicycle` chỉ có 2 vật gold; `bus` và `truck` không có vật gold. Muốn cải thiện ba lớp không đạt, cần kiểm tra ảnh của các vật gold bị bỏ sót và bổ sung/sửa nhãn trong lúc review hoặc thử nguồn model khác ở bước sau; không tự chọn ngưỡng ngoài sweep để tuyên bố đạt 90%.
- Giới hạn: sweep chỉ dùng 4 ảnh gold; cùng gold vừa chọn vừa chấm ngưỡng nên precision/recall trên gold có thể lạc quan. Ngoài ra, học viên xác nhận đã xem pre-label của ảnh gold trước khi xuất `gold_manual.json`; do đó gold bị nhiễm và số đo trên bộ này không phải đánh giá độc lập. Ngưỡng của `three_wheeler` dựa trên 7 vật; `bicycle`, `bus`, `truck` quá ít hoặc không có mẫu nên không thể kết luận riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 421 | 81.1% | 519 box máy |
| edited | 38 | 7.3% | 519 box máy |
| deleted | 49 | 9.4% | 519 box máy |
| added | 62 | 11.9% | 521 đối tượng sau duyệt |

- Trong `deleted + added`, script nhận diện 12 trường hợp đổi lớp; ước tính số vật máy bỏ sót thật là `62 - 12 = 50` (`review_stats.md`). Số đo loại 23 đối tượng quanh bẫy khỏi profile.
- Ba trạng thái accepted/edited/deleted cộng lại là 508/519 box máy. Theo cách đếm của `review_stats.py`, 11 box máy còn lại không được ghép và được `covered_by` coi là nằm trong `ignore_region`; chúng vẫn thuộc mẫu số 519 nhưng không tính là deleted. Vì vậy ba tỷ lệ không cộng thành 100%.
- `three_wheeler` có số box máy bị xóa cao nhất: 21/40 box máy của lớp này (12 accepted + 7 edited + 21 deleted). Ở img_003, pre-label có nhiều box chồng lên cùng xe ba gác xanh; hai box lớn có điểm 0.90 và 0.80, trong khi nhãn sau review giữ một box. Đây là ví dụ box trùng/lỏng; tổng deleted cũng gồm trường hợp đổi lớp theo cách đếm của script.
- So với gold: precision của **máy** `42/74 = 56.8%`; recall `42/73 = 57.5%` (73 đối tượng gold trên 4 ảnh; KTC 95% lần lượt 45–67% và 46–68%, từ `review_stats.md`). Gold bị nhiễm theo xác nhận ở mục 8 và được dùng để chọn ngưỡng, nên các số này không phải đánh giá độc lập.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Output hiện tại ghi bắt được `11/12 = 91.7%` (loose 3/3, wrong_class 3/3, extra 3/3, missing 2/3); bẫy còn lọt là `missing` tại img_018. KTC 95% là 65–99% do chỉ có 12 bẫy (`review_stats.md`).
- Lần thống kê trước trong lượt review ban đầu ghi `8/12 = 66.7%`; sau khi danh sách bẫy lọt đã được xem, nhãn được cập nhật và output hiện tại tăng lên 11/12. Vì vậy 11/12 là kết quả sau khắc phục, không phải phép đo mù ban đầu. Lý do cá nhân khiến bẫy `missing` ở img_018 lọt không có trong output; không thể suy ra checklist nào đã bị bỏ chỉ từ số liệu.
- `accepted` hiện là 421/519 = 81.1%, nhưng số này chỉ đo mức khớp của box máy còn lại; nó không đo việc người review có phát hiện vật máy bỏ sót hay bẫy cài vào ảnh. Kết quả lượt đầu 8/12 cho thấy tỷ lệ accepted cao không đủ để kết luận quy trình review đã bắt lỗi tốt.

## 4. Máy có đáng dùng không?

- Gán tay: `20/4 = 5 phút/ảnh`; review batch: `60/24 = 2.5 phút/ảnh` (`time_log.csv`), thấp hơn 50% theo số thời gian đã ghi. Đây không phải so sánh có kiểm soát: 4 ảnh gold được gán trước nên đã quen kiểu cảnh, và batch gồm ảnh khác.
- Kết luận hiện tại: thời gian ghi nhận gợi ý pre-label có thể giúp review nhanh hơn, nhưng không thể quy toàn bộ chênh lệch cho pre-label vì mẫu ảnh khác nhau và có thiên lệch quen cảnh. Vẫn cần review thủ công: accepted 421/519 = 81.1%, added 62/521 = 11.9%, còn precision/recall trên gold là 56.8%/57.5% (4 ảnh).
- **Ba quy tắc cho lượt review tiếp theo** (có thể kiểm tra bằng hành động):
  1. Ẩn toàn bộ nhãn máy và tự quét/đếm vật trước khi hiện nhãn ở cả 24/24 ảnh.
  2. Mỗi ảnh duyệt đủ bốn bước theo thứ tự: bỏ sót → thừa → sai lớp → box lỏng; ảnh đông quét theo góc phần tư.
  3. Nghỉ 3–5 phút sau khoảng 10 ảnh; ghi giờ bắt đầu/kết thúc và break thực tế vào time log.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013` | SAM 2.1 Base (`sam2.1_b.pt`) | 8 (3 dương, 5 âm) | Bản 3 click đầu còn dính người lái; các điểm âm được thêm để giảm vùng dính. Không thấy hàng hóa tách riêng để đánh giá. | Bản 7 click bỏ sót phần bánh xe; bản 8 click thêm điểm dương trên bánh, SAM trả 2 polygon và 129 933 px (18,6% ảnh). Học viên đã xem và chọn bản 8 click là tốt hơn. Chưa có mask chuẩn để đo IoU hoặc khẳng định sai số biên. | `tuktuk_img013_eight_clicks.jpg` |
| `img_004` | SAM 2.1 Base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Không thấy người lái trên tuk-tuk được chọn; người đứng bên phải được loại. Không xác định được hàng hóa riêng trong khoang. | Bản đầu có điểm dương trên mặt đường và mask tràn dưới bánh trước; bản tinh chỉnh đổi điểm dương vào bánh, thêm điểm âm trên đường. Còn nhiễu nhỏ ở mép trên trái; 1 polygon, 91 779 px (13,1% ảnh). | `tuktuk_img004_refined.jpg` |
| `img_032` | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Người đứng cạnh xích lô đã được loại; vùng khoang/vải phủ vẫn nằm trong mask. Không kết luận vùng đó là hàng hóa. | Mask theo được khoang và bánh xe; các thanh khung mảnh chưa thật đều; 1 polygon, 50 327 px (8,5% ảnh). | `cyclo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc bám biên phần nhìn thấy và tách vật chạm nhau không chồng lấn giúp xác định phạm vi mask. Ở `img_013`, một điểm âm trên thân người chưa đủ; thêm điểm âm trên đầu/chân người và người phía sau giúp loại các vùng đó. Câu hỏi về hàng hóa nhô ra vẫn chưa được chốt ở câu hỏi mở 3, mục 6 của guideline v1.0; vì vậy không suy rộng quy tắc hàng hóa từ ba ảnh này.
- Các file `tuktuk_img013.jpg` và `tuktuk_img004.jpg` là lần chạy đầu; `tuktuk_img013_refined.jpg` là bản 7 click. Các file này được giữ lại để đối chiếu lỗi với bản tinh chỉnh đã chọn.
- Interactor SAM trong CVAT: chưa thử; Part 5 dùng `sam2_points.py` offline nên chưa có so sánh trực tiếp.

## 6. Calibration (Part 6)

- Số quyết định: 8 (`calibration_decisions.csv`); quan trọng nhất là `CAL-03`: giữ ngưỡng box hàng hóa >~20% của v1.0, nhưng mask loại hàng rời ở ngoài đường bao xe; giỏ/thùng gắn cố định thuộc xe.
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: một phương tiện vật lý là một instance; quét toàn ảnh trước khi xem pre-label; khi xe khó tách, dùng `ignore_region` chỉ khi không tách được cá thể và dùng `can_review` khi chưa chắc quy tắc; không tự chấp nhận box chỉ dựa trên conf.
- Nguyên nhân gốc: box trùng ở `img_003` là lỗi pre-label quan sát được. Nguyên nhân bẫy `missing` ở `img_018` chưa xác định nên không quy cho bước checklist cụ thể. Câu hỏi xe đạp điện/xe máy điện vẫn mở vì chưa có ví dụ được xác nhận; mốc người nhìn thấy 20% được giữ tạm nhưng chưa có đo đồng thuận giữa hai người.
- Còn 1 guideline gap (câu hỏi xe điện #1); nếu chưa có ví dụ xác nhận thêm, chạy `qc_report.py --guideline-gap 1` và giữ quyết định HOLD theo cây quyết định.

## 7. QC và quyết định (Part 6)

- Mẫu: 5/24 ảnh batch (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), 108 đối tượng không tính `ignore_region`; học viên Nguyễn Thành Tài xác nhận đã kiểm 5 ảnh và không phát hiện lỗi. Hạn chế: người QC cũng là người duyệt lô.
- Lỗi được ghi theo **đối tượng**: 0/108 = 0.0%; theo **ảnh**: 0/5 = 0.0% (`qc_report_auto.md`; ngưỡng Release theo ảnh ≤ 2%).
- Không có loại lỗi nào được ghi trong phiếu QC, nên không có ví dụ lỗi để lập top. Đây chỉ là kết quả trên 5 ảnh đã chọn.
- **Giới hạn mẫu nhỏ:** với 0/5 ảnh ghi nhận lỗi, cận trên Wilson 95% cho tỷ lệ ảnh lỗi là 43% (`qc_report_auto.md`); mẫu này không chứng minh tỷ lệ lỗi thực tế ≤ 2%.
- Script đếm kết quả theo phiếu và kiểm định dạng nhãn; script không tự xác minh box ôm sát vật, vật bị cả máy lẫn người bỏ sót, hay người review đã làm đủ checklist.

**Quyết định của học viên: HOLD**, ngày 2026-11-18. Lý do: còn một câu hỏi mở về phân loại xe điện; `qc_report.py --guideline-gap 1` cũng gợi ý HOLD. Không dùng 0/5 ảnh lỗi để khẳng định chất lượng toàn bộ lô đạt ngưỡng 2%.

Việc cần làm tiếp: làm rõ câu hỏi xe điện bằng trường hợp có thể xác nhận, cập nhật quyết định calibration/guideline và rà lại nhãn bị ảnh hưởng nếu có; sau đó xây lại release và đánh giá QC theo hướng dẫn. Lưu ý kết quả bắt bẫy 11/12 là sau khắc phục và sau khi key đã được xem; lượt đầu là 8/12.

## 8. Ghi chú cá nhân

- **Dùng AI:** YOLO26m và YOLOE-26s tạo pre-label; SAM 2.1 tạo mask Part 5; Codex hỗ trợ đọc yêu cầu, chạy lệnh. Trong phiên làm việc trước, Codex đã mở ảnh gốc và overlay để nhận xét Part 5 và gợi ý điểm cần kiểm QC.
- **Gold bị nhiễm:** học viên xác nhận đã xem pre-label của ảnh gold trước khi xuất `gold_manual.json`. Các chỉ số so với gold và ngưỡng chọn từ gold cần được hiểu theo giới hạn này.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 10 Pro; GPU Intel Iris Xe Graphics; các lần inference có log dùng CPU. CVAT Docker đang chạy image `cvat/server:v2.74.1`.
- Dùng cache `data/cache/` thay vì tự chạy? Model log có các lần chạy thực tế; chưa thấy ghi nhận dùng cache. Xác nhận nếu lịch sử thao tác khác.
- Lỗi / bug script đã gặp (nếu có): chưa ghi nhận lỗi chạy script ở Part 6; một số mask ban đầu cần thêm click để giảm vùng dính người/nền (đã mô tả ở mục 5).
- Một điều bạn làm khác lần sau: quét và đếm toàn ảnh trước khi mở pre-label, sau đó kiểm từng box theo checklist; nguyên nhân bẫy `img_018` vẫn chưa xác định.
