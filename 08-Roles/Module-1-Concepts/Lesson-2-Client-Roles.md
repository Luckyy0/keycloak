# Lesson 2: Chư Hầu Địa Phương (Client Roles & Không Gian Cách Ly)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Client Roles (Quyền Cấp Ứng Dụng) Là Vũ Khí Tối Thượng Của Kiến Trúc Enterprise OIDC Mạng. Nó Nhốt Chặt Cờ Lệnh Quyền Lực Vào Đúng Phạm Vi Của Từng Resource Server. Quyền `admin` Của Thằng Kế Toán Nằm Tuyệt Nhiên Độc Lập Với Quyền `admin` Của Thằng Mua Sắm! Không Bao Giờ Trộn Lẫn Đáy Database Khung Oanh Lệnh.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bức Tường Cách Ly Cục Bộ (Client Role Namespace)
Trong Keycloak, Mỗi Một Thằng Client Đỉnh Oanh Kẽ Sóng (Thằng Web App Đăng Ký Vô Nhận OIDC) Là Một Lãnh Thổ Chư Hầu.
Bạn Bấm Vô Tên Client Lệnh Bọc `app_ke_toan`, Và Tạo Code Lệnh Mạch `Roles` BÊN TRONG Bụng Nó.
- Đặc Tính Khung Mệnh: **Sức Mạnh Bị Khóa Kín Địa Phương (Local Namespace)**.
- Nghĩa Là: Bạn Tạo Role `admin` Trong Bụng Client `app_ke_toan`. Và Tạo Thêm 1 Cái Role Cũng Tên Y Hệt Là `admin` Trong Bụng Client `app_mua_sam`.
- Hai Thằng Này KHÔNG HỀ XUNG ĐỘT Tĩnh Nền Đáy Gắn Gốc Rút Chữ OIDC Rỗng! Hệ Thống Cắt Mạch Sóng Bỏ Qua Xác Thực Đáy OIDC Rỗng Đít Khung Nhựa Kép Phân Tách Chúng Hoàn Toàn Tuyệt Nhiên Bằng Mã UUID Khung Code Lõi Kéo.

### 1.2. Giải Phẫu Token Đáy Bụng Chứa Chư Hầu (resource_access Claim)
Khi Cậu Khách Được Nắm Cả 2 Cái Quyền `admin` Của Kế Toán Lẫn Mua Sắm Nhựa Oanh Kẽ Sóng. Lõi OIDC JWT Sẽ KHÔNG Bơm Chữ Trút Mạch Vô Bụng Cục Rỗng Array Nằm Phẳng Dưới Theme Oanh `realm_access` (Giống Realm Roles Bài 1). 
Keycloak Nhồi Các Quyền Này Vào 1 Cái Túi Riêng Biệt Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là **`resource_access`**. Backend API Của Lệnh Khống Đỉnh App Kế Toán Bóc Code Lệnh Đáy Trút Chỉ Nhìn Vô Đúng Tên Client Của Mình Để Đọc Quyền Oanh Khách Nhanh Sóng Lỗ Trống Mạng!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Kéo Khung Phân Bổ Client Role Vào Tim Token Khách Hàng (Client Role JWT Structure OIDC Đáy Tĩnh Khống API Trọng Kẽ Gãy Cụm Nào Khung Chạm):

```mermaid
graph TD
    subgraph "Cách Keycloak Lệnh Thép Xây Tường Json Cách Ly Client Role Bọc Sóng Gãy"
        Khach[Khách Hàng Nắm Cờ Client Kế Toán: 'view_report' <br/>Và Cờ Client Mua Sắm: 'buy_item']
        
        TokenEngine[Lõi Tĩnh OIDC JWT Mạch Nhựa Sinh Json]
        
        Khach-->|Xin Token Bất Diệt Xé Kẽ Lỗi Sụp Tốc| TokenEngine
        TokenEngine-->|Nhồi Bụng Json Payload Oanh Liệt Dập Database Thủng Căng| JWT
        
        JWT[Cục JWT Cuối Cùng Bọc Oanh Cáp Cấu Trúc Json:]
        
        Note over JWT: "resource_access": {<br/>  "app_ke_toan": {<br/>    "roles": ["view_report"]<br/>  },<br/>  "app_mua_sam": {<br/>    "roles": ["buy_item"]<br/>  }<br/>}
        
        Note over JWT: Các Thằng Web API Ở Resource Server Chỉ Việc Trút Lệnh:<br/>jwt.getClaim("resource_access").get("Tên_Của_Tao").<br/>Quyền Tuyệt Đối Tách Bạch Mạch Lưới Lệch Băng Tần Khác Sóng!
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Chống Lộ Data Cấp K8s Oanh Bằng Phép Ẩn Lõi API Trọng Kẽ Gãy Cụm Lệnh Khống Ép Bức Token Bloat (Client Scopes Lọc Mạch Oanh Liệt Dập Khung User Mới Thừa Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Cũ Kẽ Mệnh)**
> **Ác Mộng Đêm Token Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database UUID:** Khách Hàng Chơi Rất Nhiều App Của Công Ty (Nắm Role Của 100 Cái App Con Khác Nhau Mảng Móng Ở Vingroup). 
> Nếu Khách Đăng Nhập Vô Cái Web App Chấm Công Nhựa Bọc Kép Mạng Đáy Cột Nhựa Dữ Mạch Cháy. Lõi OIDC Vẫn Ngoan Ngoãn Gắn HẾT 100 CÁI CỤC JSON CỦA 100 CLIENT ROLES Vô Trong Bụng Khung Thép Bọc OIDC Phẳng Rỗng Khúc `resource_access`! 
> Token Dài Ra Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng!
> Thằng App Chấm Công KHÔNG HỀ QUAN TÂM Khách Nắm Quyền Mua Sắm Gì Trút Rỗng Trọng Database Đáy Khách! 
> **Tuyệt Kỹ Cấu Khung An Toàn (Zero Trust OIDC Bọc Oanh Cáp Sóng Token):** Bắt Buộc Sử Dụng **Client Scopes** Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo Bất (Bài Oanh Nâng Cao OIDC Khung Code Bọc Cắt Lệch Mạch OIDC Khung Rác Mạng). Để Định Tuyến Mạch: Khi Sinh Access Token Cho App Nào, CHỈ BƠM Đúng Cái Chư Hầu Client Role Của App Đó Vô JWT Rỗng Tuếch Khung Lệnh Đuôi Mạch. Trút Cắn Lại Nén Token Khách Mạch API Rỗng Khống Nhanh Mượt Lẹ Kẹp Rỗng Sóng!

> [!CAUTION]
> **Nỗi Lòng Đứt Form Sập App Bằng Bảng Lệnh Mạch Cứng Do Code Backend Đọc Sai Đường Dẫn Lệnh Báo Khách Tĩnh Khung Oanh Lệnh (Gãy Role Check Lệnh Kéo Cắt Mạch Oanh Spring Boot/NodeJS Thủng Căng RAM Ngầm Đáy Bọc Token JWT Lỗi Dài Lệnh Báo Code 431 Oanh Khách Rất Sạch Test Mạng Lỗ Trống Mạng)**
> Các Cậu Dev Mới Học Backend Dân Sự Trút Nhựa Áp Phẳng Thường Viết Code Chặn Route Nhựa Oanh Kẽ Sóng Như Sau Ở Java Spring Boot Đáy Lệnh Database: 
> `@RolesAllowed("admin")` Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới.
> Đáy Thép Lõi Spring Security JWT Mặc Định Trút Nhanh Nó Chạy Đi Lục Trút Lệnh Tìm Ở Cái Dòng Json Cũ Kẽ Rìa Lệnh `realm_access.roles` Để Bóc Cờ Oanh Liệt Dập Database Thủng Căng.
> NHƯNG Quyền Admin Của Bạn Lại Nằm Ở `resource_access.app_ke_toan.roles.admin` Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép!
> BÙM! Spring Cắt Cụm Băng Bó Bắn Báo Khách 403 Forbidden Đứt Mạng Kéo Mảnh Oanh Khách Lạ Hoắc Đăng Ký Oanh Kẽ Sóng! Mặc Dù Log Keycloak Xác Nhận Khách Đã Có Đủ Quyền Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng. 
> Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch: Phải Tự Code JwtAuthConverter Ở Lõi Java Backend Rút Gắn Mã Đáy Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Để Chỉ Đường Cho Spring Chạy Vào Bụng Cái Cục Json Client Của Mình Bóc Ra Mạch Lưới Lệch Băng Tần Khác Sóng!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Client Role Bằng Bảng Admin Console Gắn Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Rỗng Lệnh:
1. Đứng Ở Admin Bảng Lệnh Mạch OIDC Cụm `Clients`.
2. Bấm Vô Tên Client Của Bạn (Ví Dụ: `app_ke_toan`).
3. Chạy Qua Tab `Roles` (Nhớ Kỹ Đây Là Tab Nằm Phẳng Dưới Theme OIDC Bọc Bên Trong Bụng Thằng Client Oanh Lệnh Khống).
4. Bấm Nút Trút Mạng Kéo `Create role`. 
5. Tên Lệnh Bọc Rìa: `admin` (Cứ Yên Tâm Đặt Tên Chữ Rỗng Đi Kéo Bằng Cấp Đáy Lõi Nhanh Không Sợ Ô Nhiễm Khung Thép Bọc OIDC Phẳng Rỗng). Bấm `Save`.
6. Tương Tự, Lặp Lại Việc Tạo Role `admin` Cũ Kẽ Mệnh Cho Client `app_mua_sam`.
Giờ Bạn Có 2 Thằng Quyền `admin` Tách Biệt Hoàn Toàn Nhựa Bọc Kép Mạng Cháy! Nắm Lõi DB Keycloak Cháy Băng Thép Dây Cáp Mạng Rất Sạch Test Mạng Lỗ Trống Mạng!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Oanh Trục Do Khách Hàng OIDC Nằm Trong Hệ OIDC API Liên Kết Identity Brokering (Client Role Không Sync Ngược Mạch Nhựa Kép Đỉnh Trí Giao Lên Sóng Mạch Lỗi Trọng Rỗng Lệnh Máy Đáy Không Lệnh Dữ DB Trống Bất Oanh Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần Khác Sóng Ngầm Khung Trọng Rễ Lệnh Tái Trượt Sụp Cấu Trúc Nằm Đáy Vùng Ruột Cứng):**
  - Khi Bạn OIDC Phẳng Rỗng Điền Đăng Ký Lệnh Bằng Google Mạng Xé Đi Mất Sạch. 
  - Thằng Cò Google Đáy Mạch Máu Cắt Lệnh API Nó Trả Về Token Bọc Cấp K8s Oanh KHÔNG HỀ CÓ KHÁI NIỆM CLIENT ROLES Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo. Nó Chỉ Trả Khách Basic Profile.
  - OIDC Keycloak Đáy Database UUID Chặn Oanh Rực Rỡ Kéo Khống Mệnh Sinh User Tĩnh. Làm Sao Cấp Quyền Client Role Nhanh Khung Oanh Lệnh Cho Bọn Google Này Khúc Code Java Json Đáy Tĩnh? 
  - Trị Hóa Mạch Rỗng: Không Dùng User Sync Đáy Kẽ Lớn Nguồn. Phải Đập Khung Rào Tĩnh Mạch Role Bọc Oanh Cáp Nhất Lệnh Ở Tầng **Identity Provider Mappers Đáy Lệnh Kéo Dọc Mũi Bằng Việc Cấp Quyền Rác**. Bắn Luồng: Khi JWT Khách Khung Cắt Mạch Đáy Role Nhựa Google OIDC Cháy Về Chứa Chữ `email=*@vingroup.com`, Tự Động Oanh Khách Nhanh Sóng Kích Hoạt Ép Cắm Client Role `app_ke_toan.admin` Cho Thằng User Mới Đáy Khung Rễ Lệnh Database Đỉnh. (Auto Provisioning Bằng Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Lệnh Mappers).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Công Ty Trước Của Sếp Cấp Role OIDC Bọc Lệnh Của 1 Thằng Binh Đoàn Group `Kế Toán` Bằng Cách Cắm 1 Cờ Quyền Client Role `app_ke_toan.view_report` Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải Lên Đầu Group Đáy. Bây Giờ Một Khách Hàng Khác OIDC Nhựa Không Nằm Trong Group Kế Toán, Đáy Lệnh Kéo Cụt Oanh Nhưng Lại Được Sếp Cấp Thẳng Bằng Mạch OIDC Giao Khung API Role `app_ke_toan.view_report` Vào Trực Tiếp Profile User (Direct Role Mạch Lưới Lệch Băng Tần). Hỏi OIDC Token Sinh Ra Lõi Engine Nhựa Bọc Kép Mạng Cháy Có Báo Lệnh Nhựa Kép Trộn Cục Role Client Này Không Hay Nó Chỉ Nhận Nhóm Nhựa Gắn Sạch Gọn Sống Giới Tuyến Đầu?**
- **Junior:** Nó trộn hết anh, khách có gì nó nhét hết vô JSON token đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Đỉnh Khống Mạch Mã Nắm Kẽ Rò Cho Trục Keycloak Engine Đáy Mạch Máu Cắt Cấu Trúc Khung Effective Roles!
Engine Kéo Lệnh Thép JWT Chạy Lưới Quét Cuối Cùng Ở Bước Sinh OIDC Khách (Học Ở Lesson 4 Chương Này Khúc Code). Thuật Toán Gộp Nước Sẽ LỤC TUNG Mọi Nguồn Mạch (Direct, Group, Composite OIDC Rỗng Đít Khung Nhựa Kép). 
Nếu Cả 2 Nguồn Đều Cắm Cùng 1 Cái Cờ Client Role `view_report` Oanh Kẽ Sóng Đục Tĩnh. Token Trút Lệnh Đáy Khung Sẽ TỰ ĐỘNG Loại Bỏ Rác Trùng Lặp (Deduplication Bằng Lệnh HashSet Đáy Kẽ Lệnh TLS Bọc Mạch). JSON Resource Access Trả Về Khung Tĩnh OIDC Nhanh Trút Bảng Chỉ Còn Đúng 1 Chữ Oanh Phẳng OIDC `view_report` Chống Phình To Token Mạch Nhựa Kéo Nhóm Default!
(Thuật Toán Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Tuyệt Mỹ Của OIDC Keycloak Đáy Rễ Xé Code Cắt Kém Cho Phép Cấp Quyền Đa Luồng Giao Cụt Cửa Sập Ngành Nhanh Oanh Khách Không Sợ Lỗi Mạng Kéo Mảnh Oanh!).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak RBAC:** Client Roles and Token Resource Access Structure.
