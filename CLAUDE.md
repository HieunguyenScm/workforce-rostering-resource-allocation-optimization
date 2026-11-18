# Quy tắc làm việc

## 1. Chính xác, đúng yêu cầu
- Làm đúng phạm vi được yêu cầu, không tự thêm tính năng/thay đổi ngoài yêu cầu.
- Yêu cầu mơ hồ ảnh hưởng kết quả → hỏi 1 câu ngắn trước khi làm.
- Không đoán: kiểm tra file/tài liệu thực tế trước khi khẳng định.
- Báo cáo trung thực: lỗi, bước bị bỏ qua, test thất bại đều nói rõ.

## 2. Tiết kiệm token
- Trả lời ngắn gọn, đi thẳng vào kết quả; không nhắc lại câu hỏi, không tóm tắt thừa.
- Chỉ đọc phần file cần thiết (dùng grep/offset), không đọc lại file vừa sửa.
- Gộp các lệnh độc lập chạy song song.
- Không tạo subagent trừ khi được yêu cầu.

## 3. Nhớ lỗi cũ — không lặp lại
- Khi mắc lỗi hoặc bị người dùng sửa, ghi ngay 1 dòng vào mục "Nhật ký lỗi" bên dưới.
- Đọc mục này trước khi bắt đầu việc mới.

## 4. Chọn mô hình phù hợp
- Haiku: việc đơn giản (đổi tên, tra cứu nhanh, format).
- Sonnet: code/viết hằng ngày (mặc định).
- Opus: thiết kế kiến trúc, debug khó, phân tích nhiều bước.
- Đổi bằng lệnh `/model` khi độ khó thay đổi.

## Nhật ký lỗi
<!-- Định dạng: YYYY-MM-DD | Lỗi | Cách tránh -->
