# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Tạm conf=0.25; chốt theo sweep Part 3 | r02 có 215 box ở 0.25; r03 còn 139 ở 0.5 nhưng giảm person 131→87 và motorbike 55→32. r06 chạy 0.05 trên cả 36 ảnh làm nguồn sweep. |
| three_wheeler | YOLOE-26s; prompt mặc định tuk-tuk / cyclo / cargo tricycle | Tạm conf=0.25; chốt theo sweep Part 3 | r04 tìm được xe ba gác ở img_002 (0.93) và xe ba bánh ở img_032 (0.48/0.92/0.27); r07 tạo 125 candidate toàn bộ 36 ảnh ở 0.05. |

- Lệnh `merge_prelabels.py`: chưa chạy; dự kiến ghép `work/prelabel/yoloe_all/pre.coco.json` với `work/prelabel/y26_all/pre.coco.json` sau khi hoàn tất sweep Part 3. YOLOE sẽ cung cấp `three_wheeler`; khi chồng với `car`/`truck`, quy tắc merge của script ưu tiên nhãn `three_wheeler`.
- **Phiên bản SAM chọn cho Part 5:** SAM 2.1 base (`sam2.1_b.pt`, run r05). Checkpoint chạy được trên CPU và tạo 16 mask ở 2 ảnh, 14.82 giây/ảnh; chưa so với bản khác nên đây là lựa chọn thực dụng, chưa phải kết luận bản tốt nhất.
- Lỗi **không sửa được bằng đổi ngưỡng:** YOLO26 không có lớp `three_wheeler`; tuk-tuk ở img_002 không được gán lớp này và xe ba bánh ở img_032 bị gọi `bicycle`. Dùng YOLOE cho lớp `three_wheeler`, sau đó merge và review nhãn.
- Box **điểm thấp mà có vẻ đúng:** ở img_032, YOLOE dự đoán `three_wheeler` 0.27 trên xe ba bánh phía phải; cần kiểm bằng guideline nhưng không nên loại chỉ vì score thấp (r04). Box ngưỡng cao cũng không tự bảo đảm đúng lớp.
- Giới hạn: chưa chạy `threshold_sweep.py`; ngưỡng 0.25 ở trên chỉ là baseline từ pilot. Part 3 sẽ chọn ngưỡng trên 4 ảnh gold; vì chọn và chấm trên cùng gold nên số đo máy trên gold có thể lạc quan và mẫu nhỏ.

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
