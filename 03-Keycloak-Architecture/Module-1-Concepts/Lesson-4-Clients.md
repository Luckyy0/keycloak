# Lesson 4: Ứng dụng Khách (Clients)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Lột xác khái niệm "Client". Trong ngành Lập Trình Web, Client là Trình duyệt (Chrome). Trong Ngôn ngữ Kiến trúc IAM (OAuth2/OIDC), Client là CÁI PHẦN MỀM đứng ra XIN QUYỀN TRUY CẬP (Service Provider / Relying Party). 

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Client Trực Thuộc Realm
Vương Quốc (Realm) là Mảnh Đất. Vậy **Client (Ứng Dụng Khách)** chính là Các Tòa Nhà (Bệnh Viện, Ngân Hàng, App Kế Toán) Xây Dựng Trên Mảnh Đất Đó.
- Mọi Client Bắt Buộc Phải Thuộc Về Đúng 1 Realm. Không có Client Bay Lơ Lửng.
- Khi Khách (User) Đi Vào Tòa Nhà (Client). Tòa Nhà Kêu Khách Đi Lấy Giấy Phép. Khách Chạy Lên Ủy Ban Quận (Realm) xin Tờ Giấy (JWT). Ủy Ban Đóng Dấu Chữ Ký Của Quận, Khách Đem Về Đưa Cho Tòa Nhà. Tòa Nhà Đọc Dấu Chữ Ký. Cho Khách Vào. 

### 1.2. Phân Loại Client Theo Độ Kín Đáo (Confidentiality)
Hiểu Xóa Mù Khái Niệm Phân Loại của OAuth2, Vì 90% Lỗi Rò Rỉ Bí Mật (Secret Leakage) Xuất Phát Từ Đây:
1. **Confidential Client (Ứng dụng Kín/Bảo Mật Máy Chủ):**
   - Loại Phần Mềm: Ứng dụng Chạy Dưới Đáy Backend (Spring Boot, NodeJS, PHP Laravel).
   - Đặc điểm: Có Khả Năng GIẤU ĐƯỢC Bí Mật Khỏi Con Mắt Của Khách Hàng. Do Mã Nguồn Chạy Kín Cổng Cao Tường Trên Máy Chủ Của Công Ty.
   - Cấp Phép: Nó Được Cấp Một Cặp Bài Trùng `Client ID` (Tên) và `Client Secret` (Mật Khẩu Của Ứng Dụng - Giống Mật khẩu Con Người).
2. **Public Client (Ứng dụng Trần Truồng):**
   - Loại Phần Mềm: Ứng Dụng Frontend Chạy Trực Tiếp Trên Trình Duyệt Khách Hàng (ReactJS, Vue, Angular) hoặc App Mobile Tải Trên Store (iOS, Android).
   - Đặc điểm: Mã Nguồn Phơi Bày 100%. Bất Kỳ Ai Bấm F12 Hoặc Decompile App Mobile Đều Thấy Hết Code Ở Trong.
   - Cấp Phép: TUYỆT ĐỐI KHÔNG BAO GIỜ GIAO `Client Secret` Cho Bọn Này. (Vì Giao Là Bị Hack Nửa Nốt Nhạc). Tụi Nó Chỉ Xài `Client ID` Trần Chuồng Kết Hợp Với Mã Khóa Động PKCE.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Khi Client Nói Chuyện Với Keycloak (M2M / Đổi Code Lấy Token): Làm sao Keycloak biết thằng Client gọi API Là Hàng Thật?

```mermaid
graph TD
    subgraph "Cách Máy Chủ App Gọi Keycloak Lấy JWT (Luồng Token Endpoint)"
        App(Spring Boot App <br/> Confidential)
        KC(Keycloak Server)
        
        App->>KC: "Ê Keycloak, Đổi Cái Mã Code Này Lấy JWT Cho Tao Lẹ"
        Note over App,KC: Để Chứng Minh Là App Spring Boot Xịn (Không phải Hacker Cướp Code).<br/>App Bắt Buộc Phải Đưa Ra Bằng Chứng.
        
        KC->>KC: Keycloak Dò Xem Loại Client Gì?
        KC-->>App: "Chứng Minh Mày Là App Spring Boot Đi!"
        
        App->>KC: "Đây Là Cái Client Secret Tao Cất Kỹ Dưới Đáy Mồ (Ví dụ: x89As2...)"
        KC->>KC: Check Khớp Trong DB! Cấp JWT Trả Về.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Biến Tướng Thành Robot Gọi API (Service Accounts)**
> Khi Client KHÔNG CÓ CON NGƯỜI ĐĂNG NHẬP, Nó Chạy Tự Động Hóa Dữ Liệu Lên AWS 3h Sáng. Lúc Đó Bản Thân Thằng Client LÀ MỘT CON NGƯỜI (Bot/Machine).
> **Tính năng Tuyệt Đỉnh:** Trong Keycloak, Nếu Client Thuộc Loại `Confidential`. Bạn Được Phép Bật Một Cái Công Tắc Tên Là **`Service Accounts Enabled`**.
> Lúc Bật Lên, BÙM! Keycloak Âm Thầm Sinh Ra 1 Cục Identity Vô Hình (Một Cái Thằng User Tàng Hình) Đứng Đằng Sau Cái Client Đó. Thằng App Có Thể Dùng Đúng Cái `Client Secret` Của Nó ĐỂ LOGIN TRỰC TIẾP LẤY TOKEN (Luồng Client Credentials Grant). Kiến trúc Phân Quyền API Server-to-Server Mặc định Xài Đòn Này.

> [!CAUTION]
> **Thảm Họa URL Gọi Ngược Kẻ Bắt Cóc (Valid Redirect URIs)**
> Đã Nói Ở Lesson 6 Nhưng Phải Đóng Cột Khắc Bia Chỗ Này.
> **Lỗ Hổng:** Nếu Bạn Lười Biếng, Khai Báo Ô Cấu Hình `Valid Redirect URIs` Thành Ký Tự Hoa Thị Dấu Sao `*`.
> **Hậu Quả:** Một Quả Pháo Sáng Dâng Hiến Cổng Thành Cho Bọn Cướp Open Redirect. Hacker Chỉ Cần Gọi Link Login Keycloak Của Bạn, Gắn Đuôi `?redirect_uri=https://web-den-toi.com`. Keycloak Thấy Cấu Hình Của Bạn Là `*` (Cho Phép Mọi Nơi). Nó Đẩy Nguyên Cục Access Token Về Trang Web Của Hacker Chứ Không Đẩy Về Web Của Công Ty. Đứt Bóng Toàn Tập.
> **Quy Luật Thép:** Bắt Buộc Gõ Cứng Chuẩn Xác (Exact Match) Đích Đến Của Callback. (VD: `https://erp.congty.com/oauth2/callback`).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Tái Định Tuyến Dữ Liệu Bằng Cỗ Máy **Protocol Mappers (Trình Ánh Xạ)**:
Bạn Đang Gặp Lỗi Kinh Điển Này: "Web Kế Toán Cần Thông Tin Lương Của Nhân Viên Trong Token, Nhưng Token Trả Về Mặc Định Của Keycloak Chỉ Có Tên Và Email Thôi".

Keycloak Giải Quyết Bài Toán Này Cực Kỳ Thượng Thừa Bằng Mapper Bên Trong Tab Client:
1. Bạn Vào Cấu Hình Client `App-Ke-Toan`. Chuyển Sang Táp `Client Scopes` -> `Dedicated`.
2. Bạn Thêm Một Cái `Mapper` Mới. Chọn Loại `User Attribute`.
3. Bạn Khai Báo: Lôi Cái Dữ Liệu `luong_thang` Ở Dưới Đáy Database Của Thằng User -> Gắn Vào Cột JSON Có Tên Bề Mặt Là `salary` Trong Ruột JWT.
4. Bấm Save. Lập Tức Ngay Lần Login Tiếp Theo, JWT Bắn Về App Kế Toán Mọc Thêm Một Cột `"salary": "20.000$"`.
Sự Vi Diệu Là: Bạn Không Cần Sửa Code Lõi Của Keycloak Nào Cả. Mapping (Nhào Nặn Dữ Liệu) Trực Tiếp Bằng Giao Diện Trên Từng Client. App Nào Cần Data Gì Thì Bơm Data Đó (Không Bơm Dư Cho Kẻ Khác).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Chiến Đấu Không Cần Bí Mật Ở Front-End (PKCE - Proof Key for Code Exchange):**
  - Môi Trường: App Mobile Ghi Danh (Public Client). Hacker Tải App Của Bạn Về Máy Hắn. Bấm Decompile Lôi Cháy Cả Ruột App, Đọc Được Hết API. Làm Sao Bạn Chống Cướp Token Khi Không Thể Giữ Được Cái `Client Secret` Nào Cả?
  - **Sự Giải Cứu Toán Học PKCE:** Mỗi Một Lần (Mỗi Khi Thằng User Bấm Login). Cục App Trê Điện Thoại Sẽ Tự Sinh Ra 1 Mã Bí Mật Đổi Liên Tục (Code Verifier). Nó Băm Cái Mã Đó Thành Hàm (Code Challenge) Gửi Lên Keycloak. 
  - Tí Nữa Quay Lại Đổi Lấy JWT, Nó Mới Ói Cái Mã Gốc Ra. Keycloak Thấy Khớp Hàm Băm Mới Cho Token. Hacker Lấy Được Mã Code Chặn Đứng Giữa Mạng, Nhưng DO KHÔNG HỀ BIẾT CÁI MÃ BÍ MẬT BAN ĐẦU CỦA ĐIỆN THOẠI (Vì Nằm Trong RAM Điện Thoại Đang Cầm Ở Tay Người Dùng). Hacker Đứng Khóc. Lỗ Hổng Bị Bịt Kín 100%.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Giao Thức OIDC (Keycloak), Nếu Client A Xin Quyền "Đọc Sổ Đỏ". Chuyện Gì Xảy Ra Với "Consent Screen" (Màn Hình Phê Duyệt)? Trách Nhiệm Hiển Thị Của Ai?**
- **Junior:** App A tự hiện cái form hỏi Khách.
- **Senior:** Trách Nhiệm Thuộc Về Thẩm Phán OIDC (Keycloak). 
Khi Ứng dụng Third-Party (Bên Ngoài Cty) Khai Báo Là Client Của Keycloak. Ta Phải Bật Cờ Cấu Hình **`Consent Required = TRUE`**.
Khi Client A Chạy Lệnh Gọi Login Sang Keycloak (Kèm Biến `scope=doc_so_do`).
Keycloak Bắt Khách Gõ Pass. XONG KHÔNG ĐƯA VÀO LUÔN. Nó Cắt Lập Tức Luồng Chạy Bằng Màn Hình: *"Tòa Nhà A Đang Cố Gắng Đọc Sổ Đỏ Của Bạn. Bạn Có Đồng Ý Cho Tòa Nhà Đó Đọc Không?"*. 
Chỉ Khi Bấm Nút Đồng Ý. Keycloak Mới Ghi Giao Dịch Đồng Ý Vào DB (Lưu Sự Bằng Chứng). Và Xuất Access Token Ra. Client A Tuyệt Đối Không Thể Tự Cấp Quyền Đọc Dữ Liệu Của Khách Nếu Khách Không Chủ Động Cho Phép Tại Màn Hình Của IdP.

**2. Nếu 1 Công Ty Lớn Có Tới 500 Cái Microservices (Mỗi Con Là 1 Cái REST API Độc Lập). Nếu Phải Cấu Hình Bằng Tay Từng Con Làm 1 Client Vào Keycloak Chắc Sẽ Rụng Hết Tóc IT. Có Cách Nào Tự Động Hóa Không?**
- **Junior:** Chịu Đựng Mà Bấm Chuột Tạo 500 Cái Thôi.
- **Senior:** Keycloak Hỗ Trợ Đỉnh Cao Công Nghệ API Chuẩn Khối: **Dynamic Client Registration (DCR)**.
Theo Tiêu Chuẩn RFC 7591. Thay Vì Quản Trị Viên Tạo Tay. Khi Một Con Microservices Bật Boot (Khởi Động Nguồn K8s). Nó Tự Động Bắn 1 Gói Tin JSON HTTP Lên Cổng Đăng Ký Động (`/clients-registrations`) Của Keycloak.
Trong Gói Tin Ghi Rõ Tên Đóng Đinh, Mũi Tên, Hướng Chạy Của Nó. Keycloak Đọc Và TRONG TÍCH TẮC TỰ SINH RA CLIENT BÊN TRONG CSDL, RỒI TRẢ VỀ CÁI ID/SECRET VÀO BỤNG CON MICROSERVICE ĐÓ. Không Cần Một Nửa Giọt Mồ Hôi Của IT Admin. Tự Động Hóa Toàn Diện.

**3. Làm Sao Bắt Client Chứng Minh Thân Phận (Client Authentication) Bằng Cách Cực Kỳ Tối Mật Thay Vì Dùng Cái Dây Chữ String `Client Secret` (Dễ Bị Lộ Trộm)?**
- **Junior:** Hết Cách Rồi, Nó Bắt Buộc Có Secret Mới Dùng Được.
- **Senior:** Bạn Chưa Đụng Tới Cảnh Giới Đáy Ngành Ngân Hàng (Mức Bảo Mật Dốc FAPI).
Ngân Hàng Rất Ghét Chuỗi String Secret (Vì Lỡ Bỏ Vào File Code Gặp Kẻ Cắp Xem Được Là Tiêu). Keycloak Hỗ Trợ 2 Chuẩn Xác Thực Client Cực Lệnh Kì Hơn:
1. **Signed JWT:** Client Đổi Chìa Khóa (Private Key RSA). Nó Tự Ký Cmn Vào Một Cái Tờ Giấy Trắng Bằng Key Của Nó. Keycloak Lấy Public Key Của Client (Đã Trao Đổi Từ Trước) Để Check. Nếu Đúng, Nó Vào.
2. **X509 Client Certificate (mTLS):** Xác Thực Bằng Khung Truyền Mạng. Bắt Buộc Server Client Đưa Chứng Chỉ Điện Tử TLS Khóa Kép Ngay Lớp Tầng Bốn (Layer 4 TCP) Mới Được Lọt Vào. Cái Này Đánh Sập 100% Cửa Tấn Công Từ Xa Bằng Bí Mật Rò Rỉ. Đẳng Cấp Thần Tiên.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Client Registration & Client Authentication.
- **OAuth 2.0 RFC 7636:** Proof Key for Code Exchange by OAuth Public Clients (PKCE).
