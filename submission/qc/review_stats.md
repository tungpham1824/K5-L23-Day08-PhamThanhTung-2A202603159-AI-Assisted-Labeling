# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 513; đối tượng sau duyệt: 440; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 392 | 392/513 = 76.4% | 513 box máy |
| edited (sửa) | 33 | 33/513 = 6.4% | 513 box máy |
| deleted (xoá) | 84 | 84/513 = 16.4% | 513 box máy |
| added (vẽ thêm) | 15 | 15/440 = 3.4% | 440 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 6.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 225 | 14 | 30 | 2 |
| bicycle | 5 | 1 | 4 | 2 |
| motorbike | 104 | 9 | 15 | 3 |
| car | 44 | 1 | 6 | 3 |
| bus | 2 | 0 | 1 | 1 |
| truck | 2 | 0 | 3 | 3 |
| three_wheeler | 10 | 8 | 25 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 52 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 47 | 24 | 5 | 66.2% (KTC 95%: 55–76%) | 90.4% (KTC 95%: 79–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - extra: 2/3
  - wrong_class: 2/3
  - missing: 1/3
  - lọt: img_006.jpg (loose), img_027.jpg (extra), img_028.jpg (missing), img_029.jpg (missing), img_035.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
