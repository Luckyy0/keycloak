# Chapter 35: Bắt Mạch Bắt Bệnh (Troubleshooting & Debugging)

## Giới thiệu (Introduction)
Chào mừng bạn đến với Phòng Cấp Cứu của Keycloak. Khi một hệ thống Identity Access Management sụp đổ hoặc từ chối phục vụ, nó không bao giờ nói rõ ràng "Tôi bị sai mật khẩu của Database". Nó thường ném vào mặt bạn một đống lỗi mập mờ kiểu: `HTTP 401 Unauthorized`, `invalid_grant`, hoặc đáng sợ hơn là `CORS Policy Blocked`.
Chương này không dạy bạn code thêm tính năng, mà dạy bạn "Kỹ Năng Đọc Lỗi". Đây là kinh nghiệm xương máu được đúc kết từ hàng ngàn giờ khắc phục sự cố trên Production, giúp bạn nhìn một dòng báo lỗi là biết ngay "À, quên cấu hình Redirect URI" hay "À, chứng chỉ SSL hết hạn".

## Mục lục (Table of Contents)

### Module 1: Từ Điển Bệnh Lý (Symptom Dictionary)
*   **Lesson 1: OAuth2 & OIDC Errors:** Bắt mạch các căn bệnh chết người của chuẩn OAuth2: `invalid_client` (Sai mật khẩu Client), `invalid_scope` (Đòi hỏi quá đáng), `invalid_grant` (Mã hết hạn), `redirect_uri_mismatch` (Khách lạ lạc đường).
*   **Lesson 2: HTTP & Web Errors:** Giải quyết những nỗi ám ảnh của Frontend Web: `401 Unauthorized` (Vô danh), `403 Forbidden` (Đã biết tên nhưng cấm cửa), `CORS Blocked` (Tường lửa trình duyệt), `CSRF Token Mismatch`.
*   **Lesson 3: System & Crypto Errors:** Bắt lỗi hạ tầng và mã hóa: Lệch múi giờ (`Clock Skew`), Sự cố chứng chỉ `SSL/TLS Handshake`, và Ám ảnh `Token Expired`.

### Labs & Thực hành (Labs)
*   **Lab 1:** Phòng Khám Đa Khoa: Chúng ta sẽ dựng một môi trường cố tình Bị Lỗi (Sai Secret, Sai URI, Lệch Giờ). Nhiệm vụ của bạn là xem Log và tìm cách Sửa Lại (Fix Bug) cho đến khi Ứng dụng Frontend Login được thành công.

## Bắt đầu từ đâu? (Where to start?)
Nếu bạn đang khóc vì lỗi Token, hãy lao ngay vào [Lesson 1: OAuth2 Errors](Module-1-Concepts/Lesson-1-OAuth2-Errors.md).
