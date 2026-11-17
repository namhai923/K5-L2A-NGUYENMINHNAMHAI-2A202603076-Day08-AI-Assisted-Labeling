# Thông tin học viên

- Họ và tên: Nguyễn Minh Nam Hải
- MSHV: 2A202603076
- Lớp (L2 / L3 / khác): l2A
- Tài khoản GitHub: https://github.com/namhai923
- Repo bài nộp (URL): https://github.com/namhai923/K5-L2A-NGUYENMINHNAMHAI-2A202603076-Day08-AI-Assisted-Labeling
- Commit chốt bài (hash): b22057658d9fbb437d2af0204e2781d6a034d126
- Máy: Window, không có GPU
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 379 / 21 / 28 = 87.9% / 4.9% / 6.5% (mẫu số: 431 box máy); added: 69 = 14.6% (mẫu số: 471 đối tượng)
- So với gold: precision / recall của **máy** 75.5% (40/53) / 72.7% (40/55)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 10 phút/ảnh; duyệt có pre-label 4 phút/ảnh
- Lỗi QC: 62.5% theo ảnh (5/8), 2.8% theo đối tượng (6/215)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 62.5% > 2%, lỗi `sai_vi_tri_kich_thuoc` lặp lại (4/6), bắt bẫy 66.7% < 80%.
