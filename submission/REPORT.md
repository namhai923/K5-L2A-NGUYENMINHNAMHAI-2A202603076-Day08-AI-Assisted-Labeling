# Báo cáo Day 08 — 2A202603076 - Nguyễn Minh Nam Hải

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-m | person 0.35, motorbike 0.5, car 0.35, còn lại 0.3 | r01 (0.25) nhiều FP; r02 (0.5) mất 35% vật xa; sweep: không ngưỡng nào đạt recall 90% → chọn mức giữa |
| three_wheeler | YOLOE-s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.3 | r03: bắt được xích lô mà YOLO26 bỏ (img_032); YOLO26 không có lớp này (r01) |

-  Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: `SAM 2.1` (mặc định, mask ôm sát ở img_004, r04); nhưng SAM 1 tách tài xế tốt hơn ở img_013 (mục 5). Chạy CPU 12.35 s/ảnh → chỉ chạy cho vật cần mask.
- **Không sửa được bằng ngưỡng:** tuk-tuk → `car` (img_017), xích lô → `bicycle` (img_010), tuk-tuk chia 2 nửa box (img_013). Sửa: YOLOE cho `three_wheeler` + đổi lớp / gộp box khi duyệt.
- **Điểm cao mà sai:** img_017 tuk-tuk → car 0.86; img_013 nửa tuk-tuk 0.75. **Điểm thấp mà đúng:** img_003 person 0.42; img_024 car 0.35.
- **Giới hạn:** ngưỡng chọn trên chính 4 ảnh gold (55 vật) → số đo máy trên gold lạc quan; sweep gợi ý three_wheeler 0.15 nhưng không dùng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 379 | 87.9% | 431 |
| edited | 21 | 4.9% | 431 |
| deleted | 28 | 6.5% | 431 |
| added | 69 | 14.6% | 471 |

- Lớp tệ nhất: **three_wheeler** (accepted 4/24; edited 8, deleted 12, added 8): gọi sai lớp, chia đôi box, quang gánh bị gán (img_017, img_013, img_014). Tiếp theo **motorbike** sót nhiều nhất (added 35) ở ảnh dày xe (img_005, img_028).
- So với gold: precision 40/53 = 75.5% (KTC 62–85%), recall 40/55 = 72.7% (KTC 60–83%); mẫu số 55 đối tượng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12. Lọt: wrong_class (img_005), loose (img_014, img_034), missing (img_027). Bước bị bỏ: kiểm 4 cạnh box (bước 4) ở box trông đã ổn; quét ảnh trước (bước 0) chưa kỹ ở img_027.
- Accepted 87.9% mà bắt bẫy chỉ 67% → người duyệt tin box máy, nhất là box "trông vừa"; lỗi lỏng nhỏ lọt nhiều nhất (2/3 bẫy loose).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 10 phút/ảnh (40 phút / 4 ảnh gold) vs duyệt 3.75 phút/ảnh (90 phút / 24 ảnh batch). Thiên lệch (bạn gán gold trước nên đã quen cảnh): có lợi cho bước duyệt — gold là 4 ảnh khác batch nhưng cùng loại cảnh (đường phố VN/Thái, xe máy dày, tuk-tuk), nên lúc duyệt đã quen quy tắc và kiểu lỗi máy; tốc độ duyệt 3.75 phút/ảnh vì vậy hơi lạc quan, chênh thật với gán tay có thể nhỏ hơn ~2.7×.
- Kết luận: **có** cho person / car / motorbike (accepted 240/250, 43/48, 81/88); **không** cho three_wheeler (4/24) — vẽ tay hoặc duyệt 100%.
- **Ba quy tắc giảm thiên kiến:**
  1. Đếm vật trên ảnh trước khi bật nhãn máy; so số đếm với số box (kiểm: ghi số đếm mỗi ảnh).
  2. Kiểm 4 cạnh mọi box, kể cả conf cao (kiểm: bẫy loose bắt 3/3).
  3. Duyệt 100% box `three_wheeler` và 6 ảnh dày xe (kiểm: bắt bẫy ≥ 10/12).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004 | SAM 2.1 `b` | 3 (2+, 1−) | Không (người đứng sau bị loại) | Lem bóng đổ dưới mũi xe; thiếu biển TAXI | `tuktuk_img004_sam2.jpg` |
| img_013 | SAM 1 `b` | 3 (2+, 1−) | Gần như không; lem mặt, bàn chân tài xế | Sàn xe cạnh tài xế lốm đốm; thiếu biển TAXI | `tuktuk_img013_sam1.jpg` |
| img_013 | SAM 2.1 `b` | 3 (2+, 1−) | **Có** tài xế (thân dưới, chân) | Lem sang người áo sọc ở quán phía sau | `tuktuk_img013_sam2.jpg` |
| img_013 | SAM 2.1 `b` | 4 (2+, 2−: thêm đầu tài xế) | Loại được đầu; chân, thân dưới vẫn dính | Vẫn lem người phía sau; click âm chỉ cắt vùng quanh điểm | `tuktuk_img013_sam2_addneg.jpg` |
| img_017 | SAM 2.1 `b` | 3 (2+, 1−) | Không có người / hàng | Khoang hở bị lấp kín; click âm trên mui gần như vô tác dụng; thiếu biển TAXI | `tuktuk_img017_sam2.jpg` |

- **Kết quả:** 3 click đủ cho xe không có người. Với tài xế ngồi trong xe, SAM 1 (3 click) tách tốt hơn SAM 2.1 (kể cả 4 click); còn lại cần thêm click âm ở chân hoặc sửa tay.
- **Guideline 4.2 giúp:** kính thuộc xe, bóng đổ không thuộc xe (4.2.3); mỗi vật một polygon, không chồng nhau (4.2.4), nên phải cắt tài xế khỏi mask xe.
- **Guideline 4.2 thiếu:** (1) người ngồi **trong** xe thuộc phần nào; (2) biển TAXI trên nóc có thuộc xe không; (3) khoang hở > 5% (4.2.2 chỉ nói lỗ nhỏ).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa làm.

## 6. Calibration (Part 6)

- Số quyết định: 11 (`calibration_decisions.csv`): 6 `cau_hoi_mo`, 2 `loi_may`, 1 `self_check`, 1 `bay_lot`, 1 `can_review`. Quan trọng nhất: **CAL-07** (tuk-tuk / xích lô 1 box `three_wheeler`), vì `three_wheeler` là lớp máy tệ nhất (accepted 4, edited 8, deleted 12, added 8).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 2–7 (câu 1 chưa có ảnh); không gộp ≥ 2 xe vào 1 box; tuk-tuk 1 box cả xe + mui; thêm vật không phải xe vào "Không gồm"; `ignore_region` không phủ vật rõ; kiểm đủ 4 cạnh box; van ≥ ~10 chỗ là `bus`.
- Nguyên nhân gốc: chủ yếu **guideline mơ hồ** (6/11 ca từ câu hỏi mở); tiếp theo là **máy** (2/11: gọi sai lớp xe ba bánh, gán vật không phải xe) và **lỗi duyệt** (2/11: `ignore_region` sai chỗ, box lỏng lọt).

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh, 215 đối tượng (3 ca khó: img_018, img_024, img_029 + 5 ngẫu nhiên, seed 42). Người kiểm: 2A202603076.
- Lỗi theo **đối tượng**: 6/215 = 2.8% · theo **ảnh**: 5/8 = 62.5% (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` 4/6 (vd img_014 #140 cắt tay lái; img_006 #67, img_029 #400 gộp nhiều xe vào 1 box) · `thua` 2/6 (vd img_018 #192 cao < 20 px). Không có bỏ sót / sai lớp / sai thuộc tính trong mẫu. `sai_vi_tri_kich_thuoc` lặp lại = lỗi hệ thống.
- Bắt bẫy 8/12 = 66.7% (< 80%). Máy: accepted 379/431 = 87.9%, edited 4.9%, deleted 6.5%; added 69/471 = 14.6%.
- **Giới hạn mẫu nhỏ:** 5/8 ảnh lỗi, cận trên 95% (Wilson) = 86%; 8 ảnh không chứng minh được ngưỡng 2%.
- Script kiểm được: đếm lỗi, tỉ lệ, cận tin cậy, bẫy. **Không** kiểm được: box có ôm sát thật không, vật cả máy lẫn người cùng bỏ sót, người kiểm có theo đúng checklist không.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): ảnh lỗi 5/8 = 62.5% > 2%; lỗi `sai_vi_tri_kich_thuoc` lặp lại (4/6); bắt bẫy 8/12 = 66.7% < 80%.
Nếu Rework / Hold, việc cần làm tiếp:
1. Sửa 6 lỗi QC: tách box gộp (img_006 #67, img_029 #400), ôm sát (img_014 #140), xoá (img_006 #68, img_018 #192), thu ignore khỏi xích lô (img_035 #471).
2. Guideline v1.1: không gộp ≥ 2 xe vào 1 box; cụm mờ / dày dùng `ignore_region`; `ignore_region` không phủ vật rõ.
3. Duyệt lại cả 24 ảnh theo quy tắc mới, ưu tiên 6 ảnh dày xe (005, 018, 021, 024, 028, 029).
4. Xem 4 bẫy lọt trong `review_stats.md`, thêm bước checklist cho đúng loại đó.
5. Đổi người duyệt.
6. QC lại mẫu mới (seed khác, ≥ 10–15 ảnh).
7. Release khi ảnh lỗi ≤ 2% và bắt bẫy ≥ 10/12.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (Opus 5.5) — cài thiếu `transformers`/`huggingface_hub`, tải model `--set core`, chạy `check_env.py`; xem ảnh `work/prelabel/*/viz/` cùng mình để ghi lỗi sai lớp tuk-tuk, sót nền xa, độ bám viền SAM. Kiểm tra: tự mở lại ảnh viz đối chiếu từng nhận xét (ảnh, vật, conf).
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không — gán tay 4 ảnh gold xong và export trước khi xem pre-label hay hỏi AI.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, chỉ CPU (không GPU), CVAT v2.74.1 qua Docker; `check_env.py` báo FAIL thiếu `transformers` → `pip install -r requirements.txt`; chưa có trọng số → `download_models.py --set core` (sam2.1_b, yolo26m, yoloe-26s-seg). SAM 12.35 s/ảnh trên CPU nên chỉ chạy 2 ảnh.
- Dùng cache `data/cache/` thay vì tự chạy? Không — tự chạy r01–r06 bằng `prelabel.py` trên CPU.
- Lỗi / bug script đã gặp (nếu có): không gặp bug script; `submission/model_log.csv` một lần bị ghi đè bằng nội dung `work/runs.csv` (mất 3 cột nhận xét) → điền lại và thêm r05/r06.
- Một điều bạn làm khác lần sau: duyệt 100% box `three_wheeler` và kiểm 4 cạnh mọi box ngay từ đầu (bẫy loose/wrong_class lọt nhiều nhất), thay vì tin box "trông vừa".
