# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 537; đối tượng sau duyệt: 523; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 499 | 499/537 = 92.9% | 537 box máy |
| edited (sửa) | 20 | 20/537 = 3.7% | 537 box máy |
| deleted (xoá) | 18 | 18/537 = 3.4% | 537 box máy |
| added (vẽ thêm) | 4 | 4/523 = 0.8% | 523 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 4.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 186 | 1 | 1 | 1 |
| bicycle | 5 | 2 | 1 | 1 |
| motorbike | 228 | 12 | 2 | 2 |
| car | 53 | 0 | 2 | 0 |
| bus | 2 | 0 | 0 | 0 |
| truck | 5 | 0 | 1 | 0 |
| three_wheeler | 20 | 5 | 11 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 41 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 36 | 43 | 5 | 45.6% (KTC 95%: 35–57%) | 87.8% (KTC 95%: 74–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 1/3
  - loose: 1/3
  - wrong_class: 1/3
  - lọt: img_010.jpg (missing), img_034.jpg (loose), img_006.jpg (loose), img_025.jpg (wrong_class), img_024.jpg (missing), img_029.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
