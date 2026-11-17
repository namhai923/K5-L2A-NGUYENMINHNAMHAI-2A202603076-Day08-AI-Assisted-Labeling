# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603076 - Nguyễn Minh Nam Hải

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện: bicycle hay motorbike? | **Chưa chốt**: không có ảnh ví dụ trong lô | — | — |
| 2 | Xe máy đỗ dày gần camera | Tách được (thấy 1 bánh + tay lái) → từng box; không → `ignore_region`. Không gộp ≥ 2 xe vào 1 box | img_026 | CAL-01 |
| 3 | Hàng nhô ra | Box / mask không gồm hàng nhô > 20%; khay gắn cố định ≤ 20% thì gồm | img_009 | CAL-02 |
| 4 | Người trong xe ba bánh có mui | Gán `person` khi thấy đầu + vai (thay ngưỡng 20%) | img_013 | CAL-03 |
| 5 | Kính xe | Kính + biển TAXI trên nóc thuộc mask xe; người sau kính không gán | img_017 | CAL-04 |
| 6 | Box chồng nhau ảnh đêm | Cùng lớp IoU > 0.7 → giữ 1; mờ không tách → `ignore_region` | img_029 | CAL-05 |
| 7 | Ngưỡng `conf` tự chấp nhận | Không đặt; duyệt mọi box | img_014 | CAL-06 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (mục v1.0) |
|---|---|---|---|---|
| img_017, img_010 | Tuk-tuk, xích lô | Gọi `car` / `bicycle`, hoặc chia 2 nửa box (YOLO26-m, YOLOE-s) | 1 box `three_wheeler` cả xe + mui | 2: `three_wheeler` (CAL-07) |
| img_014, img_006 | Quang gánh, giá bưu thiếp | Gán `three_wheeler` / `truck` (YOLOE-s, YOLO26-m) | Không gán | 2: thêm vào "Không gồm" (CAL-08) |
| img_035 | Cụm xe sau xích lô | — (lỗi duyệt, QC phát hiện) | `ignore_region` không phủ vật rõ | 2: `ignore_region` (CAL-09) |
| img_034 | Bẫy box lỏng | Box lỏng bị lọt khi duyệt | Kiểm đủ 4 cạnh mỗi box | 4.1.1 + checklist bước 4 (CAL-10) |
| img_024 | Xe van chở khách | Gán `truck` / `bus` (YOLO26-m) | ≥ ~10 chỗ → `bus`; minivan gia đình → `car` | 2: `bus` / `car` (CAL-11) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2–7; thêm quy tắc tuk-tuk 1 box, vật không phải xe, `ignore_region`, kiểm 4 cạnh, bus / car | CAL-01 → CAL-11 |
