# Lesson 3: Vương quốc (Realms)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Nắm vững khái niệm Cách ly Dữ liệu Tối cao (Multi-tenancy) trong Keycloak. Tại sao lại gọi là Realm? Quyền lực tuyệt đối của Master Realm và Tại sao bạn KHÔNG BAO GIỜ được phép nhét Khách hàng vào đó.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Realm là gì?
**Realm (Vương Quốc)** là Đơn vị Khối Xây Dựng Lớn Nhất và Cao Nhất trong Cấu trúc Dữ liệu của Keycloak. 
Nó cung cấp tính năng **Multi-tenancy (Đa khách thuê)** hoàn hảo.
- Giả sử Công ty bạn Cung cấp Phần mềm Kế toán cho 2 Khách hàng lớn: Tập đoàn Vingroup và Tập đoàn FPT.
- Bạn KHÔNG CẦN phải cài 2 cái Máy Chủ Keycloak tốn tiền RAM.
- Bạn chỉ cần chạy 1 Máy chủ Keycloak. Bên trong tạo 2 Realm: `Realm-Vingroup` và `Realm-FPT`.
- Mọi thứ bên trong Realm (Từ Thằng User, Cái App, Cục Role, Đến Giao diện Đăng nhập) BỊ CÁCH LY TUYỆT ĐỐI. Anh Nguyễn Văn A ở `Realm-Vingroup` HOÀN TOÀN KHÔNG TỒN TẠI trong mắt `Realm-FPT`. Hai vương quốc không có cầu nối, không chung luật lệ.

### 1.2. Đế Chế "Master Realm"
Khi mới Cài đặt Keycloak, nó tự sinh ra một Realm mặc định tên là `master`.
Đừng nhầm lẫn: `master` KHÔNG PHẢI là chỗ để bạn chứa Data của Dự Án.
**Master Realm là Nơi Cư Ngụ Của Các Vị Thần (Super Admins).**
- Thằng User nằm trong Master Realm có Khả Năng Xóa, Xây, Sửa MỌI REALM KHÁC.
- Master Realm là Realm duy nhất có khả năng "Đứng trên cao nhìn xuống". Các Realm khác hoàn toàn mù tịt về sự tồn tại của nhau.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Làm sao Keycloak Phân tách Dữ liệu dưới Database?
Trong PostgreSQL, Keycloak KHÔNG TẠO ra 2 cái Database riêng cho 2 Realm. Cả 2 Realm Nằm Chung Trong 1 Bảng `USER_ENTITY`.

```mermaid
graph TD
    subgraph "Bảng USER_ENTITY trong Database"
        Row1[ID: 001 | Username: alice | Realm_ID: master]
        Row2[ID: 002 | Username: bob | Realm_ID: vingroup]
        Row3[ID: 003 | Username: alice | Realm_ID: fpt]
    end
    
    Note over Row1,Row3: Chìa khóa Cách ly chính là Cột "Realm_ID".<br/>Khi truy vấn, Lõi Keycloak LUÔN TỰ ĐỘNG append dòng lệnh:<br/>WHERE Realm_ID = 'hiện_tại' vào MỌI CÂU LỆNH SQL.<br/>Điều này cho phép Tồn tại 2 cô Alice ở 2 Realm khác nhau.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Quy tắc Bàn Tay Sạch (Leave the Master Realm Alone)**
> **Sai lầm Tân binh:** Setup Keycloak xong, lười biếng, bèn Add luôn Danh sách User Khách Hàng (Người mua hàng, App Bán hàng) VÀO TRONG MASTER REALM.
> **Hậu quả:** Bạn vừa Giao Chìa Khóa Nhà Hạt Nhân cho Khách Hàng. Bất kỳ sự cấu hình nhầm lẫn Role nào cũng có thể biến Khách hàng thành Super Admin, Xóa Sổ toàn bộ Cụm Máy Chủ.
> **Quy Định Sinh Tử:** Master Realm CHỈ DÀNH RIÊNG cho Kỹ sư DevOps, SysAdmin. Khách Hàng / Nghiệp vụ Công ty BẮT BUỘC phải tạo ra 1 Realm riêng (VD: `Realm-MyCompany`). Không có ngoại lệ.

> [!CAUTION]
> **Bẫy Nổ Tốc Độ (Realm Explosion)**
> Bạn thấy Multi-tenancy quá tuyệt vời, bạn quyết định: "Cứ mỗi Khách Lẻ (B2C) đăng ký tài khoản, Tao tạo luôn cho họ 1 cái Realm riêng!". Sau 1 năm, Keycloak của bạn chứa 10.000 Realms.
> **Thảm họa:** Keycloak thiết kế để Scale Dọc Số lượng User (10 Triệu User trong 1 Realm chạy vèo vèo). Nhưng NÓ KHÔNG THIẾT KẾ để Gánh Hàng Chục Ngàn Realms. Vì mỗi 1 Realm khi Khởi Động sẽ được Tải (Cache) Toàn Bộ Cấu Hình Lên Thanh RAM. 10.000 Realms Sẽ Đốt Sạch 100GB RAM Của Bạn Chỉ Riêng Việc Lưu Cấu Hình.
> **Best Practice:** Chỉ Dùng Realm cho Khách Thuê Lớn (B2B - Tenants lớn). Tối đa Khuyến cáo Vài Trăm Realms / 1 Cụm. Nếu có hàng vạn Tenants Nhỏ, Hãy Nhét Tất Cả Vào 1 Realm và Dùng "Groups/Roles" để Cách Ly.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức mạnh Cấu Hình Độc Lập của Realm:
- Ở `Realm-Vingroup`: Bạn cấu hình Ép Buộc MFA (OTP), Đổi Pass sau 30 ngày, Giao diện Đăng Nhập Màu Đỏ, Tiếng Việt.
- Ở `Realm-FPT`: Bạn Cấu hình Cho phép Đăng Nhập bằng Google, Không Cần MFA, Giao diện Màu Xanh, Tiếng Anh.
- Mọi Cấu hình (Luật Lệ, Giao Diện, Flow) Đều NẰM TRONG PHẠM VI 1 REALM. Đổi cái Này Không Chết Cái Kia. Đó là Tuyệt Phẩm Của Kiến Trúc Cách Ly Hoàn Toàn (Full Isolation).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Tràn Viền (Cross-Realm Administration):**
  - Giám Đốc của Vingroup yêu cầu: "Tao muốn tự quản lý Danh sách Nhân Viên của tao (Tự Add User, Đổi Pass), nhưng Tao Đéo Biết Dùng Lệnh API. Cho tao Giao Diện".
  - **Kiến trúc Cấp quyền Vượt Cấp:** Mặc dù Giao diện Admin Console mặc định nằm ở Master Realm, Nhưng Keycloak Cho Phép Cấp Quyền "Realm Management".
  - Bạn tạo Nick `Boss_Vin` Ở TRONG `Realm-Vingroup`. Xong Bạn Cấp Cho Nick Đó Cái Role Đặc Biệt Trỏ Ngược Lại Giao Diện Admin. Lúc đó `Boss_Vin` đăng nhập Giao Diện Admin Console, Ổng SẼ CHỈ THẤY ĐÚNG 1 CÁI REALM CỦA ỔNG. Ổng tự do chọc ngoáy Realm Vingroup mà Mù Tịt Về Realm Master và Realm FPT. (Delegated Administration).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong URL Truy Cập Keycloak, Đoạn chữ nào là Dấu Hiệu Nhận Biết Vương Quốc Đang Đứng? Chuyện gì xảy ra nếu tôi gõ Sai Tên Realm trên URL Login?**
- **Junior:** Nó nằm trên Đường dẫn. Sai thì báo lỗi 404.
- **Senior:** URL Tiêu Chuẩn Của Mọi Endpoint OIDC/SAML trong Keycloak luôn Bắt Đầu Bằng Cấu trúc Rễ: 
`/realms/{realm-name}/...`
Ví dụ Trang Login: `https://auth.com/realms/vingroup/protocol/openid-connect/auth`
Cái Cột Mốc `{realm-name}` là Thằng Đầu Tiên Bộ Định Tuyến (Router) của Keycloak Đọc Được. Vừa Nhận Request, Nó Cắt Chữ `vingroup` ra, Chạy đi Load Cache Cấu hình của Vingroup.
Nếu Gõ Sai Tên (VD: `vin-group`), Keycloak Không Đi Tìm App, Nó Trả Ngay Lỗi HTTP 404 Realm Not Found Ngay Tại Tầng HTTP Layer Mà Chưa Cần Gọi Xuống Database.

**2. Nếu Cần Copy Chuyển Đổi Môi Trường (Từ Môi trường Test Lên Môi Trường Thật - Production). Làm Sao Mang Theo Toàn Bộ Cấu Hình Từ Realm Này Sang Máy Khác Mà Không Sót Data?**
- **Junior:** Copy cái Database Postgres sang.
- **Senior:** Copy DB Nguyên Khối Là Việc Nguy Hiểm Về Dữ Liệu Tác Nghiệp (Lỡ dính Data cũ).
Keycloak Thiết Kế Chức Năng Cực Kỳ Thượng Thừa: **Realm Export / Import (Dạng JSON)**.
- Bạn Vào Admin Console ở Môi trường Test. Chọn Menu Realm Settings -> Action -> Export.
- Keycloak Nén Toàn Bộ (Cấu hình Client, Flow, Nhóm, Luật Lệ) Vào ĐÚNG MỘT FILE JSON KHỔNG LỒ.
- Mang File JSON đó sang Môi trường Production. Bấm Import. Xong Phim. (Tuyệt vời Hơn, Export Cho Phép Chọn Gắn Kèm Hoặc Không Kèm User Data, Phù hợp hoàn hảo cho Quy trình CI/CD GitOps).

**3. Khái niệm "Brute Force Protection" (Chống Nhập Sai Pass) Có Được Cấu Hình Chung Cho Toàn Bộ Keycloak Không Hay Nằm Riêng Lẻ? Tại Sao?**
- **Junior:** Cấu hình chung cho cả Server.
- **Senior:** Sai. Mọi thứ Đều Bị Bó Hẹp Trong Phạm Vi Realm. Trừ Cấu hình Kết nối Database/RAM.
Brute Force Protection nằm trong Menu Realm Settings.
Lý Do Kiến Trúc: Mỗi Khách Hàng (Tenant) Có Một Khẩu Vị Rủi Ro Khác Nhau. Ngân Hàng (Realm A) Muốn Nhập Sai 3 Lần Là Khóa Vĩnh Viễn. Diễn Đàn Sinh Viên (Realm B) Muốn Nhập Sai 10 Lần Mới Bắt Chờ 1 Phút. 
Do Đó, Để Đảm Bảo Multi-tenancy Thực Thụ (True Multi-tenancy), Việc Khóa Tài Khoản BẮT BUỘC PHẢI DO REALM TỰ QUYẾT. Hệ Quả Là: Nếu Cùng 1 Thằng Hacker Đập Liên Tục Vào Realm A (Nó bị Khóa Mõm Ở Realm A). Nó Chạy Sang Realm B Đập Tiếp, Nó Vẫn Sống (Do Cache Bộ Đếm Fail Count Bị Cách Ly Theo Realm).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Server Administration Guide:** Realms and Multi-Tenancy.
