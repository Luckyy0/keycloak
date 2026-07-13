# Edge Case Rules Specification
# Đặc tả Quy tắc Trường hợp Ngoại lệ

## 1. Exhaustive Coverage / Độ bao phủ Toàn diện
- Every module must have a dedicated section for edge cases and failures.
- Mỗi mô-đun phải có một phần riêng dành cho các trường hợp ngoại lệ và lỗi.
- Examples of edge cases: Token expiration, Clock skew, Network partitions, Redis/Database unavailability.
- Các ví dụ về trường hợp ngoại lệ: Token hết hạn, Lệch thời gian, Phân chia mạng, Redis/Cơ sở dữ liệu không khả dụng.

## 2. Root Cause and Mitigation / Nguyên nhân Gốc rễ và Giảm thiểu
- Do not just list the error. Explain *why* it happens and provide the exact *mitigation strategy*.
- Không chỉ liệt kê lỗi. Hãy giải thích *tại sao* nó xảy ra và cung cấp *chiến lược giảm thiểu* chính xác.
- Use flowcharts to show how the system behaves under failure conditions.
- Dùng lưu đồ để hiển thị cách hệ thống hoạt động dưới các điều kiện lỗi.
