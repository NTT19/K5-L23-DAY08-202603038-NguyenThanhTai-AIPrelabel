# Báo cáo Day 08 — <<MSHV - Họ và tên>>

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

- Lệnh `merge_prelabels.py` chưa chạy. Part 4 dự kiến ghép `work/prelabel/yoloe_all/pre.coco.json` trước `work/prelabel/y26_all/pre.coco.json` với `--thr "three_wheeler=0.15" --conf 0.3 --iou 0.6`; truyền rõ `--conf 0.3` để các lớp không có ngưỡng riêng dùng đúng fallback nêu trong kết quả sweep (CLI của merge hiện có mặc định 0.25). Khi box chồng nhau với IoU ≥ 0.6, file YOLOE được ưu tiên trước YOLO26.
- **Phiên bản SAM chọn cho Part 5:** SAM 2.1 base (`sam2.1_b.pt`, run r05). Checkpoint chạy được trên CPU và tạo 16 mask ở 2 ảnh, 14.82 giây/ảnh; chưa so với bản khác nên đây là lựa chọn thực dụng, chưa phải kết luận bản tốt nhất.
- Lỗi **không sửa được bằng đổi ngưỡng:** YOLO26 không có lớp `three_wheeler`; tuk-tuk ở img_002 không được gán lớp này và xe ba bánh ở img_032 bị gọi `bicycle`. Dùng YOLOE cho lớp `three_wheeler`, sau đó merge và review nhãn.
- Box **điểm thấp mà có vẻ đúng:** ở img_032, YOLOE dự đoán `three_wheeler` 0.27 trên xe ba bánh phía phải; cần kiểm bằng guideline nhưng không nên loại chỉ vì score thấp (r04). Box ngưỡng cao cũng không tự bảo đảm đúng lớp.
- `conf` là điểm lọc của model, không phải phần trăm xác suất đúng: ở ngưỡng `three_wheeler=0.15`, sweep có 7 TP và 2 FP (precision 7/9 = 77.8%) dù recall là 7/7 = 100%.
- Kết quả Part 3 lấy từ `qc/threshold_sweep.md` và `qc/threshold_sweep.csv`: chỉ `three_wheeler` đạt recall mục tiêu. Với `person`, `motorbike`, `car`, dùng fallback conf 0.30 theo sweep và review thủ công vì lần lượt chỉ đạt recall tối đa 25/34, 13/22, 7/8 trong dải 0.10–0.50; `bicycle` chỉ có 2 vật gold; `bus` và `truck` không có vật gold. Muốn cải thiện ba lớp không đạt, cần kiểm tra ảnh của các vật gold bị bỏ sót và bổ sung/sửa nhãn trong lúc review hoặc thử nguồn model khác ở bước sau; không tự chọn ngưỡng ngoài sweep để tuyên bố đạt 90%.
- Giới hạn: sweep chỉ dùng 4 ảnh gold; cùng gold vừa chọn vừa chấm ngưỡng nên precision/recall trên gold có thể lạc quan. Ngưỡng của `three_wheeler` dựa trên 7 vật; `bicycle`, `bus`, `truck` quá ít hoặc không có mẫu nên không thể kết luận riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
