# Báo cáo Day 08 — 2A202603159 - Phạm Thanh Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.3 | Recall 90%, precision 55% (20 vật gold). Chấp nhận nhiều box thừa để không sót người; r01/r02 cho thấy 0.25 quá nhiễu ở đám đông, 0.5 mất vật nhỏ ở xa |
| bicycle | YOLO26m | 0.2 | Recall 100%, precision 71% (5 vật gold): mẫu rất nhỏ, kết luận yếu |
| motorbike | YOLO26m | 0.35 | Recall 92%, precision 100% (12 vật gold) |
| car | YOLO26m | 0.3 (mặc định) | Không ngưỡng ≤ 0.5 nào đạt recall 90% (tối đa 86%, 6/7 vật gold); precision tăng khi ngưỡng tăng nhưng recall không đổi |
| bus | YOLO26m | 0.3 (mặc định) | 0 vật trong gold: không chọn ngưỡng riêng |
| truck | YOLO26m | 0.3 (mặc định) | 1 vật trong gold (< 5): không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | Recall 100%, precision 78% (7 vật gold). YOLOE cho conf thấp nên ngưỡng thấp; r03 bắt tuk-tuk img_002 với conf 0.93, r06 cho thấy ở 0.05 có 125 box/36 ảnh nên cần lọc |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<python scripts\merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --thr "person=0.3,bicycle=0.2,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work\prelabel\merged_review\pre.coco.json>>
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`). Lý do: nằm trong bộ `core`, chạy được trên Colab T4 (r04: 1.67 giây/ảnh), mask bám mui và thân tuk-tuk img_013 khá sát. 
- Lỗi **không sửa được bằng đổi ngưỡng**:
  - Tuk-tuk bị gọi nhầm lớp: YOLO26 gọi tuk-tuk img_013 là `car 0.42` (r04), và không có box nào ôm tuk-tuk img_002 ở cả hai ngưỡng 0.25 và 0.5 (r01, r02). Hạ ngưỡng chỉ thêm box rác.
  - Cách sửa: dùng YOLOE (prompt `three_wheeler`) cho lớp này, gộp với YOLO26 bằng `merge_prelabels.py`, rồi sửa lớp `car` thành `three_wheeler` bằng tay khi review.
- Box **điểm cao mà sai** / **điểm thấp mà đúng**:
  - Điểm cao mà sai: xe bán tải trắng img_002 bị gọi `truck 0.73` (guideline v1.0 xếp bán tải vào `car`). 
  - Điểm thấp mà đúng: tuk-tuk img_013 có box đúng vị trí nhưng conf chỉ 0.42 và nhãn sai (`car`); three_wheeler đúng ở conf thấp nên ngưỡng phải hạ xuống 0.15.
- Giới hạn: ngưỡng chọn và chấm trên cùng 4 ảnh gold nên precision/recall của máy trên gold là con số lạc quan; lớp bicycle (5 vật), truck (1), bus (0) quá ít để kết luận.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 392 | 76.4% | 513 box máy |
| edited | 33 | 6.4% | 513 box máy |
| deleted | 84 | 16.4% | 513 box máy |
| added | 15 | 3.4% | 440 đối tượng sau duyệt |

- Lớp máy tệ nhất: three_wheeler. Trong 43 box máy (10 accepted, 8 edited, 25 deleted), 25 box bị xoá (~58%), do ngưỡng 0.15 thấp nên nhiều box thừa. person bị xoá nhiều nhất về số tuyệt đối (30 box). truck bị xoá 3 và thêm 3 nhưng chỉ 2 box accepted: mẫu quá ít để kết luận.
- So với gold: precision 66.2% (KTC 95%: 55–76%; TP 47, FP 24), recall 90.4% (KTC 95%: 79–96%; TP 47, FN 5), mẫu số 52 đối tượng gold trên 4 ảnh. Số này lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 7/12 = 58.3% (KTC 95%: 32–81%), dưới ngưỡng 80%. Loại lọt: loose 1/3 (img_006), extra 1/3 (img_027), wrong_class 1/3 (img_035), missing 2/3 (img_028, img_029). Chỉ bắt được 1/3 bẫy missing.
- Vì sao lọt: img_028 và img_029 là ảnh dày xe máy: vội
- Accepted 76.4% mà bắt bẫy chỉ 58.3%:  máy đúng phần lớn nên dễ chấp nhận cả những box sai trông hợp lý

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 12 phút / 4 ảnh gold = 3.0 phút/ảnh (~13 giây/đối tượng, 55 đối tượng); duyệt 85 phút / 24 ảnh batch = 3.54 phút/ảnh (~11 giây/đối tượng, 451 đối tượng). Theo ảnh, duyệt chậm hơn gán tay ~18%; theo đối tượng, duyệt nhanh hơn ~14%, vì ảnh batch dày hơn (18.8 so với 13.8 đối tượng/ảnh). Thiên lệch: tôi gán gold trước nên đã quen cảnh và guideline; gold chỉ 4 ảnh nên số phút/ảnh không ổn định; thời gian duyệt gồm cả việc nạp CVAT, sửa ignore_region và dừng nghỉ.
- Kết luận: chỉ đáng dùng cho person, motorbike, car (accepted cao, recall gold 90.4%), không đáng dùng nguyên trạng cho three_wheeler (58% box máy bị xoá ở ngưỡng 0.15, 25/43). Dựa trên: accepted 392/513 = 76.4%, nhưng thời gian duyệt/ảnh không thấp hơn gán tay, và bắt bẫy chỉ 7/12.
- **Ba quy tắc giảm thiên kiến của bạn:**
  1. Luôn ẩn nhãn máy và tự quét ảnh trước khi bật nhãn.
  2. Với ảnh dày xe máy, đặt giờ nghỉ riêng và duyệt lại một lượt riêng cho “vật thiếu”.
  3. Với lớp có precision thấp (three_wheeler, person): kiểm từng box theo quy tắc thay vì mặc định chấp nhận
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

- **Dùng AI để làm bài** "không dùng" 
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không 
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
