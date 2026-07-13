# Lesson 7: Danh xưng Quyền lực (Roles)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Lật tẩy bản chất của Roles (Danh xưng Quyền lực). Phân biệt Rạch ròi Huyết Mạch giữa Realm Roles và Client Roles. Lĩnh hội Tuyệt chiêu "Role Lồng Role" (Composite Roles) để thu gọn JWT Token, cứu vãn mạng lưới Băng thông.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Role Là Sự Băng Bó Lười Biếng (Nhưng Cần Thiết)
Trong bài trước, ta biết ABAC (Gán quyền theo Thuộc Tính động) là Đỉnh cao. Còn Role (RBAC) là đồ cũ.
Tuy nhiên, Thế giới Vẫn Phải Sống Bằng Role vì Nó Rất Gọn. 
Role Bản Chất Là Một Cái **NHÃN DÁN (Label)** Ghi Bằng Chuỗi Text (Ví Dụ: `ADMIN`, `SUPERVISOR`, `USER`). 
Bản Thân Cái Role Đứng Im Lìm Trên Keycloak THÌ KHÔNG CÓ TÁC DỤNG GÌ SẤT. Nó Chẳng Chặn Được Trình Duyệt Nào, Chẳng Đuổi Được Kẻ Gian Nào. Tác Dụng Của Nó Là KHI VÀO ĐƯỢC ỨNG DỤNG KHÁCH (Backend). Các Bạn Backend Sẽ Soi Cái Nhãn Đó Bằng Lệnh `if (token.roles.contains("ADMIN"))`. Lúc Đó Sức Mạnh Sát Thương Mới Được Giải Phóng Bằng HTTP 403.

### 1.2. Mâu Thuẫn Vương Quyền: Realm Role vs Client Role
Sự Khó Hiểu Nhất Khi Học Keycloak Nằm Ở Điểm Phân Mạch Này:

- **Realm Roles (Quyền Vương Quốc):** 
  - Là Những Cái Danh Xưng CÓ SỨC ẢNH HƯỞNG TOÀN CỤC Bề Mặt Quốc Gia.
  - Ví Dụ: Role `MANAGER`. Khi Bạn Gắn Nó Cho Cô Alice. Bất Kỳ Cái App Nào Chạy Dưới Trướng Công Ty Đó (App Kế Toán, App Nhân Sự, App Giữ Xe) ĐỀU THẤY CHỮ MANAGER TRONG TOKEN CỦA CÔ ẤY. Dẫn Tới Có Khả Năng Cô Ấy Quản Lý Khống Cả Cái Nhà Giữ Xe. Quá Khủng Khiếp Tràn Lan.
  
- **Client Roles (Quyền Thuộc Địa App Riêng):**
  - Là Những Danh Xưng BỊ CẮT NHỎ VÀ NHỐT GỌN GÀNG TRONG PHẠM VI 1 CÁI APP DUY NHẤT.
  - Ví Dụ: App Kế Toán Sinh Ra Cái Client Role `CAN_VIEW_REPORT`. 
  - Nếu Cô Alice Có Cầm Nhãn Đó. Khi Đăng Nhập Đập Vào App Giữ Xe. Thằng App Giữ Xe HOÀN TOÀN KHÔNG THẤY CÁI ROLE ĐÓ. Vì Keycloak Thấy App Giữ Xe Xin Đăng Nhập, Nên Chỉ Ném Vào JWT Những Quyền Của App Giữ Xe. Giấu Biệt Mọi Quyền Khác. (Blast Radius / Cách Ly Rủi Ro Sát Thương Vô Cùng Tuyệt Đỉnh Trọn Trị Theo Tôn Chỉ Least Privilege).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Tuyệt Chiêu Nhồi Nhét Thuộc Địa Đỉnh Cao - **Composite Roles (Quyền Kép Khối Gộp)**:

```mermaid
graph TD
    subgraph "Nghệ Thuật Lồng Ghép Role Của Kiến Trúc Sư"
        Role_Doc_CTy[Realm Role: STAFF]
        
        App_A_Read[Client Role Kế Toán: READ_ONLY]
        App_A_Write[Client Role Kế Toán: WRITE_ALL]
        
        App_B_Read[Client Role HR: VIEW_PAYROLL]
        
        Composite_Ketoan_Truong((Realm Role Kép:<br/>KẾ TOÁN TRƯỞNG))
        
        Composite_Ketoan_Truong -.->|Gộp 3 Role Con Vào Ruột| Role_Doc_CTy
        Composite_Ketoan_Truong -.->|Gộp| App_A_Write
        Composite_Ketoan_Truong -.->|Gộp| App_B_Read
    end
    
    Note over Role_Doc_CTy,Composite_Ketoan_Truong: Bạn Không Cần Đi Gắn Từng Cái Cực Khổ Cho Cô Alice.<br/>Bạn Gắn ĐÚNG 1 CÁI MÁC "KẾ TOÁN TRƯỞNG" Cho Cô Ấy.<br/>Khi Login, Cỗ Máy Keycloak Auto Nổ Bung Bụng (Unwrap) Ra Các Quyền Con Ẩn Giấu Bên Trong Chạy Tức Thời Ra Gói JWT.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Đuổi Cổ Realm Roles Khỏi Ứng Dụng (Stop using Realm Roles for App AuthZ)**
> Sai Lầm Lớn Của Tân Binh: Code 5 Cái Spring Boot. Xong Xài Chung 1 Cái Tên Là `Realm_Admin`. 
> Hậu Quả: Khi Nâng Cấp Công Ty Đập Bỏ 1 App Spring Boot Đi, Nhưng Vẫn Còn Sót Tên `Realm_Admin` Tràn Lan Trên Keycloak Gây Tắc Nghẽn Cấu Trúc Khủng Khiếp, Ai Cũng Nhận Vơ Quyền Của Nhau Được. Mớ Rối Cực Hại Về Trừu Tượng Lệnh API Không Hồi Kết Tách Tầng Microservices.
> **Kiến Trúc Tĩnh:** BẮT BUỘC Phân Quyền App Nào (Microservices) Thì Lên Keycloak Khai Báo Sinh Ra Client Riêng Cho App Đó, Sau Đó Tạo Client Roles Gắn Chặt Lõi Từng Trục, Rời Đi Chết Kéo Chết Chum Sạch Sẽ Bất Khả Xâm Lẫn Của Nhau.

> [!CAUTION]
> **Thảm Họa Đổi Tên Role Phá Hủy Api (Rename Role Breakage)**
> Giao Diện Web Keycloak Hiện Dòng `Edit Role Name` Trông Rất Hiền Lành Ngon Ăn. Bạn Bấm Đổi Role Từ `VIEWER` Thành `READER`. Cười Cười Tưởng Xong Nhiệm Vụ Sếp Giao Đổi Văn Phong Tiếng Việt.
> Tí Về: **Đứt Mạng Sập Cổng Toàn Cục! Khách Hàng Gọi Gào Khóc Báo Lỗi 403 Mọi Nơi Không Truy Cập Được Gì!**
> Tại Sao? Vì Backend Code Viết Cứng Cáp Hàm `if (role.equals("VIEWER"))` Biên Dịch Xong Từ Đời Nào Rồi Ốp Lên Sever. Cái Tên `VIEWER` Đã Trở Thành Mệnh Đề Hợp Đồng API Contract Giao Kèo. Việc Bấm Đổi Tên Trên Server Cắt Đứt Sự Giao Thoa Kết Đôi Kênh Băng Thông Tại Chỗ Bắn Lủng Sọ Toàn Khối 403 Sập Phổ Rộng. Tránh Xa Đổi Tên Trừ Khi Fix Cả 2 Phía Code Dưới!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Làm Sao Ép Keycloak Dịch Tên Role Mới Bằng Mappers:
Làm Lại Ví Dụ Trên 1 Cách Lành Lặn Hơn Đẹp Mạng Hơn Nếu Bắt Buộc Đổi Văn Phong (Không Sửa Tên Code Cũ).
1. App Kế Toán Code Cứng Đòi Chữ `"role_cua_toi" = "VIEWER"`.
2. Sếp Bạn Chạy Lên Keycloak Bắt Gõ Chữ Đổi Giao Diện Thành Quyền `"NGUOI_XEM_GIAO_DICH"`.
3. Bạn Đổi Tên Thật Ở Keycloak Sang Chữ Dài Của Sếp (Cho Vừa Mắt Lòng Người Lập Trình Hệ Phả Hệ).
4. SAU ĐÓ BẠN CỨU RỖI Bằng Cách: Vào Client Của App. Thêm Cấu Hình Protocol Mappers. Chọn Mẫu Chuyển Ngữ `Role Name Mapper`. 
5. Cột Trái Bạn Trỏ Role Mới Của Sếp. Cột Phải Bạn Ép Trả Trực Tiếp Trả Ra Dãy Chữ Cứng `"VIEWER"`.
KẾT QUẢ ĐỈNH CAO: Bề Mặt Quản Trị Hệ Hệ Dùng Chữ Đẹp Đẽ Mới, Nhưng Khi Ép Xuất JSON Chạy Về Cho Code Dưới Bụng Cũ Thì Mapper Tự Nắn Lại Đích Về Chữ Cũ. Khớp Lệnh API Trở Lại Mà Cả Làng Đều Mừng. Tuyệt Đỉnh Khớp Xoay Keycloak Mapping Đúc Hình Linh Hoạt!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Default Roles (Quyền Hạn Auto Gắn Mặc Định Từ Bụng Mẹ Ra Đời Khóc Thé):**
  - Trong Hệ Sinh Thái Có 1 Số Ứng Dụng Chẳng Cần Bảo Mật Gì Quá Khắt Khe, Ví Dụ App Chat Giao Tiếp Chung, Vào Công Ty Là Xài Được Khỏi Xin.
  - Bạn Khỏi Mất Công Bấm Cấu Hình Xin Cho Nhức Đầu. Mở Cài Đặt `Realm Roles` Lên Của Keycloak, Chọn Chọn Tên Trái Tab Tab Cột Ghi `Default Roles`. Nhét Sạch Các Quyền Lặt Vặt (VD: `read_notice`, `chat_allow`) Vào Đóng Hộp Đó.
  - Khi Cửa Số Hàng Triệu Thằng Lính User Mới Đăng Ký Acc Bơm Bằng Lệnh Script Liên Hoàn Tạo Mới... CHÚNG SẼ AUTO SỞ HỮU Gắn Bám Mấy Cặp Quyền Đó. Không Còn Lo Bị Bỏ Chết Đứng Ở Ngoài Hiên Báo Lỗi 403 Do IT Quên Kéo Group Gắn Lâu Lắc Quên Tích Nữa.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Giải Thích 1 Chiều Oái Õm Trong Quyền Kép (Composite Role): Nếu Quyền Tổng Cục A Bọc (Contains) Quyền Con Hạt B. Cấp B Cho Mày Thì Mày Có Cầm Được A Không? Hay Phải Cấp Cả Khối A Mới Vỡ B Bằng Thuật Toán Giải Lỗ Mọc Cánh Màng Composite Unwrap Của Keycloak?**
- **Junior:** Chắc cấp nào dính nấy, A hay B lôi chùm hết.
- **Senior:** Chú Ý Chiều Chạy Suối Băng Một Hướng (One-way Delegation).
Composite LÀ KẾ THỪA TỪ TRÊN NHÚNG XUỐNG DƯỚI RỄ.
Nếu A (Giám Đốc) gộp B (Quét Rác) và C (Pha Trà).
- CẤP A: Thằng Cầm A Nhận Đủ Luôn Khí Thế Của Bộ 3: A, B, C Nhờ Lồng Quả Bóc Ra.
- CẤP B: Nếu Đưa Khơi Khơi Mệnh Lệnh B Cho Thằng Quét Rác Thuần. Mặc Dù B Nằm Trong A, Nhưng THẰNG B CHỈ BIẾT NÓ LÀ B (Quét Rác). Nó TUYỆT ĐỐI KHÔNG BÒ NGƯỢC LÊN ĂN CẮP CHIẾC GHẾ A CỦA GIÁM ĐỐC TRÊN CÂY LÕI PHẢ HỆ COMPOSITE. Đó Chính Là Gắn Kẽ Kéo Phân Khúc 1 Chiều.

**2. Token JWT Bị Phình To 16 KB Vì Sếp Yêu Cầu Cấu Trúc Khối Composite Gắn Cây Hàng Trăm Role Khắp Cả Vương Quốc. Máy Chủ API Báo Chết Không Xử Lý Nổi Headers Hắt Hủi 431. Khắc Phục Kiến Trúc Cho App Thế Nào Mà Vẫn Dùng Lõi OIDC?**
- **Junior:** Tăng cái giới hạn Nginx cho to lên là nhận được hết.
- **Senior:** Đó Là Bịt Lỗ Tạm Thời, Không Cứu Nổi Băng Thông 1 Triệu Người Đăng Nhập Cắn Nát Tốc Độ Mạng Load Khung Header Đè Bẹp Chết Sạch Hệ Kênh.
Sửa Đổi Tại Đáy Gốc Khung Mã Máy (Token Claim Limiting):
- Đứng Từ Bảng Console Của Client, Chọn Phần `Client Scopes`. Thấy Mục `Roles`. Mở Cấu Hình Răng Cưa Lên Tắt Mục Tùy Chọn **`Add to access token`** Và Tắt Cả Cờ Khai Báo **`Add to ID token`**. Thay Vào Đó CHỈ BẬT DUY NHẤT Lưỡi Cưa Báo Cáo **`Add to userinfo`**.
- Hậu Quả Áp Dụng: JWT Token Trả Đi Siêu Mỏng Nhẹ Nhàng Bốc Hơi Hoàn Toàn 1 Trăm Mảng Rác Roles Nặng 16 KB Bụng Ở Đáy Đẩy Lên Cổng (Bay Được Chướng Ngại Nginx Báo Gắt Khung Chặn 431).
- Ứng Dụng App Bị Mù Roles Tại Đỉnh Đón Giao Tiếp. Đổi Lại, Nó Dùng Tầm Thẻ Lệnh Cầm Đầu Gọi 1 Chuyên Dịch POST Backchannel Đi Riêng Đi Kín Sang Endpoint Gọi Là Lãnh Cung Phía Sau Keycloak Lưng **(`/userinfo`)** Bằng Dữ Liệu Gọi Nghẽn JSON Dễ Ăn Phân Mảnh Rút Khối Data Role An Toàn Ra Để Kiểm Chặn Tại Khối Dữ Liệu Backend (Tách Trọng Tải Băng Thông Ngược Kẹp Đánh Chặn Header Khung Trần REST Mảnh Dẻ Phơi Nắng Bị Dò).

**3. Làm Sao Bơm Quyền Cấu Hình API Backend Mở Cho Microservices Mới Không Qua Giao Diện Admin Console Trực Tiếp Mà Dùng Code?**
- **Junior:** Thì xài bot xài chuột click tự động RPA cắm thôi chứ.
- **Senior:** Keycloak Được Cấy Trọn Vẹn Khung **Admin REST API (Siêu API Thống Trị)**.
Mọi Hành Động Bấm Chuột Từ Tạo Client Trống Trơn, Nhồi Group Ánh Sáng Tới Khớp Role Cầm Vị Khóa JWT Trả Nhanh.. TẤT CẢ Tựu Chung Lại Qua Giao Giao Tiếp GET/POST/PUT Của REST Ngầm Dưới Đường Đua Network Admin. 
Chỉ Cần Cấp Access Token Loại Quyền Mạng Master Realm (Client = admin-cli). Mọi Mã Lệnh Python Bấm Nút Gọi `POST /realms/{realm}/clients/{id}/roles` Lập Tức Dội Quá Trình Ốp Đóng Data Mảnh Lõi Không Cần Màn Hình Web Thao Thao Rác Việc DevOps Ơi Hời Phí Tay Nghề IaC Terraform Nhạt Màu. Hệ Terraform Provider Của Keycloak 99% Hoạt Động Nguyên Rễ Từ Bộ Sát Thủ Endpoint Dấu Này Sinh Tử Hủy Diệt Nhanh Chóng Khung Hạ Tầng Sống (IaC Provisioning Vô Song).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Server Administration Guide:** Realm Roles vs Client Roles.
