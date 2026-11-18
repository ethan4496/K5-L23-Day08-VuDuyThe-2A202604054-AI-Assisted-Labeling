# Thông tin học viên

- Họ và tên: Vũ Duy Thế
- MSHV: 2A202604054
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: ethan4496
- Repo bài nộp (URL): https://github.com/ethan4496/K5-L23-Day08-VuDuyThe-2A202604054-AI-Assisted-Labeling
- Commit chốt bài (hash): 5e402c68d492f513890248fbc83c5aa8970c9de1
- Máy: macOS (MacBook Air), GPU MPS (Apple Silicon)
- CVAT (Docker): 2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 474/519 = 91.3% / 8/519 = 1.5% / 37/519 = 7.1% (mẫu số: 519 box máy); added: 19/501 = 3.8% (mẫu số: 501 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 32/74 = 43.2% / 32/39 = 82.1% (39 đối tượng gold, 4 ảnh; lạc quan vì ngưỡng chọn trên chính gold)
- Bắt bẫy: 11/12 = 91.7% (lọt: extra ở img_034)
- Thời gian: gán tay 15/4 = 3.75 phút/ảnh; duyệt có pre-label 40/24 = 1.67 phút/ảnh
- Lỗi QC (vòng 1, trước khi sửa): 62.5% theo ảnh (5/8), 2.6% theo đối tượng (5/190); đã sửa đủ 5 lỗi trong `qc_fixed.json`
- **Quyết định:** Rework — lý do một dòng: QC vòng 1 có 5/8 ảnh lỗi (62.5%, vượt ngưỡng 2%), lỗi lặp ở ảnh dày (box lệch + bỏ sót); đã sửa đủ 5 lỗi trong `qc_fixed.json` nhưng chưa QC lại bản đã sửa nên chưa Release.
