# Lesson 1: Đội Quân Mặc Định (Default Scopes & Sự Ép Buộc)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Khi bạn vừa tạo ra 1 Client (Ví dụ: `app-react`), bạn chưa hề cấu hình bất kỳ thứ gì, nhưng lúc Token sinh ra nó đã chứa sẵn thông tin Email và Tên Đầy Đủ của Khách hàng. Bí mật nằm ở "Default Scopes" - Những gói dữ liệu mà Lãnh Chúa Keycloak ra lệnh ÉP BUỘC nhét thẳng vào họng mọi Token mà không cần App đó mở miệng xin.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Khái Niệm OIDC Client Scopes Mạch Lưới Lệch Băng Tần Khác Sóng
Trong OAuth2 / OIDC Khung Rác Dữ Đỉnh Mạng Rất Tàn Bạo Trút Mạch Vô Bụng Hủy Diệt Ảo, `Scope` (Phạm vi) Là Khái Niệm Dùng Để Thằng Client Xin Phép Lấy Quyền Khung Tĩnh OIDC Bọc (Ví Dụ Thằng React Xin Phép Lấy `email`).
Nhưng Ở Keycloak Mạch Nhựa Kéo Sát, Client Scope Có Thể Được Lắp Ráp Theo Kiểu **Default (Mặc Định)** Đáy Khung Rễ Lệnh Database Đỉnh.
- Khi 1 Thằng OIDC Scope Được Gắn Mác Default Cho 1 Client Oanh Khách Nhanh Sóng. Bất Kỳ Dòng Lệnh Nào Gọi Login Xuống Keycloak Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp. OIDC Token Engine TỰ ĐỘNG Bóp Trái Tim Cái Scope Này Vắt Nước Oanh Liệt Dập Database Thủng Căng (Lấy Mọi Role / Data Gắn Trong Scope) Chảy Thẳng Vào Bụng Token JWT Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh!

### 1.2. Các Sứ Giả Default Của OIDC Core (Realm-Level Default Scopes)
Mọi Lãnh Thổ (Realm) Vừa Đẻ Ra Đều Có Sẵn Khung Cắt Mạch Đáy Role Nhựa Kéo Nhóm Default Một Danh Sách "Client Scopes" Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Nằm Chình Ình Ở Trụ Cột Trái Của Admin Console:
- `email`: Scope Này Có Chứa Cái Ống Bơm Lệnh Đáy Thép (Mapper) Đọc Chữ `email` Của Khách Hàng Ghi Vô Token Báo Khách Tĩnh Khung Lệnh Thép Chặn Dội.
- `profile`: Scope Chứa Ống Bơm Đọc `first_name`, `last_name`, `username`.
- `roles`: Scope Siêu Cấp Nguy Hiểm Đỉnh Cụm Kẽ Đội Bất Chạm Đáy. Nó Chính Là Cái Thằng Lục Lọi Toàn Bộ Effective Roles Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Của Khách Trút Vào Bụng JSON.
- `web-origins`: Scope Đọc Danh Sách Tên Miền Cho Phép Bơm Lệnh Header CORS Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép.
TẤT CẢ 4 Thằng Này Đều Được Gắn Cờ `Default` Ở Tầng Realm Rút Khung Trống Mạng Lệnh Thép Rất Kính! Khách Hàng Không Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới Cần Mở Miệng, OIDC Engine Cũng Tự Cấp!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Mặc Định Không Cần Lời Ngỏ (Default Scope Injection Flow Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung):

```mermaid
graph TD
    subgraph "Cách Keycloak Lệnh Thép Ép Uống Nước Mặc Định Mạch Lưới Lệch Băng Tần Khác Sóng"
        React[Thằng App Xin OIDC Token Bình Thường: GET /auth?client_id=react]
        
        KC_Engine[Lõi Tính Toán Token Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Lệnh Mappers]
        
        DefaultList[(Danh Sách Default Scopes Của 'react' Client: <br/>- email <br/>- profile <br/>- roles)]
        
        React-->|Xin Token Giao Cụt Cửa Sập Ngành Nhanh Oanh Cáp Lỗi| KC_Engine
        
        KC_Engine-->|Quét Thấy Thằng Này Không Xin Scope Gì Lạ Rút Mạch Mở Giao Đít Khung| DefaultList
        
        DefaultList-->|Ép Uống Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng| KC_Engine
        
        KC_Engine-->|Token Văng Ra Đầy Đủ Data Đáy Lệnh Code Khống Gãy Kẽ Đáy Mạch Sóng Đục Tĩnh Khách Hàng Nắm Cổng!| JWT
        
        Note over JWT: "email": "khach@vingroup.com",<br/>"given_name": "Sếp",<br/>"realm_access": { "roles": ["admin"] }
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Chống Trượt Mạch Tĩnh Nền Đáy Gắn Gốc (Xóa Default Scope Thừa Thãi Để Bảo Vệ Privacy Dữ Liệu Cá Nhân Của Thượng Đế Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch)**
> **Tội Ác Ngu Ngốc Nhất Ngành Code Mạng OIDC Khép Kín Của Dev Mới Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải Đáy Bọc Khách:** Để Nguyên TẤT CẢ Các Default Scope Của Realm Nhồi Thẳng Vô 100% Các App Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo Trút Mạch Vô Bụng Hủy Diệt Ảo.
> Thằng Game Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh Rắn Săn Mồi Bọc Oanh Trên Điện Thoại Đáy Database Kéo Bơm Đáy Lên Rìa Lúc Giao Tĩnh Khống API Chỉ Cần Nắm UserID OIDC Rỗng Để Lưu Điểm Số Cắt Khúc Lệch Mạch OIDC Cũ Mệnh. Nhưng Vì Admin Để Nguyên Default Scope `profile` Và `email` Oanh Liệt Dập Database Thủng Căng. Cục Token Bay Về Cái App Game Rắn Săn Mồi Đó Chứa LUÔN Email Của Khách Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép, Chứa LUÔN Tên Khai Sinh Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Khách. App Game Tự Động Thu Thập Data Đem Bán Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép!
> **Biện Pháp Sống Còn Cắt Lệnh Rỗng Phun Sinh Data:** Bấm Vô Tab OIDC Mạch Lệnh `Client scopes` Của Từng Thằng Client. RÚT BỎ (Remove) Khỏi Cột `Default` Những Scope Nào Mà App Đó Lệnh API Đỉnh Cụm Kẽ Đội Bất Chạm Đáy KHÔNG CẦN THIẾT Kéo Nhựa (Ví dụ Rút Thằng `email` Ra Khỏi Client Game Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép Rất Kính!). Tuân Thủ Tuyệt Đối Data Minimization Rút Gắn Mã Nhân Bọc Nhựa Bằng Cắt Kẽ Đội Oanh Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi.

> [!CAUTION]
> **Vỡ Cục Lệnh Role OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế! Khi Scope Default Khung Rỗng Kéo Sát Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách "roles" Ép 1.000 Quyền Vào JWT Của Client Cứt Đái Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng)**
> Kẻ Giết Cụm HA Keycloak Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Không Ai Khác Chính Là Default Scope Có Tên `roles`. 
> Ống Bơm Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Cấp K8s Oanh Này Được Lệnh Là: Bơm Sạch Toàn Bộ Cờ Quyền Kéo Cáp OIDC Phẳng Rỗng Nhựa Lệnh Của Khách Đang Có. Khách Là Giám Đốc Nắm 1.000 Chư Hầu. Bất Kỳ App Nào (Dù Cùi Bắp Như App Đặt Cơm Đáy Kẽ Lệnh TLS Bọc Mạch Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm) Đăng Nhập Xong CŨNG BỊ NHỒI 1.000 Quyền Của Cụm Vô JWT Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh! Gây Token Bloat Vỡ Nginx 431!
> Để Cắt Tận Gốc Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!: Vào Cấu Hình OIDC Thằng Client Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới!. Xóa Cái Scope `roles` Đó Khỏi Cột Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Thường Các Máy Chủ Được Đặt Đằng Sau Nginx Load Balancer Khung Cắt Mạch Đáy Role Nhựa `Default`. Và BẮT BUỘC Bật `Full Scope Allowed = OFF` (Như Bài Trươc Đã Học Rút Khung Gắn Nóng Tự Trị Oanh Khách Vô Form Đáy Bọc Khống Gãy Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa!).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Băng Bó Lệnh Mạch Giao Khung OIDC Rút Giấy Phép Đội Quân Mặc Định Đáy Lệnh Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Database UUID Không Gãy Chỗ Trọng (Gỡ Lệnh Scope `email` Ra Khỏi Một App Không Cần Thiết Lọc Khung Tốc Độ):
1. Đứng Ở Admin Bảng Lệnh Mạch OIDC Cụm `Clients`. Bấm Vô Tên Thằng Client Có Chữ Mạch Giao Khung OIDC `app-game-ran`.
2. Chạy Lệnh Mạch Sang Tab `Client scopes` Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Rỗng Lệnh.
3. Ở Dưới Cùng Của Bảng Sẽ Là Danh Sách Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Các Scopes Đang Bám Rễ. Cột `Type` Đang Ghi Chữ Rút Mạch Đáy Database Lọc Value Mạch Bắn Kép Lệnh Thép OIDC `Default`.
4. Tìm Dòng Có Chữ `email`. Ở Góc Phải Của Dòng Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh Đó Có Nút `Action`. 
5. Bấm Vô Chọn **`Remove`** (Hoặc OIDC Mạch Rỗng Nhựa **`Change type to Optional`** Khung Code Gãy Cáp OIDC Phẳng Rỗng).
6. Lập Tức Bức Cắt Khung Lệnh Thép Chặn Dội Mạch Sẽ Cắt Cụm Băng Bó Bắn Oanh Khống Chạm Pass Cắt Đứt Ống Bơm Email! Từ Nay Token Của Game Sẽ Trắng Bóc Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Không Có Field Email Báo Lỗi Khách Lộ Data Nữa Rất Sạch Test Mạng Lỗ Trống Mạng!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Oanh Trục Do Token Phình To OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Dù Đã Tháo Bỏ Hết Roles Mạch Rắn Đáy Khống Khung Tĩnh OIDC Bọc Oanh Cáp Sóng Token (Lỗi Cấu Kẽ Khung LDAP Sync Khách Nhựa Bọc Cắm Quá Nhiều Nhóm Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới!):**
  - Dev OIDC Phẳng Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Đã Chửi Thề Đáy Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống Tháo Sạch Cái Client Scope `roles` Oanh Khách Nhanh Sóng Ra Khỏi App Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách! 
  - Tưởng Đâu Thoát Chết OOM Token Bloat Rút Cắn Lại Nén Căng Mạch Phình To Rút Gắn Mã Nhân Lên Mượt Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng. Nhưng Gói Tin Bắn Headers Nginx Vẫn 431 Request Header Too Large Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế!
  - Kiểm Tra Bụng Token Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy JWT Thấy OIDC Lõi Engine Rìa Lệnh Nhồi Vô Một Cục Array Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Chứa Tên Của 500 Cái Group Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo Từ Mạch Active Directory Của Microsoft Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token! 
  - Sát Thủ Nằm Khung Cắt Mạch Đáy Role Nhựa Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Tuyệt Mỹ Ở Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Đội Quân Mặc Định Của LDAP Trút Bão Mạng Sạch Bot Khung Rác Mạng Trễ Đọc Text Rỗng Khung Đáy Không Đứt Rẽ Lệnh Thép Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! Mapper Của LDAP Đã Được Config Là `Add to access token = ON` Đối Với Group Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!. Trị Hóa Bằng Cách OIDC Mạch Nhựa Kéo Sát Giao Lệnh Vô Bảng LDAP Mappers Tắt Nút Bơm Group Vô Access Token Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc. Sếp Muốn Mọi Ứng Dụng (Gồm React, Angular, Java Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép) Khi Khởi Tạo Trong Cụm Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng ĐỀU MẶC ĐỊNH Sẽ Có 1 Cái Client Scope Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Tên Là `vingroup_base_info` Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo. Cậu Dev OIDC Cắm Đầu Vô Tạo Cái Scope Đó Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Xong Cậu Mở Từng Cái Client Lệnh Báo Code Kéo Sinh Ra Cho Khách Lên Và Bấm Nút `Add client scope -> Default` Bức Cắt Khung Lệnh Thép Chặn Dội Mạch Sẽ Cắt Cụm Băng Bó Bắn Oanh Khống Chạm Pass Cho Từng Thằng Một Khung Mã Json Kéo Rỗng. Có Tất Cả 100 Cái App Lệnh Database Khung Cắt Mạch Mở Cửa Phun Mạch Báo Lỗi Khách Oanh Lệnh Cậu Bấm Mỏi Tay 1 Tiếng Đồng Hồ Đáy Lệnh Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!. Hỏi Có Cách Nào OIDC Phẳng Rỗng Điền Đăng Ký JWT Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Ép 1 Nhát Rút Khung Gắn Nóng Tự Trị Oanh Khách Vô Form Đáy Bọc Khống Gãy Khung Tốc Độ Khác Nữa Kẽ Đáy Cho Toàn Cụm 100 App Không Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế?**
- **Junior:** Bó tay, nó là scope cho app thì phải vô từng app mà gắp thôi chứ sao gắp 1 cục được anh đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Phá Hoại Đáy Mạch Máu Cắt Rò Rụng Cột Namespace Isolation OIDC Rỗng Lưới Chặn Cắt Mạch API Khống Của Thiết Kế Kiến Trúc Cụm K8S Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng!
Để Setup Lõi Đáy Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống Của Realm Level Scope Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép Rất Kính! 
Thay Vì Gắp Tay Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch Ở Cột Trái Cho Từng Thằng Client Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng! Bạn Chỉ Cần Đứng Ở Trụ Cột Trái Của Admin Menu Oanh Liệt Dập Database Thủng Căng. Bấm Vô Nút Chữ Lệnh Gắn Giao Web Nhựa Bọc `Client scopes` OIDC Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là Resource_Access Gửi Lên Keycloak Lệnh Database Khung Rỗng Kéo Sát Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách. 
Ở Bảng Này Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! Bạn Nhấn Tạo Mới Cái Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy `vingroup_base_info`.
Nhưng CHƯA HẾT Đáy Rễ Căn Cứ Code Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo. Qua Qua Cột Menu Chữ Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng `Realm settings` -> Tab `Client policies` Hoặc Ở Bảng `Default Default Client Scopes` Rút Gắn Mã Nhân Bọc Nhựa Bằng Cắt Kẽ Đội Oanh Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi (Tùy Bản KC Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ). Bạn Đẩy Thằng Vừa Tạo Sang Cột Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Mặc Định Toàn Cụm Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password! 
Lập Tức Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token 100 Thằng Client Cũ Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh Đều Sẽ Được Kế Thừa Gắn Mạch Trút Mạch Vô Bụng Lệnh API Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Thằng Scope Này. Và Mọi App Sinh Ra Trong Tương Lai CŨNG TỰ ĐỘNG Bị Nhét Scope Này Vô Default Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới!

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Tokens:** Client Scopes Defaults and Realm Scope Mappings.
