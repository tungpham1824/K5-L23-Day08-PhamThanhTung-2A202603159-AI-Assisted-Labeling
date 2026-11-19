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

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.3,bicycle=0.2,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
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
- Vì sao lọt: img_028 và img_029 là ảnh dày xe máy: vội và bị ảnh hưởng bởi nhãn máy có sẵn nên bỏ qua khâu tự quét ảnh độc lập.
- Accepted 76.4% mà bắt bẫy chỉ 58.3%: máy đúng phần lớn nên dễ chấp nhận cả những box sai trông hợp lý, dẫn đến hiện tượng thiên kiến tự động hoá (automation bias).

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
| img_013.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 dương: `400,480`, `600,110`; 1 âm: `720,300`) | Mask bao mui và thân xe tuk-tuk, gồm kính chắn gió; điểm âm loại bỏ người lái bên phải | Viền răng cưa ở chắn bùn bánh trước; dễ lan sang người lái nếu thiếu điểm âm | `tuktuk_img013.jpg` |
| img_002.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 dương: `450,550`, `500,400`; 1 âm: `570,350`) | Mask bao thân xe và mui xe tuk-tuk xanh; điểm âm loại bỏ người ngồi trước buồng lái | Bóng đổ dưới gầm xe lan nhẹ xuống mặt đường; cần thêm click âm ở mặt đất để tách bóng | `tuktuk_img002.jpg` |
| img_004.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 dương: `520,330`, `480,450`; 1 âm: `420,250`) | Mask bao khung xích lô và giỏ hàng; điểm âm loại bỏ người đạp phía sau | Khe hở nan hoa bánh xe và khung giằng bị vỡ thành nhiều polygon nhỏ | `cyclo_img004.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2.1 (bám biên nhìn thấy) và 4.2.3 (bóng đổ không thuộc vật, kính xe thuộc vật) rất hữu ích để đặt click dương/âm. Quy tắc còn thiếu: chưa hướng dẫn cách làm mịn viền khi vật cơ khí có cấu trúc nan hoa, thanh giằng phức tạp tạo ra nhiều lỗ thủng nhỏ (< 5%).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Trên Docker local không có GPU, Interactor SAM chạy qua CPU có độ trễ khoảng 3-5 giây mỗi click, trong khi chạy script `sam2_points.py` trên Colab T4 GPU chỉ mất ~1.67 giây/ảnh. Tuy nhiên thao tác trực tiếp trên giao diện CVAT trực quan hơn nhờ phản hồi thị giác tức thì.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): `CAL-01` (xử lý lỗi máy YOLO26 nhầm tuk-tuk thành car/truck, thống nhất gán `three_wheeler` và tách `person` riêng khi thấy ≥ 20%) và `CAL-02` (chốt tiêu chí phân loại xe đạp điện có bàn đạp trợ lực vs xe máy điện thuần động cơ).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung định nghĩa xe điện (Mục 2); quy định xe 3 bánh có mui/kính vẫn là `three_wheeler` (Mục 2 & 4.1.4); giới hạn box xe khi chở hàng cồng kềnh nhô ra > 20% (Mục 4.1.5); chuẩn hoá tiêu chí phân định cụm xe dày đặc và box cá thể (Mục 4.1.6).
- Nguyên nhân gốc: Kết hợp cả guideline v1.0 mơ hồ (chưa có quy định xe điện, hàng nhô ra), giới hạn của máy (YOLO26 closed-set gọi tuk-tuk là car, YOLOE conf thấp tạo nhiều FP) và lỗi thao tác của người duyệt (vội nên lọt bẫy missing trong cụm xe đông).

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review`: img_003.jpg, img_005.jpg, img_035.jpg + 5 ảnh ngẫu nhiên: img_004.jpg, img_009.jpg, img_017.jpg, img_018.jpg, img_034.jpg, seed 42); người kiểm: Phạm Thanh Tùng (**hạn chế**: tự kiểm tra lại bài của chính mình nên có thể mang thiên kiến chủ quan).
- Lỗi theo **đối tượng**: 0/167 = 0.0% · theo **ảnh**: 0/8 = 0.0% (ngưỡng Release ≤ 2% trên mẫu kiểm).
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Trong mẫu kiểm 8 ảnh không ghi nhận lỗi sai sót lớn (0/8 ảnh lỗi). Tuy nhiên trong cả lô batch, các lỗi điển hình gồm: bỏ sót vật nhỏ trong cụm đông (img_028.jpg, img_029.jpg), box ôm hàng cồng kềnh hơi lỏng (img_003.jpg) và nhầm lẫn lớp xe ba bánh khi phụ thuộc máy (img_035.jpg).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên 95% (khoảng tin cậy Wilson) của tỉ lệ ảnh lỗi thật là **32%**. Ngưỡng 2% không thể được chứng minh chỉ bằng 8 ảnh mẫu nhỏ; cần lưu ý khoảng tin cậy này khi đánh giá chất lượng toàn bộ dataset.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script kiểm tra được tính hợp lệ của file JSON, schema taxonomy, bounding box trong toạ độ ảnh, các tỉ lệ accepted/edited/deleted. Script KHÔNG kiểm tra được độ ôm sát pixel thực tế của box, lỗi bỏ sót vật mà cả người và máy đều không nhìn thấy, hoặc việc reviewer có thực sự tuân thủ checklist 4 bước hay không.

**Quyết định: Rework**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release):
1. Guideline v1.1 đã chốt xong các câu hỏi mở (không bị Hold).
2. Tỉ lệ bắt bẫy chỉ đạt 7/12 = 58.3% < 80% (ngưỡng tối thiểu bắt buộc theo Label Contract); có hiện tượng thiên kiến tự động hoá rõ rệt khi lọt 2/3 bẫy missing ở cảnh đông xe máy (img_028, img_029) và bẫy wrong_class (img_035). Do đó lô nhãn chưa đủ độ tin cậy để Release và bắt buộc phải đưa ra quyết định **REWORK**.
Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại toàn bộ 24 ảnh batch, đặc biệt là 6 ảnh có mật độ xe máy dày đặc (img_005, img_018, img_021, img_024, img_028, img_029); thực hiện nghiêm ngặt bước ẩn nhãn máy để quét tìm vật sót trước khi bật nhãn máy; kiểm tra lại kích thước box chở hàng cồng kềnh theo guideline v1.1.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Có dùng Google Antigravity Agent để hỗ trợ chạy các script tự động hoá, kiểm tra định dạng dữ liệu, tính toán số đo thống kê và soạn nháp báo cáo. Tất cả số đo đều được đối chiếu trực tiếp từ file nhật ký thật (`model_log.csv`, `review_stats.md`, `runs.csv`). Không dùng AI để gán nhãn gold hay hỗ trợ nhìn ảnh khi gán gold.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không. 4 ảnh gold được gán tay từ đầu trong CVAT hoàn toàn không có AI hỗ trợ, trước khi chạy bất kỳ mô hình pre-label nào.
- Dùng cache `data/cache/` thay vì tự chạy? Không. Các mô hình YOLO26, YOLOE và SAM 2.1 được chạy trực tiếp trên GPU T4 của Google Colab theo tài liệu `docs/COLAB.md` (ghi nhận trong `runs.csv` với `device = cuda`).
- Lỗi / bug script đã gặp (nếu có): Môi trường WSL2 local không có GPU và không cài torch/ultralytics nên các script pre-label và SAM phải chạy trên Colab T4 theo kiến trúc hai đường (lane A) của bài lab.
- Một điều bạn làm khác lần sau: Sẽ phân bổ thời gian nghỉ ngơi hợp lý hơn giữa các lượt review (nghỉ 3-5 phút sau mỗi 8-10 ảnh) để tránh mỏi mắt và giảm thiên kiến tự động hoá, giúp bắt trọn vẹn 100% bẫy kiểm định.
