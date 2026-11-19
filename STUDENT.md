# Thông tin học viên

- Họ và tên: Phạm Thanh Tùng
- MSHV: 2A202603159
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: tungpham1824
- Repo bài nộp (URL): https://github.com/tungpham1824/K5-L23-Day08-PhamThanhTung-2A202603159-AI-Assisted-Labeling
- Commit chốt bài (hash): e9ac802
- Máy: WSL2 Ubuntu 24.04 (Windows 11), không GPU (chạy model trên Google Colab T4)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 392/513 = 76.4% / 33/513 = 6.4% / 84/513 = 16.4% (mẫu số: 513 box máy); added: 15/440 = 3.4% (mẫu số: 440 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 47/71 = 66.2% (KTC 95%: 55–76%) / 47/52 = 90.4% (KTC 95%: 79–96%)
- Bắt bẫy: 7/12 = 58.3% (KTC 95%: 32–81%)
- Thời gian: gán tay 3.0 phút/ảnh (12 phút / 4 ảnh gold); duyệt có pre-label 3.54 phút/ảnh (85 phút / 24 ảnh batch)
- Lỗi QC: 0/8 = 0.0% theo ảnh (mẫu số: 8 ảnh), 0/167 = 0.0% theo đối tượng (mẫu số: 167 đối tượng)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy đạt 7/12 = 58.3% < 80% (ngưỡng tối thiểu của label contract), cần rà soát lại quy trình duyệt lô xe máy dày đặc và siết chặt checklist trước khi release.
