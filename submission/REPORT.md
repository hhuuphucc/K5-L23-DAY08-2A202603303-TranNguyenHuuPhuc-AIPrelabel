# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk, người lái + khách ngồi sau) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Gồm người lái (áo, quần, chân) và phần người ngồi sau; hàng hoá không có | Một click âm trên người lái không đủ để loại người lái ra khỏi mask; mask có lỗ nhỏ ở vùng áo người lái | `masks/tuktuk_img013_sam2.jpg` |
| img_013 (thêm 2 điểm âm: đầu, chân người lái) | SAM 2.1 base | 5 (2 dương, 3 âm) | Loại được cả người lái lẫn người ngồi sau; chỉ còn xe | Tách người xong thì mask xe bị vỡ: mái và thân còn nhiều lỗ, bánh sau và sàn xe không còn trong mask (13.5% ảnh, 2 polygon). Thêm điểm âm gần xe có thể làm mất luôn phần xe kề đó, cần thêm điểm dương | `masks/tuktuk_img013_sam2_neg3.jpg` |
| img_013 (cùng toạ độ) | MobileSAM (SAM 1, `mobile_sam.pt`) | 3 (2 dương, 1 âm) | Loại được thân người lái (theo click âm) nhưng mask còn dính một mảng ở mặt; người ngồi sau không nằm trong mask | Mask rời rạc ở sàn xe, chắn bùn và phần đầu xe bên phải, kém đầy đủ hơn SAM 2.1 | `masks/tuktuk_img013_sam1.jpg` |
| img_004 (tuk-tuk xanh-đỏ ở phải) | SAM 2.1 base | 3 (2 dương, 1 âm) | Chỉ gồm xe (mái, thân, bánh trước), không có người | Click dương thứ hai nằm trên mặt đất dưới bánh nên mask lem ra vài điểm nền; YOLO26 ở lượt r03 không có box cho xe này, nên SAM theo điểm bắt được vật mà detector bỏ sót | `masks/tuktuk_img004_sam2.jpg` |
| img_017 (tuk-tuk xanh-cam) | SAM 2.1 base | 3 (2 dương, 1 âm) | Gồm cả xe lẫn nội thất (ghế); không thấy người hay hàng hoá trong xe | Click âm đặt trên mái xe (chữ trên mái) nên đặt sai chỗ: mask vẫn phủ mái, tức click âm bị bỏ qua; mask đủ, khít | `masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
