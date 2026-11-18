# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 190 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 5/190 = 2.6% | 190 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/8 = 62.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 474/519 = 91.3% / 8/519 = 1.5% / 37/519 = 7.1% | 519 box máy |
| Máy: added | 19/501 = 3.8% | 501 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 2 | img_005.jpg, img_018.jpg |
| bo_sot | 2 | img_034.jpg, img_035.jpg |
| sai_lop | 1 | img_003.jpg |

## Giới hạn thống kê
- 5 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **86%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 62.5% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
