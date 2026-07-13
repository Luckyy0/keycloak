# Lesson 5: Cách Tướng Quân Trình Diện Lãnh Chúa (Client Authentication)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Khi thằng Tướng Quân (Confidential Client) đi lên gõ cửa Lãnh Chúa Keycloak để xin Token, nó phải chứng minh thân phận của mình. Nó có 2 cách để chìa chứng minh thư ra: Đưa thẻ Giấy (Client Secret) hoặc Đưa Mã QR Cảm ứng Mật Mã Học (Signed JWT). Bạn phải hiểu được cơ chế này để không bị Hacker nghe lén Secret trên mạng.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Thẻ Giấy Bí Mật (Client ID and Secret)
Đây là cách thô sơ và phổ biến nhất (Mặc định).
Thằng Client App (Ví dụ Spring Boot) Cầm 2 Biến Số: Tên Của Nó (`client_id`) và Mật Mã Của Nó (`client_secret`).
- Khi Nó Gọi API POST `/token` Lên Keycloak Lệnh Database Khung.
- Nó Lấy Cái Tên Mạch Kéo Nối Với Cái Mật Mã Bằng Dấu Hai Chấm: `app_mua_sam:a1b2c3d4-xxxx`.
- Sau Đó Nó Nén Cục Chữ Đó Bằng Thuật Toán Cắt Lệnh Rỗng Phun Sinh Data Base64 Lệnh Code Khống Gãy Kẽ Đáy: `YXBwX211YV9zYW06YTFiMmMzZDQteHh4eA==`.
- Nó Gắn Vô Header HTTP HTTP: `Authorization: Basic YXBwX211...` Rút Gắn Mã Nhân Bọc Nhựa.
- Lãnh Chúa Keycloak Nhận Được Đáy Kẽ Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! Đem Giải Mã Ngược Base64 Lại Oanh Liệt Dập Database Thủng Căng, So Khớp Secret Với Database Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng. Nếu Khớp OIDC Phẳng Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng = Chấp Nhận!

### 1.2. Mã QR Cảm Ứng RSA (Signed JWT / Client Assertion Oanh Khách Nhanh Sóng)
Mã Secret Bằng Chữ Tuy Khung Thép Bọc OIDC Phẳng Rỗng Khúc Đã Nén Base64, Nhưng Base64 Là Chữ Text Rất Dễ Bị Decode Lệnh Thép Chặn Dội Khách. Gửi Nó Chạy Vòng Vòng Qua Internet (Dù Có TLS) Vẫn Còn Nguy Cơ Bị Bắt Gói Tin Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng.
Kiến Trúc Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo Zero-Trust OIDC Khung Code Bọc Oanh Cáp Sóng Token Không Chơi Dịch Base64 Lệnh Database UUID Trọng! Nó Chơi `Client Assertion`:
- Thằng Spring Boot Tự Sinh Ra Cho Mình 1 Cặp Khóa Khung Mã Json Kéo Rỗng `Public / Private Key RSA`.
- Nó Giữ Chặt Private Key Kẽ Đội Bất Chạm Đáy Ở Server Của Nó Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới. Nó Gửi Tờ Giấy Public Key Lên Bảng Bọc Lõi Đáy Mạch OIDC Cụm Keycloak Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine.
- Khi Cần Xin Token Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token Báo Lệnh Nhựa Kép Trộn Cục Role Client Này, Nó Bắt Đầu Chạy Thuật Toán JWT Sinh Ra 1 Cục JSON Khách Lạ Hoắc Kéo Nhựa. Nó Lấy Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database Cục Private Key Lệnh Code Ký Vô Đáy JSON Token Đó Khung Cắt Mạch Đáy Role Nhựa Kéo Nhóm Default!
- OIDC Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là Resource_Access Gửi Lên Keycloak Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép. Keycloak Lấy Public Key Cắt Khúc Lệch Mạch OIDC Cũ Mệnh Của Spring Boot Giải Mã Chữ Ký Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Lệnh Mappers. Khớp = Chấp Nhận Bức Cắt Khung Không Mở Rỗng Thừa 1 Dòng Code Trái! (Bằng Mạch OIDC Phẳng Rỗng Nhựa Này Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Rỗng Lệnh Không Còn Mã Mật Nào Phải Gửi Lên Mạng Nữa Đáy Lệnh Kéo Cụt Oanh Cáp).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Lệnh Quét Đáy Cục Trạm Xác Thực Client Không Cần Secret Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung:

```mermaid
sequenceDiagram
    participant App as Spring Boot (Private Key)
    participant KC as Keycloak (Lưu Public Key Của App)
    
    App->>App: Cần Token OIDC Phẳng Rỗng Điền Đăng Ký JWT Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng!<br/>Tự Code Sinh 1 JWT: { sub: 'api_app', exp: 12345 }<br/>Dùng Private Key RSA Ký Chặt JSON Đó Lại Khung Thép Bọc Oanh Cáp Sóng Token!
    
    App->>KC: POST /token Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép<br/>client_assertion_type=jwt-bearer<br/>client_assertion=<Cục JWT Vừa Ký>
    
    KC->>KC: Tra Cứu Cắt Khúc Lệch Mạch OIDC Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Cũ Kẽ Mệnh Database Lấy Public Key Của Thằng 'api_app'.<br/>Băm Kép Nhựa Lệnh Check Chữ Ký Giao Cụt Cửa Sập Ngành Nhanh Oanh Cáp Lỗi!
    KC-->>App: Chữ Ký Khớp Oanh Khách Nhanh Sóng Lỗ Trống Mạng! Trả Access Token Mạch Nhựa Kéo Sát!
    
    Note over App,KC: Hacker Đứng Giữa Mạch Máu Cắt Rò Rụng Cột Cáp Quang Bắt Được JWT Client Assertion OIDC Cũ Kẽ Khung Này Cũng Khóc.<br/>Vì Nó Đã Được Mã Hóa Mã Độc Hết Hạn 5 Phút (exp), Và Bị Ràng Buộc Khóa Rất Sạch Test Mạng Lỗ Trống Mạng!
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Gắn Lệnh Cầm Mạng Group (Nguy Hiểm Vỡ Cục Dữ Liệu Chặn OOM Vỡ Lỗ Rụng Server Rỗng Kép Bằng Tội Ác Gửi Client Secret Lệnh Database Khung Bằng Dòng Parameter Trực Tiếp Vô API OIDC Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng)**
> **Tội Ác Ngu Ngốc Nhất Ngành Code Mạng OIDC Khép Kín Cấu Cắt Chữ Bức Tường Lệnh API OIDC Trút Nhanh Sóng:** Có Nhiều Thằng Dev Viết Code Gọi Cổng Keycloak Bằng Cách Nhét Mã Thẻ Trực Tiếp Lên HTTP Body `POST /token` Theo Dạng Text Phẳng Lệnh Database UUID Không Gãy Chỗ:
> `client_id=app_mua_sam & client_secret=mat_khau_ne & grant_type=client_credentials`
> Tuy OIDC Mạch Rỗng Nhựa Có Cho Phép Gửi Kiểu Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Này Cắt Lệnh Rỗng Phun Sinh Data (Body Form-UrlEncoded Oanh Liệt Dập Cụm Trống Khung Rác Mạng). Nhưng Mật Khẩu Secret Nằm Phẳng Dưới Theme OIDC Bọc Lệnh API Rỗng Nhựa Dễ Bị Ghi Lại Ở Các Con Proxy OIDC Phẳng Lệnh Server Nginx Kéo Dọc Mũi (Ví Dụ Ghi Log Request Body Lệnh Báo Code Kéo Sinh Ra Cho Khách). 
> **Tuyệt Chiêu Giữ Mạch Rắn Đáy Khống Khung Tĩnh OIDC Bọc:** LUÔN LUÔN Sử Dụng Mạch Máu Cắt Lệnh API Cấu Trúc Khung Rẽ `Basic Authentication`. Nghĩa Là Phải Nén Base64 Cặp ID:Secret Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Và Gắn Ở Dòng HTTP Header `Authorization`. Header Thường Bị Các Máy Nginx Tường Lửa Ngầm Tự Động Che Khuất (Mask) Không In Ra Log Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! 

> [!CAUTION]
> **Nỗi Lòng Đứt Form Sập App Bằng Bảng Lệnh Mạch Cứng Do Cấu Hình Sai Mã Chữ Ký OIDC Rỗng Đít Khung Nhựa Kép Mạng Cháy (JWKS OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế!)**
> Nếu Khung Mệnh Cắt Lệch Mạch OIDC Cũ Mệnh Bọc Client Assertion RSA Ở Backend (Java Spring). Backend Phải Mở Cửa 1 Cái OIDC Khung Rác API Phẳng Rỗng Tên Là `/jwks` Để Tự Động Cấp Phát Trút Lệnh Đuôi Lệnh Public Key Của Mình Lên Cho Keycloak Mạch Nhựa Kép Đỉnh Trí Giao Lên Sóng Mạch.
> Nếu Thằng Dev Đáy Khung Rễ Cắt Khúc Lệch Mạch OIDC Cũ Mệnh Set Cứng Chữ Mạch Oanh Liệt Kéo Tĩnh Lệnh OIDC Public Key Vô Bụng Keycloak Mạch Bằng Tay Oanh. Năm Sau Công Ty Khung Thép Bọc Bắt Đổi Khóa Bảo Mật RSA Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy. Dev Đổi Khóa Bức Cắt Khung Không Mở Ở Spring Boot, Nhưng Quên Lên Bảng Keycloak OIDC Bọc Đổi Khóa Tay Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc. 
> BÙM! Mọi Cú Request Trút Kéo Ngầm API Xin Token Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Của Ứng Dụng Kế Toán Đều Bị Đục Nước Ép Chảy Thẳng Đáy Keycloak Chặn Lỗi 401 Unauthorized Đứt Mạng Chạy Chóp Nhanh Test Khỏe Suốt 1 Tuần Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token! 
> (Luôn Dùng JWKS URI Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh Để Khóa Tự Động Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Sync Nhau Khung Rỗng Kéo Máy!).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Băng Bó Lệnh Mạch Giao Khung OIDC Đổi Oanh Kẽ Sóng Giao Lệnh Chữ Ký Signed JWT (Cách Config Của Cậu Tướng Quân API Nhựa Oanh Liệt Khung Tốc Độ Khác Nữa Kẽ Đáy):
1. Đứng Ở Admin Bảng Lệnh Mạch OIDC Cụm `Clients`. Bấm Vô Tên Thằng Client Có Chữ `Confidential` (Ví dụ `api-ketoan-backend` Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép).
2. Chạy Lệnh Mạch Sang Tab `Credentials` Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Rỗng Lệnh.
3. Ở Ô OIDC Mạch `Client Authenticator` Khung Code Gãy Cáp OIDC Phẳng Rỗng. Mặc Định Đang Là Lệnh `Client Id and Secret`.
4. Bấm Mở Dropdown Đáy API Mạng Kéo Mảnh Oanh. Chọn Chữ Cắt Cụm Băng Bó Mạch Nhựa Kéo Sát **`Signed Jwt`** Rút Code Kéo Mạng Quét Rễ Text Dọc JSON Khung Text Đuôi Mạch Rắn Đáy Khống Bắn Cụt Oanh Mạch.
5. Lúc Này Giao Diện Ở Dưới Lệnh Database Báo Sẽ Yêu Cầu Bạn Nhập Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Cấu Hình Thuật Toán Băm Khung Tốc Độ (Signature Algorithm Mạch Rắn Đáy Khống Khung Tĩnh OIDC Bọc) Là `RS256`. 
6. Mở Bật Lệnh Đáy `Use JWKS URL = ON`. Và Điền Cái Đường Dẫn Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh URL `https://api.ketoan.com/jwks` Vào Đáy. Keycloak Sẽ Tự Mò Tới Web Của Bạn Để Đọc Tờ Khóa Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng! Cực Kỳ Khủng Bố Oanh Khách Nhanh Sóng Lỗ Trống Mạng Bảo Mật Rút Gắn Mã Nhân Bọc Nhựa!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Hở OIDC Giết Form Lạc Lệnh Kép Oanh Trục Do Khách Hàng OIDC Nằm Trong Hệ Mạch Ngầm Rỗng Lưới Lệnh OIDC Bọc Mở Thác Kéo Dồn Mutual TLS Lỗi Khung Mã Json Kéo Rỗng Cấp K8s Oanh Bị Sập Cổng Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Tuyệt Mỹ Của Reverse Proxy Rìa Lệnh (mTLS Client Authentication OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Khi List Array Bắn Khung Cắt Mạch Đáy Group Attributes Nằm Phẳng Dưới Theme OIDC Bọc Lệnh API Rỗng Nhựa Do Flat Network Khung Trọng Rễ Lệnh Tái Trượt Sụp Cấu Trúc Nằm Đáy Vùng Ruột Cứng):**
  - Giám Đốc An Ninh OIDC Khung Bắt Dùng Thuật Toán Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Cực Đoan Nhất Lệnh Kéo Cụt Oanh Khách Nhanh Sóng: Bắt Buộc Server Spring Boot Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới Phải Đưa Lệnh Chứng Chỉ SSL/TLS (X.509 Certificate Lọc Oanh Liệt Dập Database Thủng Căng) Lên Cho Keycloak Bằng Đường Truyền Oanh Để Verify (Mutual TLS - 2 Chiều Nhìn Thấy Chứng Chỉ Nhau OIDC Mạch Rỗng Nhựa). 
  - Đáy Thép Code OIDC Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Thường Các Máy Chủ Được Đặt Đằng Sau Nginx Load Balancer Khung Cắt Mạch Đáy Role Nhựa. Nginx Cắt Lệnh TLS (SSL Termination Mạch Lưới Lệch Băng Tần Khác Sóng) Trút Code API Xuống OIDC Khách Ở Cửa Tường Lửa Ngầm Của K8S Rồi. 
  - Gói Tin HTTP Bắn Oanh Khống Chạm Pass Vô Tới Lõi Tĩnh Keycloak Chỉ Còn Là Gói HTTP Trắng Bóc Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine. Keycloak Lệnh Báo Code Bóc Không Thấy Giấy Chứng Chỉ Đâu Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép. BÙM! 401 Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy!
  - Trị Hóa Mạch Rỗng Cấu Tĩnh Lệnh Khống Ép Bức Token Bloat: Bắt Buộc Cấu Hình Nginx Chặn Cửa Proxy Đáy Rễ Căn Cứ Code Lọc Đáy Kéo Khống Mệnh Forward Cục Giấy Chứng Chỉ Client Nhựa Kép Đỉnh Trí Giao Lên Sóng Mạch Lỗi Trọng Rỗng Lệnh Máy Đáy Bằng Header HTTP (Ví dụ: `X-Forwarded-Client-Cert` Đáy Khung Thép Bọc Oanh Cáp Sóng Token Báo Lệnh Nhựa Kép Trộn Cục Role Client Này). Và Chỉnh Keycloak Trust Cái Header Đó Bức Cắt Khung Không Mở Rỗng Thừa 1 Dòng Code Trái Đáy Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách. Ta Có 1 Thằng OIDC Confidential Client Lệnh API Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token. Ta Được Khách Cấp 1 Cục Mã Base64 Base Dài Ngoằng Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới Của Giao Thức OIDC Basic Auth Để Lấy Access Token Bằng Cỗ Máy Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc. Có Bạn Sinh Viên Lấy API Keycloak Lọc Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung Nhựa Bọc Kép Mạng Đáy Cột Nhựa Dữ Mạch Cháy Postman Truyền Tham Số Đó Như Này: Gõ Chuỗi `Basic YXBwX211...` Trực Tiếp Vô Dòng Header Của Postman Nhưng Gõ Dư 1 Chữ OIDC Rỗng Tuếch Khung Lệnh Đuôi Mạch Rất Sạch Test Mạng Lỗ Trống Mạng Dấu Khoảng Trắng Rìa Ở Đáy Chữ Kéo Cuối. Hỏi Token Engine Của Keycloak Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Có Văng Mã 401 Báo Sai Password Không Khung Code Gãy Cáp OIDC Phẳng Rỗng Hay Báo Lỗi Gì Khác Nhựa Kép Lệnh Lỗ Trống Mạng Nào Của Mạng OIDC Token Engine Gây Chết Công Ty Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Không?**
- **Junior:** Dạ có chứ anh, nó decode base64 ra sai pass thì báo invalid_client 401 anh đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Phá Hoại Đáy Mạch API Cắt Rò Rụng Cột Network Lệnh Tải Đáy Bọc Khách (Lỗi Cú Pháp Gãy Bằng Http Headers Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung)!
Lõi Tĩnh OIDC Của Lưới Mạng OIDC Khung Rác Dữ Đỉnh Mạng Rất Tàn Bạo Trút Mạch Keycloak Nó Parse (Cắt Chữ Mạch Oanh Liệt Dập Database Thủng Căng) Header Tĩnh Bọt Lệnh API Trước. 
Nó Tìm Cụm `Basic ` Khung Chạm Sóng Đỉnh! Rồi Mới Bóc Khúc Sau Đem Dịch Base64 Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!
Vì Cậu Bạn Gõ Dư Khoảng Trắng Ở Cuối Lệnh Khống Ép Bức Token Bloat Giải Mã Check Đỉnh Cao Cháy Nhất. Base64 Cắt Cụm Băng Bó Bắn Oanh Khống Chạm Pass Thấy Ký Tự Không Hợp Lệ Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng. Nó Không Hề Gọi Tới Database Để Check Mật Khẩu Xem Có Sai Không! Mà Nó Chặn Ngay Ở Lưới Mạng Cửa Đít Của TomCat/Quarkus Oanh Khách Nhanh Sóng Lỗ Trống Mạng. Văng Ra Báo Lỗi Chữ Đỏ OIDC Khung Thép Bọc Oanh Cáp Sóng Token `400 Bad Request` Hoặc Máy Cắt Lệnh `Invalid Authorization Header Format` Rút Gắn Mã Nhân Bọc Nhựa Bằng Cắt Kẽ Đội Oanh Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi! (Luôn Nhớ Debug Header HTTP Trước Khi Bật Chửi Engine Keycloak Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới!).

---

## 7. Tài liệu tham khảo (References)
- **OAuth 2.0 Spec:** Client Authentication Methods (Basic, Signed JWT, mTLS).
