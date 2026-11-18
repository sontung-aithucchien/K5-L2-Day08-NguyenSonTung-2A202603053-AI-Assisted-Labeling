# Báo cáo Day 08 — 2A202603053 - Nguyễn Sơn Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (r01, r02, r05) | person=0.5; motorbike=0.1; bicycle/car/bus/truck=0.3 mặc định | person đạt recall 100%, precision 45% (10 vật gold); motorbike đạt recall 93%, precision 42% (14 vật). Bicycle chỉ có 4 vật nên không chọn riêng; car không đạt recall 90% ở ngưỡng ≤ 0.5 (cao nhất 83%); bus và truck không có vật gold. |
| three_wheeler | YOLOE-26s, prompt mặc định tuk-tuk / cyclo / cargo tricycle (r03, r06) | 0.15 | Recall 100%, precision 78% trên 7 vật gold; giữ được recall mục tiêu, nhưng mẫu nhỏ và precision còn hạn chế. |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- **SAM đã chọn:** SAM 2.1 (`sam2.1_b.pt`), đã chạy thành công cho 3 ảnh offline; không cần đổi model để thực hiện click. Mask khởi tạo được nhưng cần người sửa phần người/hàng hoá lẫn vào.
- Lỗi không sửa được bằng đổi ngưỡng: YOLO26 có thể gọi tuk-tuk/ba gác là `car`/`truck`; dùng YOLOE cho `three_wheeler` và người review phải xác minh lớp, không chỉ chỉnh confidence.
- Box điểm cao mà sai / điểm thấp mà đúng: `model_log.csv` không ghi cặp ảnh-confidence cụ thể đủ để kết luận; không suy đoán sau khi chạy.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan; học viên xác nhận đã xem pre-label trước khi gán xong gold, nên gold bị nhiễm và precision/recall không phải đánh giá độc lập. Ngưỡng chỉ là ước lượng thô, đặc biệt với lớp ít vật.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 499 | 92.9% | 537 box máy |
| edited | 20 | 3.7% | 537 box máy |
| deleted | 18 | 3.4% | 537 box máy |
| added | 4 | 0.8% | 523 đối tượng sau duyệt |

- Điểm yếu rõ nhất là `three_wheeler`: trong 36 box máy của lớp này, 20 accepted, 5 edited và 11 deleted; lớp này chiếm 11/18 box bị xoá. Điều này phù hợp với quan sát ở `model_log.csv`: YOLO26 thường gọi xe ba bánh là car/truck, còn box YOLOE có thể lỏng hoặc bỏ sót vật ở xa. `review_stats.md` không ghi ảnh cụ thể cho từng lỗi lớp, nên không suy ra ảnh ví dụ từ số liệu này.
- So với gold: số đo script là precision 45.6% (36/79 dự đoán là TP), recall 87.8% (36/41 đối tượng gold được tìm thấy), nhưng **không dùng như đánh giá độc lập** vì gold bị nhiễm (đã xem pre-label trước khi gán xong) và ngưỡng cũng được chọn trên cùng 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 6/12 bẫy (50%; mục tiêu 80%). Bắt được: extra 3/3, missing 1/3, loose 1/3, wrong_class 1/3. Bị lọt: missing ở `img_010.jpg` và `img_024.jpg`; loose ở `img_034.jpg` và `img_006.jpg`; wrong_class ở `img_025.jpg` và `img_029.jpg`.
- `review_stats.md` không lưu lại người duyệt đã bỏ bước checklist nào, nên không thể quy nguyên nhân cá nhân cho từng bẫy lọt. Số missing và wrong_class lọt cho thấy lượt quét độc lập và lượt rà lớp cần được kiểm tra chặt hơn ở lần sau.
- Accepted 92.9% chỉ nói rằng phần lớn box máy được giữ nguyên; nó không chứng minh đã tìm đủ vật bị bỏ sót hay sửa hết lỗi. Việc chỉ bắt 50% bẫy cho thấy vẫn có lỗi lọt qua review, nên cần quét ảnh trước khi nhìn nhãn máy và kiểm tra riêng vật thiếu, box lỏng, sai lớp.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): ước tính theo timebox của GUIDE, không bấm giờ: gán tay 25/4 = 6.25 phút/ảnh; duyệt 72/24 = 3 phút/ảnh. Đây không phải phép đo thực tế nên không dùng để khẳng định mức tiết kiệm. Ngoài ra, gold bị nhiễm và đã quen cảnh trước khi review, nên so sánh càng yếu.
- Kết luận: **có, nhưng chỉ dùng làm bản nháp cần người duyệt**. Có 499/537 box máy được giữ nguyên (92.9%), nhưng chỉ 6/12 bẫy được phát hiện; QC còn thấy lỗi ở 2/5 ảnh mẫu. Cần đặc biệt kiểm tra `three_wheeler`; thời gian ước tính không chứng minh được tiết kiệm thực tế.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Với mỗi ảnh, ẩn nhãn máy và quét toàn ảnh trước; chỉ sau lượt quét độc lập mới hiện nhãn máy để đối chiếu.
  2. Duyệt theo cùng thứ tự cho cả 24 ảnh: bỏ sót → vật thừa → sai lớp → box lỏng; không bỏ qua bước nào.
  3. Sau mỗi 10 ảnh nghỉ ngắn; khi quay lại, ưu tiên kiểm tra vật nhỏ/xa và `three_wheeler`, đồng thời ghi lại mọi bẫy hoặc lỗi còn lọt để đối chiếu checklist.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có lẫn một phần người lái/hành khách vào mask xe; người vẫn cần nhãn riêng. | Mask phủ xe và mui nhưng xử lý chưa sạch các khoảng trống lớn trong cabin; click âm hiện tại chưa tách người hoàn toàn. | `tuktuk_img013.jpg` |
| img_002 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Lẫn một phần vùng người ngồi trong cabin; hàng gắn trên xe nằm trong vùng mask xe. | Mask bắt được mui và thân tuk-tuk; cabin hở còn lẫn vùng người/nền. | `tuk_tuk_img002.jpg` |
| img_009 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có lẫn một phần túi/hàng che xe; không có người lái rõ trong mask này. | Thân và bánh xe thấy được được bắt khá tốt, nhưng phần xe bị hàng che không thể khôi phục; mask lẫn một phần hàng hoá. | `loaded_motorbike_img009.jpg` |

- Đối chiếu guideline mục 4.2: SAM tạo mask khởi đầu nhưng chưa tách nhất quán các lỗ lớn ở cabin khung xe (4.2.2) hoặc phần người chạm/ở trong xe (4.2.4); cần thêm click âm và sửa biên bằng mắt. Mui và phần khung/thùng gắn liền xe được giữ trong mask. Với hàng hoá nhô ra ở img_009, guideline v1.0 còn để mở (mục 6.3), nên cần chốt quy tắc ở calibration thay vì tự coi hàng là một phần xe.
 - *(Tuỳ chọn)* Interactor SAM trong CVAT: không thực hiện; dùng `sam2_points.py` offline nên chưa có so sánh trực tiếp với Interactor.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất là CAL-02: làm rõ hàng hoá nhô khỏi xe có thuộc box/mask hay không.
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: xác minh lớp xe ba bánh bằng vật thật; loại hàng rời nhô quá 20% khỏi box/mask xe; quét/đếm ảnh độc lập và rà riêng missing, wrong_class.
- Nguyên nhân gốc: lỗi lớp của máy, câu hỏi v1.0 còn mở về hàng hoá, và bước review chưa phát hiện hết bẫy; không gộp các nguyên nhân này thành một lỗi duy nhất.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42), 110 đối tượng; kiểm tra hình ảnh được hỗ trợ bởi AI, không phải người kiểm độc lập.
- Lỗi theo **đối tượng**: 3/110 = 2.7% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2%).
- Top lỗi: `thua` 2 đối tượng (#5, #9 trùng xe ở img_003); `sai_vi_tri_kich_thuoc` 1 đối tượng (#442 ôm hàng nhô xa ở img_031).
- **Giới hạn mẫu nhỏ** (`qc_report_auto.md`): cận trên Wilson 95% cho tỉ lệ ảnh lỗi là 77%; mẫu 5 ảnh không chứng minh được ngưỡng lỗi 2%.
- Script kiểm được cấu trúc, lớp, kích thước/biên box và tổng hợp số liệu; không tự xác định vật thật, box ôm sát, vật cả máy lẫn người cùng bỏ sót, hoặc việc tuân thủ checklist.

**Quyết định: Rework**, ngày 2026-11-18. Lý do: 2/5 ảnh mẫu có lỗi (40.0% > 2%) và chỉ bắt được 6/12 bẫy (50% < 80%).
Việc cần làm tiếp: sửa hai box trùng ở img_003 và box img_031 trong CVAT theo CAL-02, export `submission/labels/qc_fixed.json`; rà lại batch do tỉ lệ bắt bẫy thấp, rồi chạy lại `build_release.py` và `qc_sample.py --n 5 --seed 42 --force`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có, Copilot hỗ trợ diễn giải số liệu/soạn báo cáo và kiểm tra trực quan các mask, ảnh QC; các chỉ số được đối chiếu với script và file đầu ra.
- Gold có bị nhiễm không: có; học viên xác nhận đã xem pre-label trước khi gán xong, nên kết quả trên gold không phải đánh giá độc lập.
- Máy / GPU / CVAT dùng: Windows, chạy model với CPU theo `model_log.csv`; CVAT Docker v2.80.0. PowerShell không dùng dấu `\` để nối dòng lệnh.
- Dùng cache `data/cache/`: không thấy dùng trong `model_log.csv`; có các lần chạy model local trên CPU.
- Lỗi / bug script đã gặp: lệnh nhiều dòng dùng dấu `\` gây lỗi cú pháp trong PowerShell; chạy một dòng hoặc dùng dấu backtick.
- Một điều làm khác lần sau: ghi thời gian ngay khi bắt đầu/kết thúc từng bước và hoàn tất lượt quét độc lập trước khi xem pre-label.
