# Báo cáo Day 08 — 2A202603303 - Trần Nguyễn Hữu Phúc

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`, CPU) | mặc định `--conf 0.3` (không lớp nào có ngưỡng riêng) | r01 và r02: bắt chắc person, motorbike, car. `threshold_sweep` không tìm được ngưỡng nào ≤ 0.5 đạt recall 90% cho person, bicycle, motorbike, car; bus (0 vật) và truck (1 vật) dưới 5 vật nên không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r05: tìm được xe ba bánh mà YOLO26 bỏ sót (img_002, img_032). `threshold_sweep`: ngưỡng cao nhất còn giữ recall ≥ 90% là 0.15 (recall 7/7 = 100%, precision 78%, 7 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (sau đó `inject_traps.py --student "2A202603303"`).
- **Phiên bản SAM tôi chọn: SAM 2.1 base (`sam2.1_b.pt`)**, lý do: ở img_013 mask đầy đủ hơn MobileSAM (SAM 1, `mobile_sam.pt`), chỉ mất khoảng 8.8 giây mỗi ảnh trên CPU (r03) nên chấp nhận được; MobileSAM nhanh hơn nhưng mask xe vỡ ở sàn và đầu xe. Cả hai chạy offline, không cần quyền truy cập đặc biệt (khác SAM 3).
- Lỗi **không sửa được bằng đổi ngưỡng**: YOLO26 không có lớp xe ba bánh nên tuk-tuk xanh ở img_002 không có box nào ở cả `--conf 0.25` (r01) lẫn 0.5 (r02), và ở img_013 bị gọi `car` 0.42. Đổi ngưỡng chỉ thêm hoặc bớt box, không thêm được lớp mà model không có. Cách sửa: lấy `three_wheeler` từ YOLOE (r05) và đặt YOLOE đứng trước khi gộp để thắng `car/truck` của YOLO26 khi IoU ≥ 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**: img_032 box `three_wheeler` 0.27 là xích lô thật, trong khi img_002 các box 0.37–0.40 và img_011 box 0.28 là sai (r05); điểm máy không phản ánh đúng sai. Ở QC, cả 3 box thừa đều là `three_wheeler` điểm thấp (img_003 #19 0.21 và #20 0.20, img_035 #508 0.48).
- Giới hạn: tôi chọn ngưỡng trên chính 4 ảnh gold rồi chấm máy trên chính các ảnh đó, nên precision 73.0% và recall 68.4% của máy trên gold là lạc quan. Riêng `three_wheeler` chỉ có 7 vật gold; các lớp còn lại có 8–37 vật hoặc ít hơn.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 467 | 89.8% | 520 box máy |
| edited | 10 | 1.9% | 520 box máy |
| deleted | 43 | 8.3% | 520 box máy |
| added | 46 | 8.8% | 523 đối tượng sau duyệt |

(`review_stats` đã loại 21 đối tượng quanh vùng bẫy khỏi bảng này. Trong số deleted và added có 10 ca đổi lớp, nên máy sót thật khoảng 36 vật.)

- **Lớp máy tệ nhất: `three_wheeler`**: 20 deleted trên 41 box máy (48.8%), chỉ 16 accepted (39.0%). Nguyên nhân chính là ngưỡng 0.15 sinh nhiều box thừa, ví dụ img_003 #19 và #20. Kế đến `truck`: 5 deleted trên 6 box (83.3%), và `car`: 20 added (máy sót nhiều ô tô). Ngược lại `person` accepted 264/273 và `motorbike` 135/139 (nhưng motorbike có 9 added).
- So với gold: precision / recall của **máy** = 73.0% (KTC 95%: 62–82%; 54 TP trên 74 box máy) và 68.4% (KTC 95%: 57–78%; 54 TP trên 79 đối tượng gold). Con số lạc quan vì chọn ngưỡng trên gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **7/12 = 58.3%** (KTC 95%: 32–81%), dưới ngưỡng đạt 80%. Bắt: `extra` 3/3, `wrong_class` 3/3. Lọt: `loose` 0/3 (img_014, img_031, img_024) và `missing` 1/3 (img_006, img_028). Tôi bắt tốt lỗi lộ rõ (box ma, sai lớp) nhưng lọt lỗi cần soi kỹ: tôi nghĩ mình đã bỏ qua việc soi 4 cạnh từng box (bước 4: lỏng) và bước quét ảnh trước khi hiện nhãn máy (bước 0: bỏ sót), nhất là ở các ảnh dày.
- Accepted cao (89.8%) mà bắt bẫy thấp (58.3%) cho thấy tôi giữ nguyên phần lớn nhãn máy nên dễ bỏ qua lỗi tinh vi; đây chính là thiên kiến tự động hóa: nhãn máy trông hợp lý thì mắt ít nhìn lại ảnh. Số accepted cao cũng không nói gì về vật máy bỏ sót (recall của máy trên gold chỉ 68.4%).
- **Bẫy lộ trước:** lần đầu chạy `review_stats.py` lúc tôi chưa review thật, báo cáo có in danh sách ảnh và loại bẫy; vì vậy phép đo bắt bẫy lần review sau có thể bị ảnh hưởng. Tôi giữ nguyên kết quả 7/12 và nêu rõ giới hạn này.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 8 phút / 4 ảnh = **2.0 phút/ảnh**; duyệt có pre-label 15 phút / 24 ảnh = **0.63 phút/ảnh**, nhanh hơn khoảng 69%. Thiên lệch: tôi gán gold trước nên đã quen cảnh phố này; nhiều ảnh batch có cụm xe dày nhưng tôi duyệt nhanh hơn mức khuyến nghị (~3 phút/ảnh), nên phần tiết kiệm này một phần là do duyệt ít kỹ hơn (bắt bẫy 58.3%, ảnh lỗi 62.5% ở QC).
- Kết luận: **chỉ đáng dùng cho person / motorbike / car**, kèm quét ảnh trước và soi 4 cạnh; **không tin `three_wheeler`** ở ngưỡng 0.15 (39.0% accepted, 48.8% deleted). Dựa trên: accepted theo lớp (person 264/273, motorbike 135/139, car 42/51, three_wheeler 16/41), recall máy trên gold 68.4%, và tỉ lệ bắt bẫy 58.3%.
- **Ba quy tắc giảm thiên kiến của tôi** (kiểm tra được):
  1. Với mọi box `three_wheeler` điểm dưới 0.5, đối chiếu từng box với ảnh và xoá box trùng hoặc không có xe. Kiểm bằng QC: số box `thua` trên `three_wheeler` ở mẫu QC lần sau phải bằng 0 (lần này 3/222).
  2. Soi 4 cạnh của từng box trước khi bấm Accept, nhất là ảnh dày xe. Kiểm bằng số: bắt bẫy `loose` phải đạt ít nhất 2/3 (lần này 0/3).
  3. Mỗi ảnh: ẩn nhãn máy, ghi số đếm vật theo lớp vào sổ tay trước khi hiện nhãn máy, rồi so với số `added`. Kiểm bằng sổ ghi chép có đủ 24 ảnh, và bắt bẫy `missing` đạt ít nhất 2/3 (lần này 1/3).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk, người lái + khách ngồi sau) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Gồm người lái (áo, quần, chân) và phần người ngồi sau; hàng hoá không có | Một click âm trên người lái không đủ để loại người lái ra khỏi mask; mask có lỗ nhỏ ở vùng áo người lái | `masks/tuktuk_img013_sam2.jpg` |
| img_013 (thêm 2 điểm âm: đầu, chân người lái) | SAM 2.1 base | 5 (2 dương, 3 âm) | Loại được cả người lái lẫn người ngồi sau; chỉ còn xe | Tách người xong thì mask xe bị vỡ: mái và thân còn nhiều lỗ, bánh sau và sàn xe không còn trong mask (13.5% ảnh, 2 polygon). Thêm điểm âm gần xe có thể làm mất luôn phần xe kề đó, cần thêm điểm dương | `masks/tuktuk_img013_sam2_neg3.jpg` |
| img_013 (cùng toạ độ) | MobileSAM (SAM 1, `mobile_sam.pt`) | 3 (2 dương, 1 âm) | Loại được thân người lái (theo click âm) nhưng mask còn dính một mảng ở mặt; người ngồi sau không nằm trong mask | Mask rời rạc ở sàn xe, chắn bùn và phần đầu xe bên phải, kém đầy đủ hơn SAM 2.1 | `masks/tuktuk_img013_sam1.jpg` |
| img_004 (tuk-tuk xanh-đỏ ở phải) | SAM 2.1 base | 3 (2 dương, 1 âm) | Chỉ gồm xe (mái, thân, bánh trước), không có người | Click dương thứ hai nằm trên mặt đất dưới bánh nên mask lem ra vài điểm nền; YOLO26 ở lượt r03 không có box cho xe này, nên SAM theo điểm bắt được vật mà detector bỏ sót | `masks/tuktuk_img004_sam2.jpg` |
| img_017 (tuk-tuk xanh-cam) | SAM 2.1 base | 3 (2 dương, 1 âm) | Gồm cả xe lẫn nội thất (ghế); không thấy người hay hàng hoá trong xe | Click âm đặt trên mái xe (chữ trên mái) nên đặt sai chỗ: mask vẫn phủ mái, tức click âm bị bỏ qua; mask đủ, khít | `masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2 (từ 4 ảnh mask ở trên):
  - **Quy tắc giúp được:**
    - *4.2.4 (hai vật chạm nhau: mỗi vật một polygon, vùng chạm thuộc vật phía trước)* cho tôi tiêu chí để chấm mask `img_013`. Người lái ngồi phía trước thân xe, nên mask xe phải loại người lái. SAM 2.1 với 1 điểm âm vi phạm quy tắc này (mask gồm cả người lái), còn bản 3 điểm âm thì đúng quy tắc nhưng mask xe bị vỡ.
    - *4.2.3 (kính thuộc vật)* khớp với `img_013`: phần kính chắn gió và cabin nằm trong mask xe ở cả hai bản SAM, tôi không phải quyết thêm.
    - *4.2.1 (polygon bám biên phần nhìn thấy, lệch ≤ ~3 px)* dùng được cho `img_004` và `img_017`, nơi mask khít xe; ở `img_004` điểm dương trên nền làm mask lem vài điểm, vi phạm quy tắc này.
  - **Quy tắc thiếu hoặc mơ hồ:**
    - **Người ngồi trong xe có thuộc mask xe không?** 4.2 không nói. Quy tắc 4.1.4 (box) chỉ nói người và xe là hai box riêng, nhưng với mask thì người ngồi lọt vào giữa khung xe nên SAM 2.1 gộp luôn vào mask xe. Cần một quy tắc rõ: mask xe loại người lái và hành khách, và hành khách nhìn xuyên qua khung hở có mask `person` riêng.
    - **Lỗ thủng của khung hở (4.2.2):** quy tắc chỉ nói "lỗ lớn thì bỏ ra khỏi polygon", nhưng tuk-tuk có 3 mặt hở, khoảng trống giữa khung và sàn có thể chiếm tới hơn 5% diện tích. Nếu áp dụng đúng chữ thì mask bị khoét thủng nhiều, còn SAM thì tự trả mask đặc. Chưa rõ nên theo bên nào.
    - **Hàng hoá và mái nhô ra** (câu hỏi mở số 3 của mục 6): chưa có quy tắc cho mask.
    - **Ngưỡng chấp nhận mask khi làm bằng SAM:** không nói tối đa bao nhiêu điểm click là hợp lý, và cách xử lý lỗ nhỏ SAM tạo ra (ví dụ vùng áo người lái ở `img_013`) so với lỗ 5% của 4.2.2.
  - Nguyên nhân gốc những chỗ thiếu: **guideline mơ hồ**, không phải máy hay tôi sai, nên các ca này đưa vào calibration (Part 6) và `guideline_v1.1`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử. CVAT Docker mặc định của tôi (v2.74.1) chưa cài serverless (Nuclio) nên chưa có model AI trong CVAT; tôi dùng `scripts/sam2_points.py` offline thay thế.

## 6. Calibration (Part 6)

- Số quyết định: **6** (`calibration_decisions.csv`); quan trọng nhất: **CAL-01** (box `three_wheeler` điểm thấp trùng hoặc thừa còn sót, lỗi hệ thống trong QC) và **CAL-04** (mask xe có gồm người ngồi không, từ Part 5). Đủ các nguồn `loi_may` (CAL-01, CAL-04), `cau_hoi_mo` (CAL-02, CAL-03), `can_review` (CAL-05) và `bay_lot` (CAL-06).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: 4.1.8 (một vật một box), 4.2.5 (mask xe không gồm người ngồi), 4.2.6 (khoảng hở của xe ba bánh hở tính là phần xe), làm rõ 4.1.6 (ngưỡng 20 px, `ignore_region` chỉ cho cụm ≥ 5 vật nhỏ), và chốt các câu 2, 5, 6, 7 cùng một phần câu 3 của mục 6. Câu 1 (xe đạp điện), câu 4 (người trong xe có mui) và phần hàng nhô xa của câu 3 chưa chốt vì chưa gặp ca trong dữ liệu.
- Nguyên nhân gốc: CAL-01, CAL-03 (box dưới 20 px) và CAL-06 là **tôi (người duyệt) làm sai hoặc lỗi của máy**, không đổi guideline (riêng 4.1.8 làm rõ thêm); CAL-02 và CAL-04 là **guideline mơ hồ**; CAL-05 là ca khó của người gán, đã có quy tắc 4.3.
- Tồn đọng đã ghi nhận: gold còn 12/79 box cao dưới 20 px (không sửa để giữ gold sạch); batch còn 16/532 box như vậy trong `batch_reviewed.json` và đã sửa bằng `labels/qc_fixed.json` (xoá 16 box, thêm 2 `ignore_region`), nên release còn 518 đối tượng.

## 7. QC và quyết định (Part 6)

- Mẫu: **8** ảnh (3 ảnh `can_review` nhiều nhất: img_005, img_024, img_028, và 5 ngẫu nhiên: img_003, img_008, img_014, img_017, img_035; seed 42; 222 đối tượng); người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt).
- Lỗi theo **đối tượng**: **6/222 = 2.7%** · theo **ảnh**: **5/8 = 62.5%** (ngưỡng Release ≤ 2%).
- Top lỗi: `thua` 3 (img_003 #19 và #20, img_035 #508); `sai_vi_tri_kich_thuoc` 2 (img_005 #52, img_024 #309); `bo_sot` 1 (img_028, xe máy ở mép trái). Hai trong sáu lỗi (img_024 #309, img_028) chính là bẫy `loose` và `missing` đã lọt lúc review.
- **Giới hạn mẫu nhỏ**: 5/8 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **86%**. Ngưỡng 2% không thể chứng minh bằng 8 ảnh. Ngoài ra, hai lỗi ở img_024 và img_028 được xác nhận với đáp án bẫy sau khi export, nên QC ở hai ảnh đó không hoàn toàn độc lập.
- Script kiểm được tên file, lớp, box trong ảnh, đủ ảnh và số đo. Không kiểm được box có ôm sát vật không, lớp có đúng thực tế không, vật mà cả tôi lẫn máy cùng bỏ sót (ngoài gold), hay việc tôi có theo checklist thật không.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (Hold → Rework → Release): không Hold vì không có ảnh hỏng và các câu mở còn lại chưa gặp ca trong dữ liệu; Rework vì ảnh lỗi 62.5% > 2%, lỗi `thua` lặp lại ở `three_wheeler` (lỗi hệ thống) và bắt bẫy 58.3% < 80%, nên lô duyệt chưa đáng tin.
Việc cần làm tiếp: sửa 6 lỗi QC trong CVAT (xoá box trùng, sửa box lệch, vẽ lại xe máy bị bỏ sót) rồi export vào `qc_fixed.json` và chạy lại `build_release.py`; duyệt lại các ảnh dày xe theo 3 quy tắc ở mục 4; nâng ngưỡng hoặc duyệt riêng `three_wheeler`; QC lại với mẫu lớn hơn 8 ảnh.

## 8. Ghi chú cá nhân

- **Dùng AI để hỗ trợ:** tôi dùng Claude (Claude Code) để hướng dẫn cài đặt, đọc tài liệu lab
- Gold có bị nhiễm không? **Không**: `gold_manual.json` được export trước mọi lần chạy model (file pre-label sớm nhất tạo sau gold), tôi không chạy model lên 4 ảnh gold và không xem `data/cache/`, và AI không nhìn ảnh gold trước khi tôi gán xong. Chỉ có **bẫy** bị lộ: lần chạy `review_stats.py` sớm in danh sách ảnh bẫy (xem mục 3).
- Máy / GPU / CVAT: Windows 11, Python 3.14.7, torch CPU (2.15.0+cpu; có RTX 3050 Ti nhưng chưa dùng CUDA); CVAT Docker v2.74.1 (cũ hơn bản 2.81 trong hướng dẫn: hộp thoại xác nhận **Replace annotations** hiện sau khi bấm OK ở bước Upload annotations).
- Dùng cache `data/cache/` thay vì tự chạy? **Không**; tôi tự chạy YOLO26 và YOLOE cho 36 ảnh.
- Lỗi / bug đã gặp: (1) YOLOE tự cài CLIP bị `Access is denied` khi ghi vào `C:\Python314`; sau đó CLIP nằm ở thư mục người dùng và YOLOE chạy được. (2) Lần đầu tôi chạy `merge_prelabels.py` và `inject_traps.py` với `--thr` và MSHV mẫu của hướng dẫn, phát hiện và chạy lại với giá trị của tôi. (3) Lần export `batch_reviewed.json` đầu tiên chưa sửa nhãn nên `review_stats` ra 100% accepted và bắt bẫy 0/12; tôi bỏ file đó và export lại sau khi review thật. Chưa sửa bug nào trong script.
