# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604054 - Vũ Duy Thế

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Gán **từng vật** nếu tách được và cao ≥ 20 px (box ôm cả phần bị che một phần); `ignore_region` chỉ cho cụm ở xa cao < 20 px. Mật độ không phải lý do để gom | img_005, img_018 | CAL-05 |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe? | Kính (có mặt kính) **thuộc** mask xe. Khung hở không kính là **lỗ**: lỗ > 5% diện tích vật phải bỏ ra khỏi polygon | img_013 | CAL-04 |

**Chưa chốt** (không có đủ ca trong 24 ảnh batch để quyết có cơ sở):
- Câu 1 (xe đạp điện): không gặp ca nào rõ ràng; bicycle chỉ 7 vật trong release.
- Câu 3 (hàng nhô ra), câu 4 (ngưỡng 20% người trong xe có mui): chưa có lỗi QC liên quan; giữ quy tắc v1.0.
- Câu 6 (box chồng nhau ảnh đêm): bộ dữ liệu không có ảnh đêm.
- Câu 7 (conf tối thiểu không cần sửa): **không đặt** — accepted 91.3% nhưng QC vẫn tìm 5 lỗi trong 190 vật, nên mọi box vẫn phải nhìn.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_004, img_013 | Tuk-tuk có mui | YOLO26m gọi là `car` (0.29–0.72), có khi 2 box car trên một xe | Đổi thành `three_wheeler`; merge đặt YOLOE trước YOLO26 | Mục 2: xe ba bánh có mui luôn là `three_wheeler` (CAL-01) |
| img_002, img_003 | Xe đẩy bán hàng, xe bán tải, vật cắt mép | YOLOE-26s báo `three_wheeler` nhầm ở conf 0.37–0.40; deleted 17 box three_wheeler | Xoá box không phải xe ba bánh và box trùng; conf < 0.45 bắt buộc kiểm | Mục 2 + 4.3: phải thấy 3 bánh hoặc mui + khoang khách (CAL-02) |
| img_003, img_013, img_014, img_035 | Vật bị mép ảnh cắt, chỉ còn dải 20–55 px | YOLO26 / YOLOE vẫn sinh box; mình tick `can_review` | Không gán; đã xoá trong `qc_fixed.json` | Mục 4.1 (mới): vật cắt mép chỉ gán khi phần thấy rộng ≥ 40 px và đủ nhận lớp (CAL-03) |
| img_013 | Tuk-tuk có người lái, khung mui hở | SAM 2.1-b 1 click âm vẫn tô người lái + người phía sau khung | Người lái là `person` riêng; lỗ lớn qua khung bỏ khỏi mask | Mục 4.2: thêm click âm lên đầu, thân, chân người lái (CAL-04) |
| img_034 | Ảnh dày 29 vật | Bẫy `extra` lọt; box motorbike cao 17 px; 1 xe máy bị sót | Xoá box không có vật thật, xoá box < 20 px, thêm xe sót | Checklist: đếm vật trước khi nhìn box máy với ảnh > 25 vật (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu mở 2 và 5; thêm quy tắc tuk-tuk luôn `three_wheeler`, tiêu chí nhận `three_wheeler`, vật cắt mép < 40 px không gán, tách người lái khỏi mask xe, đếm vật trước khi duyệt ảnh dày | CAL-01 … CAL-06 |
