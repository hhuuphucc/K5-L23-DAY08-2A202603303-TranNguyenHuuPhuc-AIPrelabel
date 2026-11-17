# Thông tin học viên

- Họ và tên: Trần Nguyễn Hữu Phúc
- MSHV: 2A202603303
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: hhuuphucc
- Repo bài nộp (URL): https://github.com/hhuuphucc/K5-L23-DAY08-2A202603303-TranNguyenHuuPhuc-AIPrelabel
- Commit chốt bài (hash): 90f936f
- Máy: Windows 11 Pro, CPU (torch 2.15.0+cpu; có GPU RTX 3050 Ti nhưng chưa dùng CUDA)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 467 / 10 / 43 (mẫu số: 520 box máy; 89.8% / 1.9% / 8.3%); added: 46 (mẫu số: 523 đối tượng sau duyệt; 8.8%)
- So với gold: precision / recall của **máy** 73.0% (54/74) / 68.4% (54/79)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 2.0 phút/ảnh (8 phút / 4 ảnh); duyệt có pre-label 0.63 phút/ảnh (15 phút / 24 ảnh)
- Lỗi QC: 62.5% theo ảnh (5/8), 2.7% theo đối tượng (6/222)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 5/8 = 62.5% > 2%, lỗi `thua` của `three_wheeler` lặp lại và bắt bẫy 7/12 = 58.3% < 80%
