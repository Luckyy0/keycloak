# Lesson 1: Bộ Đồ Nghề Tích Hợp (Built-in Mappers)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong Keycloak, để lấy Dữ Liệu Khách Hàng (Tên, Email, Tuổi, Phòng Ban) nhét vào Token JWT dưới dạng JSON Key-Value (gọi là Claims), bạn KHÔNG PHẢI CODE. Keycloak đã cung cấp sẵn hàng chục Ống Bơm (Built-in Mappers) cực kỳ xịn sò. Chỉ cần Click chuột và trỏ đúng nguồn là Data sẽ tự chảy vào Token.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản Chất Của OIDC Protocol Mapper Mạch Lưới Lệch Băng Tần Khác Sóng
Khi Một Thằng Frontend Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Xin Keycloak Cấp OIDC Token Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token. Cỗ Máy Keycloak Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Phải Dịch (Map) Từ Ngôn Ngữ Khung Cắt Mạch Đáy Role Nhựa Kéo Nhóm Default Dữ Liệu Nội Bộ Của Nó (Java Object Oanh Khách Nhanh Sóng, Database Tables Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép) Sang Chuẩn Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Thường JSON Web Token.
- **Protocol:** Tức Là Chuẩn Giao Tiếp (VD: OIDC, SAML Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!).
- **Mapper:** Cái Ống Bơm Đáy Lệnh Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Database UUID Không Gãy Chỗ Trọng. Một Đầu Gắn Vào Bảng Postgres Lệnh Database Khung Cắt Mạch Mở Cửa Phun Mạch Báo Lỗi Khách Oanh Lệnh Của Keycloak (Nguồn), Đầu Kia Thọc Vào Bụng Cái Token Sắp Sinh Ra Rút Khung Gắn Nóng Tự Trị Oanh Khách Vô Form Đáy Bọc Khống Gãy Khung Tốc Độ Khác Nữa Kẽ Đáy (Đích).
Nếu Thiếu Mappers Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng, Payload Của Cục Token Sẽ Hoàn Toàn Trắng Bóc Mạch Rắn Đáy Khống Khung Tĩnh OIDC Bọc Bức Cắt Khung Lệnh Thép Chặn Dội Mạch Sẽ Cắt Cụm Băng Bó Bắn Oanh Khống Chạm Pass!

### 1.2. Kho Vũ Khí Built-in (Mappers Tích Hợp Sẵn Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng)
Keycloak Lệnh Database Khung Rỗng Kéo Sát Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Đã Đóng Gói Sẵn Rút Cắn Lại Nén Căng Mạch Phình To Hàng Chục Ống Bơm Đỉnh Cao Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch Oanh Khách Nhanh Sóng. Khi Mở Tab `Mappers` Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo Bất Báo Lỗi Nhựa Lệnh Của Client Hoặc Client Scopes Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng, Bấm Chọn `Configure a new mapper` Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng!, Một Bảng Danh Sách Sẽ Đổ Ra Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh:
1. **User Attribute:** Móc 1 Cột Tùy Chỉnh (VD `sdt`) Từ Bảng User Profile Nhồi Vô Token Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh!.
2. **User Property:** Móc Các Cột Cứng Bất Di Bất Dịch Của Keycloak OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Khi List Array Bắn Khung Cắt Mạch (VD: `email`, `username`, `firstName` Lọc Oanh Liệt Dập Database Thủng Căng).
3. **Group Membership:** Bơm Toàn Bộ Danh Sách Chữ Các Nhóm (Group) Mà Khách Hàng Đang Sinh Hoạt Vô Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Của Keycloak Khung Code Bọc Oanh Cáp.
4. **User Client / Realm Role:** Cỗ Máy Gom Cờ Quyền Nhét Vào Cấu Trúc Json Array `realm_access.roles` Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ.
5. **Audience:** Đóng Dấu Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Đích Đến Khách Hàng (Tên Của API Backend Server Rút Khung Trống Mạng Lệnh Thép Rất Kính) Lên Token Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy. Để Bảo Vệ Không Cho Đem Token React Của VNPost Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống Qua Gọi API ViettelPost Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Bằng Ống Bơm Mặc Định (Built-in Mapper Execution Flow Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung):

```mermaid
graph TD
    subgraph "Cách Token Engine Kéo Data Từ Các Built-in Mappers Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy"
        Login[Khách Đăng Nhập Xong Đáy Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo]
        
        TokenEngine[Cỗ Máy Lõi JWT Engine Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế!]
        
        ScopeList[(Lọc Ra Được Các Scopes Đã Cấp: <br/>email, profile)]
        
        MapperEmail((User Property Mapper: <br/>Đọc 'email'))
        MapperProfile((User Property Mapper: <br/>Đọc 'firstName', 'lastName'))
        
        DB[(Postgres DB Khung Mệnh Cắt Lệch Mạch OIDC Cũ Mệnh Ngắn Gọn)]
        
        Login-->TokenEngine
        TokenEngine-->|Quét Scopes Thấy Khách Được Cho Phép Oanh Khách Nhanh Sóng| ScopeList
        ScopeList-->|Kích Hoạt Ống Bơm Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng| MapperEmail
        ScopeList-->|Kích Hoạt Ống Bơm Đáy Lệnh Kéo Dọc Mũi| MapperProfile
        
        MapperEmail-->|Móc Cột Chữ Email Lọc Oanh Liệt Dập Database Thủng Căng| DB
        MapperProfile-->|Móc Cột Chữ Name Mạch Lưới Lệch Băng Tần Khác Sóng| DB
        
        DB-->|Trả Về String Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials| TokenEngine
        
        TokenEngine-->|Lắp Ráp Json Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine| JWT[JSON Token Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database UUID]
        
        Note over JWT: "email": "khach@gmail.com",<br/>"given_name": "Tèo",<br/>"family_name": "Nguyen"
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tối Ưu Tẩy Khách Mạng Bọc (Luôn Tái Sử Dụng Client Scopes Mạch Lưới Lệch Băng Tần Khác Sóng Thay Vì Config Mappers Chết Tại Từng Client Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép)**
> **Tội Ác Ngu Ngốc Nhất Ngành Code Mạng OIDC Khép Kín Cấu Cắt Chữ:** Một Thằng Dev OIDC Phẳng Rỗng Nhựa Lệnh Muốn Đưa Cột `chuc_vu` Vô Token Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token. Nó Vô Client React Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng, Nhảy Tab Mappers Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Cấm Cửa Mù Lòa, Bấm Tạo Mapper Ở Đó Lọc Oanh Liệt Dập Database Thủng Căng. Xong Nó Lại Qua Client Angular Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng!, Mở Tab Mappers Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo, Tạo Thêm 1 Cái Mapper Y Chang Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng. Nó Làm Vậy Cho 100 Cái Clients Oanh Khách Nhanh Sóng Lỗ Trống Mạng!
> **Biện Pháp Sống Còn Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng!:** Các Mappers NÀY Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Lệnh Thép Chặn Dội Khách Tuyệt Đối KHÔNG ĐƯỢC Khai Báo Tại Tab Của Từng Client Rút Khung Trống Mạng Lệnh Thép Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép. Bạn Phải Rút Mạch Đáy Database Lọc Value Mạch Bắn Kép Đứng Ở Bảng Menu Trái `Client scopes`, Tạo Một Hộp Scope Riêng Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! Tên Là `scope_chuc_vu`. Bấm Tạo Mapper Bức Cắt Khung Không Mở Rỗng Thừa 1 Dòng Code Trái Bơm `chuc_vu` Nằm Ở Trong Scope Đó Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới. Sau Đó Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch, Thằng Client Nào Cần Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu, Cứ Việc Chạy Vô Client Scopes Lệnh Database Khung Cắt Mạch Mở Cửa Phun Mạch Báo Lỗi Khách Oanh Lệnh Bảng UI Chặn JWT Mạch Nhựa Kéo Sát Của Nó, Nhấp Thêm `scope_chuc_vu` Dạng Default Hoặc Optional Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Rỗng Lệnh. Đó Mới Là Tái Sử Dụng Code Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Đúng Đỉnh Cao!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Băng Bó Lệnh Mạch Giao Khung OIDC Tìm Kiếm Built-in Mappers Đáy Lệnh Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!:
1. Đứng Ở Admin Bảng Lệnh Mạch OIDC Cụm `Client scopes`. Bấm Tạo 1 Scope Tên Là Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh `test-mappers`.
2. Bấm Vô Tên Thằng Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế! Mới Khởi Tạo Đó Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới!.
3. Chạy Lệnh Mạch Oanh Liệt Sang Tab Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng **`Mappers`** Rút Gắn Mã Nhân Bọc Nhựa Bằng Cắt Kẽ Đội Oanh Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi.
4. Bấm Nút Lệnh Báo Code Bóc Mạch Chữ Khung Rác Dữ Đỉnh Mạng **`Configure a new mapper`** (Bản 22+ Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ).
5. Bạn Sẽ Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database UUID Không Gãy Chỗ Trọng Nhìn Thấy Nguyên 1 Cái Siêu Thị Trút Kéo Ngầm Lập Tức Bức Cắt Khung Lệnh Ống Bơm Của Keycloak Được List Ở Đây OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Khi List Array Bắn Khung Cắt Mạch Đáy Group Attributes Nằm Phẳng Dưới Theme OIDC Bọc Lệnh API Rỗng Nhựa Do Flat Network Khung Trọng Rễ Lệnh Tái Trượt Sụp Cấu Trúc Nằm Đáy Vùng Ruột Cứng. Cứ Chọn Thằng Phù Hợp Rồi Click Vô Cấu Hình Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc. Có 1 Yêu Cầu Từ Sếp Frontend Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép: "Em Phải Code Cho Anh Làm Sao Trong Payload JSON Của Access Token Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo, Nó Phải Có Một Dòng Chữ Hardcode (Code Chết Lệnh Code Khống Gãy Kẽ Đáy Mạch Sóng Đục Tĩnh Khách Hàng Nắm Cổng) Là `"Cong_Ty": "Vingroup"` Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Cấm Cửa Mù Lòa Lệnh Báo Code Kéo Sinh Ra Cho Khách. Áp Dụng Cho Mọi App Của Cụm!". Cậu Dev Junior Kêu Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Phải Xuống DB Postgres Của Keycloak Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Sửa Tay 10.000 Dòng Dữ Liệu Của 10.000 User Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Để Thêm Cái Thuộc Tính Cong_Ty Vào Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng. Hỏi Có Cách Nào Nhẹ Nhàng 1 Click Không Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế!?**
- **Junior:** Dạ phải có data ở DB thì cái Mapper nó mới móc lên bơm vô được chứ anh, em update DB 1 lệnh SQL là xong đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Sai Lầm Của Tư Duy Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải Đáy Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Lấy Thịt Đè Cỗ Máy OIDC Mạch Nhựa Kéo Sát Giao Lệnh!
Trong Kho Vũ Khí Built-in Mappers Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép Rất Kính! Của Lãnh Chúa Đáy Database Kéo Bơm Đáy Lên Rìa Lúc Giao Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung. Có Một Ống Bơm Tên Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Là Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! **`Hardcoded claim`**. 
Chỉ Cần Bấm Tạo Mapper Này Ở Scope Default Của Cụm Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo Trút Mạch Vô Bụng Hủy Diệt Ảo. Cỗ Máy Này KHÔNG HỀ Gọi Xuống Đáy Kẽ Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống DB Postgres Của Keycloak Lọc Oanh Liệt Dập Database Thủng Căng. Nó Sẽ Dùng Sức Mạnh Của Mã Nguồn Ở Lõi OIDC Phẳng Rỗng Điền Đăng Ký JWT Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng In Trực Tiếp 1 Giá Trị Tĩnh (Constant Lệnh Database Khung Rỗng Kéo Sát Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách) Vào Bụng JWT Token Engine Rút Gắn Mã Nhân Bọc Nhựa Bằng Cắt Kẽ Đội Oanh Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Mỗi Khi Có Lệnh Bắn Sinh Mã Token Rút Khung Trống Mạng Lệnh Thép Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép. 
Ở Bảng Cấu Hình Của Thằng Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh `Hardcoded claim`, Cậu Chỉ Cần Nhập 2 Ô Khung Mã Json Kéo Rỗng: `Token Claim Name = Cong_Ty` Và Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế! `Claim value = Vingroup`. BÙM! 100 App Nhận Token Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ Chứa Vingroup Mà DB Vẫn Sạch Bong Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng, Không Phải Mất Công Chạy Update Cả Triệu Dòng Data Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Protocol Mappers:** Built-in Mapper Types.
