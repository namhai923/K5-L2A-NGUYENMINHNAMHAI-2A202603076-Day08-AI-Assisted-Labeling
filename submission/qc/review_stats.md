# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 431; đối tượng sau duyệt: 471; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 379 | 379/431 = 87.9% | 431 box máy |
| edited (sửa) | 21 | 21/431 = 4.9% | 431 box máy |
| deleted (xoá) | 28 | 28/431 = 6.5% | 431 box máy |
| added (vẽ thêm) | 69 | 69/471 = 14.6% | 471 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 62.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 240 | 4 | 6 | 13 |
| bicycle | 5 | 1 | 3 | 5 |
| motorbike | 81 | 6 | 1 | 35 |
| car | 43 | 2 | 3 | 3 |
| bus | 3 | 0 | 0 | 3 |
| truck | 3 | 0 | 3 | 2 |
| three_wheeler | 4 | 8 | 12 | 8 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 40 | 13 | 15 | 75.5% (KTC 95%: 62–85%) | 72.7% (KTC 95%: 60–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - missing: 2/3
  - loose: 1/3
  - extra: 3/3
  - lọt: img_005.jpg (wrong_class), img_014.jpg (loose), img_027.jpg (missing), img_034.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
