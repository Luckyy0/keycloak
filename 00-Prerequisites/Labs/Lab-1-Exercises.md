# Lab 1: Bài Tập Tổng Hợp Chương Tiền Điều Kiện (Prerequisites)

> [!NOTE]
> **Category:** Labs & Troubleshooting (Thực hành & Bắt lỗi)
> **Goal:** Bài Sát hạch Cấp độ Chuyên gia kết thúc Chương `00-Prerequisites`. Áp dụng toàn bộ 37 bài học lý thuyết về Cryptography, Web Security và Protocols vào các Kịch bản Kiến trúc thực tế.

---

## 1. Tình huống Kiến trúc (Architecture Scenarios)

**Tình huống 1: Thiết kế Token cho Hệ thống Ngân hàng Vĩ mô**
Công ty BankCorp chuẩn bị xây dựng hệ thống Mobile Banking thế hệ mới với 50 Microservices. 
GĐKT (CTO) đưa ra yêu cầu: 
1. Không được phát sinh độ trễ (Latency) khi gọi giữa các Microservices.
2. Tránh hiện tượng OOM (Tràn RAM) khi parse dữ liệu.
3. Bảo mật Thông tin Cá nhân Dữ liệu Bệnh án/Tín dụng tuyệt đối ngay cả khi Token lọt vào tay IT mạng.
4. Có cơ chế chặn lệnh chuyển tiền 2 lần do mạng chập chờn.

*Nhiệm vụ của bạn: Hãy chọn công nghệ cấu thành nên chiếc Chìa khóa (Token) cho BankCorp dựa trên các chuẩn đã học.*

**Đáp án phân tích (Dành cho Kiến trúc sư):**
- Để thỏa mãn (1): Dùng chuẩn **JWT (JSON Web Token)** để Microservices tự xác minh chữ ký (Stateless), không cần chọc về Database sinh trễ.
- Để thỏa mãn (2): Tuyệt đối không dùng cấu trúc XML/SAML nặng nề. Phải parse JSON dạng luồng (Stream) thay vì DOM nếu payload lớn, nhưng tốt nhất là áp dụng nguyên tắc Cắt tỉa (Token Bloat prevention) giữ JWT dưới 4KB.
- Để thỏa mãn (3): JWT thông thường (JWS) dạng Base64 ai cũng đọc được -> Vi phạm. Bắt buộc phải áp dụng **JWE (JSON Web Encryption)** dùng Nested JWT (Ký RS256 xong bọc bằng AES/RSA).
- Để thỏa mãn (4): Triển khai tiêu chuẩn **Idempotency** bằng cách gài thêm thuộc tính `jti` (JWT ID) hoặc Header `Idempotency-Key` (Nonce), kiểm tra qua Redis để chặn **Replay Attack**.

---

## 2. Bắt lỗi Hệ thống (Troubleshooting Challenges)

**Thử thách 1: Lỗi Chữ ký Xuyên Không**
Đội Frontend (React) báo cáo: *"Token do Keycloak cấp có `exp` là 1 tiếng nữa mới hết hạn. Nhưng hễ gửi lên API Resource Server (Spring Boot) thì toàn bị báo lỗi `HTTP 401: Token expired or Not Before (nbf) failed`. Lạ nhất là lúc thì bị, lúc thì gọi được."*

*Nhiệm vụ: Lỗi này do Frontend gửi sai chuẩn, do Cấu hình Keycloak, hay do Tầng Cơ sở hạ tầng? Cách khắc phục?*

**Đáp án phân tích:**
- Hiện tượng "Chữ ký xuyên không" hoặc sai lệch `nbf` (Not Before) và `exp` giữa 2 máy chủ thường xuyên xảy ra do **Clock Skew (Lệch đồng bộ thời gian)** (Học ở Bài 35).
- Nguyên nhân: Máy chủ Keycloak (Tạo token) chạy nhanh hơn Máy chủ API (Xác minh token) khoảng vài phút. Keycloak đóng cái dấu: "Token này chỉ được xài sau 10:05 (`nbf=10:05`)". API nhận Token lúc đồng hồ của nó mới là 10:03. API sẽ từ chối Token vì "Chưa tới giờ xài". Lỗi chập chờn do Kubernetes Scale Pod ngẫu nhiên vào các Node bị lệch giờ.
- Cách fix: 
  - Khắc phục Gốc: Cài đặt NTP (Network Time Protocol) đồng bộ hóa đồng hồ phần cứng (Hardware clock) trên mọi máy chủ Microservices.
  - Khắc phục Ngọn: Cấu hình thư viện JWT của Spring Boot mở rộng `Allowed Clock Skew` (Dung sai thời gian) lên 60 giây - 120 giây.

---

## 3. Phân tích Dấu vết (Log Analysis)

**Cảnh báo Đỏ từ Tường lửa (WAF Alert):**
Hệ thống báo động có cuộc tấn công trên Endpoint đọc File cấu hình XML của Khách hàng:

```text
[2026-05-20 15:30:10] WARN: XML Parsing initiated.
[2026-05-20 15:30:11] ERROR: System resource exhausted. RAM spike 15GB detected.
[2026-05-20 15:30:11] FATAL: JVM OutOfMemoryError. Application crash.
```

Payload thu thập được từ Hacker:
```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 ... (Lặp lại đến lol9)
]>
<data>&lol9;</data>
```

*Nhiệm vụ: Định danh đòn tấn công và đưa ra cách vá lỗi ở Tầng Code (Java).*

**Đáp án phân tích:**
- **Định danh:** Đây là đòn tấn công kinh điển **XML Bomb (Billion Laughs Attack)**. Khác với XXE (để cướp ổ cứng), đòn này thuộc nhóm Từ chối dịch vụ (DoS) (Học ở Bài 29). Nó lạm dụng khả năng Định nghĩa Thực thể (DTD) lồng nhau đệ quy, biến 1 Kilobyte XML thành hàng tỷ chữ "lol" đánh nổ RAM máy chủ.
- **Cách vá lỗi Java:** Chặn tuyệt đối cấu hình DTD của JAXP (DocumentBuilderFactory).
  ```java
  factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
  factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
  factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
  ```

---

## 4. Bài tập Thiết kế Kiến Trúc (Design Exercises)

**Yêu cầu:** Bạn hãy tư vấn Kiến trúc bảo mật tĩnh cho một Web App Single-Page (ReactJS) chứa Dữ liệu Y tế, dựa trên Bộ Headers bảo vệ tiêu chuẩn (Web Security Headers).

**Bản Thiết Kế Đề Xuất:**
Trên Nginx Gateway (Hoặc thẻ Meta của Web), CẤU HÌNH BẮT BUỘC 5 TẤM KHIÊN:

1. **HSTS (Strict-Transport-Security):** Ép buộc mọi tương tác qua HTTPS, chặn đứng đòn bẻ khóa SSL Stripping khi bác sĩ mở Web ở quán Cafe.
2. **CSP (Content-Security-Policy):** `default-src 'self'`. Ngăn chặn đòn **XSS** bùng phát, cấm Trình duyệt thực thi mã JS lạ (Thứ có thể chôm Token hay Session Cookie).
3. **CSP `frame-ancestors` (Hoặc X-Frame-Options):** `frame-ancestors 'none'`. Chặn đứng đòn **Clickjacking** (Hacker lừa bác sĩ bấm nút Xóa Bệnh án bằng Iframe tàng hình).
4. **Referrer-Policy:** `strict-origin-when-cross-origin`. Tránh việc đường Link Reset Password của Bệnh nhân bị rò rỉ (Referer Leakage) ra cho bên thứ ba (Ví dụ Facebook/Google Analytics).
5. **Cookie Security (Cho JWT nếu lưu ở Cookie):** Đánh cờ `HttpOnly` (Chống XSS), cờ `Secure` (Chống MitM), và đặc biệt cờ `SameSite=Lax/Strict` để bẻ gãy đòn **CSRF** mượn dao giết người.

---

> [!NOTE] 
> **LỜI KẾT CHƯƠNG:**
> Chúc mừng bạn đã vượt qua Module khốc liệt nhất về Nền tảng Công nghệ lõi. Mọi Khái niệm về Mã hóa (RSA/AES/JWT), Dữ liệu (JSON/XML/REST) và Bảo mật (XSS/CSRF/Owasp Top 10) chính là những "Binh khí" bắt buộc phải nhuần nhuyễn trước khi bước ra chiến trường.
> Kể từ Module tiếp theo, chúng ta sẽ cầm những Binh khí này bước vào Cuộc chiến Thượng tầng: **Giao thức OAuth 2.0 và OpenID Connect (OIDC)**.
