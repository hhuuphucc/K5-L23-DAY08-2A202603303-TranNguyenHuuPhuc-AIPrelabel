# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 222 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/222 = 2.7% | 222 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/8 = 62.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 467/520 = 89.8% / 10/520 = 1.9% / 43/520 = 8.3% | 520 box máy |
| Máy: added | 46/523 = 8.8% | 523 đối tượng sau duyệt |
| Bắt lỗi bẫy | 7/12 = 58.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 3 | img_003.jpg, img_035.jpg |
| sai_vi_tri_kich_thuoc | 2 | img_005.jpg, img_024.jpg |
| bo_sot | 1 | img_028.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 5 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **86%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 62.5% > 2%
- lỗi hệ thống: 'thua' chiếm 3/6 lỗi
- bắt bẫy 58% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
