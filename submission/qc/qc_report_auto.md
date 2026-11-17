# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 215 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/215 = 2.8% | 215 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/8 = 62.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 379/431 = 87.9% / 21/431 = 4.9% / 28/431 = 6.5% | 431 box máy |
| Máy: added | 69/471 = 14.6% | 471 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 4 | img_006.jpg, img_014.jpg, img_029.jpg |
| thua | 2 | img_006.jpg, img_018.jpg |

**Lỗi hệ thống**: 'sai_vi_tri_kich_thuoc' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 5 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **86%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 62.5% > 2%
- lỗi hệ thống: 'sai_vi_tri_kich_thuoc' chiếm 4/6 lỗi
- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
