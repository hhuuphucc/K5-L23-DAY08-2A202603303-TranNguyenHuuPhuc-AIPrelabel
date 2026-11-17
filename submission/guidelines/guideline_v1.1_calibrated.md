# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603303 - Trần Nguyễn Hữu Phúc

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Vật cao từ 20 px trở lên gán **từng vật**, kể cả khi xếp sát nhau. `ignore_region` chỉ dùng cho cụm từ 5 vật trở lên mà tất cả đều cao dưới 20 px (đúng 4.1.6). | img_005, img_028 | CAL-03 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | **Chốt một nửa.** Hàng xếp cố định trong thùng, khay hoặc giỏ của xe thuộc box xe (xe ba gác chở chuối ở img_008, box #107 ôm cả chuối trên khay). **Chưa chốt** hàng nhô xa (tre, rổ): giữ 4.1.5 (nhô quá ~20% chiều dài xe thì không tính), chưa có ảnh ví dụ trong bộ đã duyệt. | img_008 | CAL-03 |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | **Thuộc** mask xe. Người ngồi sau kính vẫn có mask `person` riêng (xem 4.2.5). | img_013 | CAL-04 |
| 6 | Nhiều box chồng nhau trong ảnh đêm: gộp hay giữ? | **Giữ** từng vật, không gộp box; chồng nhau là bình thường (đúng 4.1.4). | img_024 | CAL-03 |
| 7 | Mức conf tối thiểu để nhận box máy không cần sửa? | **Không đặt**: conf chỉ để xếp thứ tự duyệt. Box `three_wheeler` có conf dưới 0.5 phải đối chiếu từng box với ảnh. | img_003, img_035 | CAL-02 |

**Chưa chốt trong v1.1:** câu 1 (xe đạp điện: `bicycle` hay `motorbike`), phần hàng nhô xa của câu 3 và câu 4 (người ngồi trong xe ba bánh có mui che, ngưỡng "còn thấy ≥ 20%"). Lý do: tôi chưa tick `can_review` hay gặp ca nào về hai câu này trong 4 ảnh gold và 24 ảnh batch, nên chưa có bằng chứng để chốt (riêng câu 3: ví dụ img_008 chỉ có hàng nằm trong khay, không có hàng nhô ra). Các phần này giữ nguyên như v1.0.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_035 | Một xe ba bánh có hai hoặc ba box `three_wheeler`; có box phủ vùng không có xe | YOLOE-26s (ngưỡng 0.15) sinh box trùng và box thừa điểm thấp (0.20–0.48); các box này còn sót sau review | Mỗi vật đúng một box: giữ box ôm sát nhất, xoá các box trùng và box không có xe. Ví dụ img_003: giữ #18, xoá #19 và #20 | **4.1.8 (mới):** một vật một box. Hai box cùng lớp phủ cùng một vật thì xoá box kém. (CAL-01) |
| img_013 | Mask tuk-tuk có người lái và hành khách ngồi | SAM 2.1 base với 1 điểm âm gộp cả người vào mask xe; 3 điểm âm thì mask xe bị vỡ | Mask xe loại người; người ngồi có mask `person` riêng; thiếu phần xe thì thêm điểm dương thay vì chỉ thêm điểm âm | **4.2.5 (mới):** mask xe không gồm người ngồi; mỗi người một mask. (CAL-04) |
| img_013, img_004 | Tuk-tuk hở 3 mặt, khoảng trống giữa khung và sàn lớn hơn 5% diện tích | SAM trả mask đặc cả khối xe, còn 4.2.2 yêu cầu khoét lỗ lớn | Khoảng hở bên trong khung xe ba bánh hở được tính là phần xe (không khoét lỗ), trừ vùng người ngồi | **4.2.6 (mới):** ngoại lệ của 4.2.2 cho xe ba bánh hở. (CAL-04) |
| img_005, img_028 | Cụm xe máy dày gần camera, nhiều box chồng nhau | YOLO26 vẽ từng xe và từng người, nhiều box chồng | Giữ từng vật cao từ 20 px; chỉ dùng `ignore_region` cho cụm từ 5 vật dưới 20 px; vật lẻ dưới 20 px không vẽ | **4.1.6** làm rõ, không đổi ngưỡng 20 px. (CAL-03) |
| img_020 | Xe lớn màu tím-vàng bị SUV che gần hết: bus hay truck? Box xe bị cắt ở mép phải | YOLO26 gán `truck` | Không đổi guideline: áp dụng mục 4.3 (chọn lớp tin nhất, tick `can_review`) và 4.1.3 (vẽ đến mép ảnh) | Không đổi (đây là ca khó của người gán, không phải guideline mơ hồ). (CAL-05) |
| img_014, img_031, img_024, img_006, img_028 | Bẫy `loose` và `missing` lọt khi review | Máy không liên quan (bẫy cài sẵn) | Không đổi guideline: quy tắc 4.1.1 (ôm sát) đã rõ; sửa ở quy trình duyệt (soi 4 cạnh, đếm vật trước khi hiện nhãn máy) | Không đổi (lỗi của người duyệt, ghi ở REPORT mục 4). (CAL-06) |

### Quy tắc mới (nội dung đầy đủ)

- **4.1.8 Một vật, một box.** Với cùng một vật, chỉ giữ một box. Nếu hai box cùng lớp phủ gần như cùng một vùng (IoU lớn, hoặc một box nằm gần như trong box kia) thì giữ box ôm sát nhất và xoá box còn lại. Người ngồi trên xe vẫn có box riêng (4.1.4); đó không phải box trùng vì khác lớp.
- **4.2.5 Người ngồi trong xe.** Mask xe không gồm người lái và hành khách. Mỗi người có mask `person` riêng. Nếu người bị khung xe che một phần thì mask người chỉ phủ phần nhìn thấy.
- **4.2.6 Xe ba bánh hở.** Với xe ba bánh có mui hoặc khung hở các mặt, khoảng hở bên trong khung và sàn xe được tính là phần xe; không khoét lỗ theo 4.2.2. Chỉ khoét vùng người ngồi theo 4.2.5.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 5, 6, 7 và một phần câu 3 của mục 6; thêm 4.1.8, 4.2.5, 4.2.6; làm rõ 4.1.6; câu 1 và 4 chưa chốt | CAL-01 đến CAL-06 |
