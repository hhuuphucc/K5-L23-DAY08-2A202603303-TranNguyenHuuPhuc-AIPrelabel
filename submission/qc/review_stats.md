# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 520; đối tượng sau duyệt: 523; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 467 | 467/520 = 89.8% | 520 box máy |
| edited (sửa) | 10 | 10/520 = 1.9% | 520 box máy |
| deleted (xoá) | 43 | 43/520 = 8.3% | 520 box máy |
| added (vẽ thêm) | 46 | 46/523 = 8.8% | 523 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 36.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 264 | 2 | 7 | 6 |
| bicycle | 6 | 0 | 1 | 2 |
| motorbike | 135 | 1 | 3 | 9 |
| car | 42 | 2 | 7 | 20 |
| bus | 3 | 0 | 0 | 3 |
| truck | 1 | 0 | 5 | 3 |
| three_wheeler | 16 | 5 | 20 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 79 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 54 | 20 | 25 | 73.0% (KTC 95%: 62–82%) | 68.4% (KTC 95%: 57–78%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - wrong_class: 3/3
  - loose: 0/3
  - missing: 1/3
  - lọt: img_014.jpg (loose), img_031.jpg (loose), img_006.jpg (missing), img_024.jpg (loose), img_028.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
