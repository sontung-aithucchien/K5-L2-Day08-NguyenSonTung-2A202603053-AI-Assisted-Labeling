# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603053 - Nguyễn Sơn Tùng

> Chỉ ghi phần bổ sung/điều chỉnh so với guideline v1.0. Quy tắc mới dựa trên lỗi mô hình, bẫy lọt và câu hỏi mở được chốt trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng hoá nhô ra khỏi xe ba bánh/xe đạp ảnh hưởng box và mask thế nào? | Giữ khung xe và giỏ/giá chở hàng cố định. Hàng rời nhô quá 20% chiều dài xe không thuộc box/mask của xe. Chỉ gán phần xe nhìn thấy, không suy đoán phần bị hàng che; nếu không phân biệt được khung và hàng thì tick `can_review`. | img_009, img_031 | CAL-02 |

Các câu hỏi mở khác (1, 2, 4, 5, 6, 7) chưa đủ bằng chứng để chốt thống nhất; giữ nguyên v1.0 và tiếp tục đánh dấu `can_review` khi gặp ca không chắc.

## 2. Ca khó mới

| Ảnh | Tình huống | Máy/lỗi quan sát được | Quyết định và quy tắc bổ sung | `case_id` |
|---|---|---|---|---|
| img_013 | Tuk-tuk có người ngồi, dễ lẫn giữa xe và người | YOLO26 có thể gán xe ba bánh thành `car`/`truck`; mask theo box sai không sửa được lớp | Chốt lớp từ vật thật, không theo nhãn máy: xe ba bánh là `three_wheeler`. Không đếm được bánh thì tick `can_review`. Kiểm tra lớp trước khi dùng box/mask máy. | CAL-01 |
| img_024, img_025, img_029 | Bẫy missing và wrong_class bị lọt | Review chỉ bắt 6/12 bẫy; missing lọt ở img_024, wrong_class lọt ở img_025 và img_029 | Trước khi hiện nhãn máy, quét ảnh và đếm vật theo lớp; sau đó rà riêng vật bị bỏ sót và sai lớp. Ảnh đông quét theo góc phần tư; không lấy số box máy làm số vật chuẩn. | CAL-03 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (`case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ xử lý hàng rời nhô khỏi xe; yêu cầu xác minh lớp xe ba bánh độc lập với model; thêm lượt quét/đếm trước khi xem nhãn máy và rà missing/wrong_class riêng | CAL-01, CAL-02, CAL-03 |
