# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603159 - Phạm Thanh Tùng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện: `bicycle` hay `motorbike`? | Xe 2 bánh có bàn đạp (pedals) trợ lực phân loại là `bicycle`. Xe 2 bánh chạy điện không có bàn đạp (xe máy điện, xe scooter điện) phân loại là `motorbike`. | img_004.jpg, img_018.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Nếu xe đỗ ken sát nhau bị che khuất > 80% diện tích hoặc chiều cao < 20 px thì gom thành khối `ignore_region`. Các xe đỗ phía trước nhìn thấy ≥ 20% thân xe vẫn phải gán box `motorbike` riêng. | img_005.jpg, img_028.jpg | CAL-04 |
| 3 | Hàng hoá nhô ra trên xe: box xe có gồm hàng không? | Box phương tiện chỉ bao quanh khung xe và giỏ/giá chở hàng gắn cố định vào thân xe. Phần hàng hoá rời (sọt hoa quả, bao tải, cọc cồng kềnh) nhô ra vượt quá 20% kích thước xe KHÔNG tính vào box xe. | img_003.jpg, img_035.jpg | CAL-03 |
| 4 | Kính chắn gió / cửa sổ xe ba bánh: thuộc mask xe hay không? | Mask của xe bao gồm toàn bộ khung xe và kính chắn gió (kính là bộ phận cấu thành phương tiện). Người ngồi bên trong nhìn xuyên qua kính vẫn được gán box `person` riêng biệt. | img_013.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg, img_002.jpg | Tuk-tuk có khung mui lái bị nhận nhầm | YOLO26 gọi tuk-tuk là `car 0.42` (img_013) hoặc bỏ sót (img_002) do tập COCO không có lớp xe ba bánh. | Sửa tất cả các phương tiện ba bánh chở khách/hàng thành `three_wheeler`. Gán `person` riêng cho người lái/hành khách nếu thấy ≥ 20%. | Mục 2 & 4.1.4: Mọi xe ba bánh có mui/kính vẫn là `three_wheeler`, không gán `car` hay `truck`. |
| img_004.jpg | Xe đạp điện nhỏ dựng ven đường | YOLO26 gán `bicycle 0.68`, người duyệt phân vân với xe máy điện nhỏ. | Kiểm tra có bàn đạp trợ lực hay không; xe có pedal xếp vào `bicycle`. | Mục 2: Định nghĩa lớp `bicycle` và `motorbike` đối với phương tiện chạy điện. |
| img_003.jpg, img_035.jpg | Xe máy chở sọt hàng to nhô hẳn ra ngoài | Máy ôm cả sọt hàng cồng kềnh tạo box rất lỏng hoặc gán sai lớp. | Thu hẹp box chỉ ôm khung xe máy và giá chở hàng gắn liền xe; bỏ phần hàng nhô > 20%. | Mục 4.1.5: Giới hạn box xe khi chở hàng cồng kềnh. |
| img_028.jpg, img_029.jpg | Bẫy missing lọt ở đám đông xe máy dày đặc | Bẫy xoá mất 1 box xe máy nhỏ bị che một phần ở giữa đám đông; reviewer lướt nhanh bị lọt bẫy. | Xác định ranh giới rõ ràng: xe thấy ≥ 20% phải gán box riêng; cụm xe phía sau mờ nhạt gom `ignore_region`. | Mục 4.1.6: Tiêu chuẩn phân định cụm xe dày đặc và vật cá thể. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn làm contract ban đầu | Chuẩn bị dữ liệu ban đầu cho 4 ảnh gold |
| v1.1 | Chốt 4 câu hỏi mở mục 6 (xe điện, cụm xe dày, hàng nhô ra, kính xe); bổ sung quy định gán xe ba bánh có mui/kính chắn gió; chuẩn hoá ranh giới giữa `ignore_region` và box cá thể trong cảnh đông xe máy | Xử lý lỗi phân loại sai tuk-tuk của YOLO26 (CAL-01), giải quyết ca khó xe điện (CAL-02), hàng hoá cồng kềnh can_review (CAL-03) và bẫy lọt do nhìn lướt (CAL-04) |
