# Lesson 1: Biên Giới Kẽ Nứt (Realm Configuration & Multi-Tenancy)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Hiểu ranh giới Lãnh thổ Tuyệt Đối Của Realm. Realm là công cụ giúp Tập đoàn Vingroup dùng chung 1 Server Keycloak nhưng Tách Biệt Hoàn Toàn 3 Công Ty Con (Vinmec, Vinfast, Vinhomes) Mà Nhân Viên Không Hề Biết Chuyện Khác Nhau Mảng Móng. Học Tội Ác Tử Hình Nếu Đụng Vào Master Realm.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản Chất Đa Thuê Bao (Multi-Tenancy)
Một Máy Chủ Keycloak Khi Khởi Chạy Lên Nắm Mạng Đáy Cứng Tốn Tận Hơn 1GB RAM. Thật Quá Lãng Phí Nếu Nó Chỉ Phục Vụ Đúng 1 App Bán Hàng Trọng Lệnh Đơn Giản.
Vũ Khí Lõi Của Keycloak Nằm Ở **Đa Thuê Bao Dạng Realm**.
- Bạn Bấm Lệnh Tạo 1 Realm Tên `Vinfast`. Realm Này Tự Động Tạo Lập 1 Chân Ống Kép Độc Lập Nằm Dọc Ở Đường Dẫn `.../realms/Vinfast`.
- Bạn Bấm Bật Realm `Vinmec`. Nó Cắt Lập Khung Trụ Cứng Kẽ Mạng `.../realms/Vinmec`.
- Admin Quản Lý Vinmec KHÔNG THỂ Bấm Mở Nhìn Thấy Bất Kỳ Tên Tài Khoản, Session Đăng Nhập, Hay Cấu Hình Của Thằng Vinfast. Mọi Mảng Mã Được Khóa Đóng Kín Cấu Cắt Chữ Bức Tường Đáy Database UUID Ngầm Ngăn Trục Tĩnh Không Vượt Lỗ Hổng Nào! (Nhớ Lại Bài Lab 1 Chương 3 Soi Postgres Thấy REALM_ID Dày Chặt Cột Không?).

### 1.2. Thượng Đế `master` Và Đám Bề Tôi
Khi Khởi Lên, Keycloak Đẻ Sẵn Tặng Bạn Một Lãnh Thổ Duy Nhất Có Tên Là `master`.
Sự Vĩ Đại Và Sự Nguy Hiểm Của Vùng Đất `master`:
- Đứa Nằm Ở Realm Này (Tức Khách User Gắn Tại Realm Này) CÓ QUYỀN Mở Rộng Bàn Tay Chạm Sang Khung Admin Của TẤT CẢ Các Realm Khác Đang Chạy Sống (Nắm Quyền Quản Trị Hệ Trục Server Tĩnh).
- Chính Vì Quyền Lực Xé Tường Này, Realm `master` Được Định Nghĩa Là Lãnh Thổ Dành Cho **Quản Trị Viên Hạ Tầng (Super Admins)**. KHÔNG BAO GIỜ CHO PHÉP Mở Rộng Trút Rác Lệnh Đưa Khách Web App Thường Nằm Ở Vùng Quỷ Thần Trọng Khí Này!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Bức Tường Cách Ly Kẹp Mạng Khi Router Keycloak Đón Khách Rớt Cổng OIDC (Multi-Tenancy Routing Trọng Cấu Phẳng Mỏng):

```mermaid
graph TD
    subgraph "Cách Lý OIDC Xác Thực Dọc Mảng Cắt Nghẽn Sóng Lệch Đuôi Tên Realm"
        AppVFast[Web App Vinfast Đưa Khách Đăng Nhập]
        AppVMec[Web App Vinmec Đưa Khách Bệnh Viện Tới]
        
        Router[Mạng Router Bọc HTTP Của Máy Keycloak Tại Lõi 8080]
        
        R_VFast((Realm Kín Vinfast Lệnh Vực))
        R_VMec((Realm Độc Lập Vinmec Trọng Khí))
        
        AppVFast-->|Gửi Request Cực Dài Có Kẽ: /realms/Vinfast/protocol/openid...| Router
        AppVMec-->|Gửi Request Đuôi Kẹp Gắn Nút: /realms/Vinmec/protocol/openid...| Router
        
        Router-->|Máy Parse Chữ Cắt Kéo Chữ 'Vinfast' Ép Gọi Data Chỉ Đáy Kẽ Lệnh Database UUID Của Nó Bọc Khung Không Mở Rỗng Thừa 1 Dòng Mạng Bên Kia| R_VFast
        Router-->|Phân Dòng Cách Ly Cắt Khung Tức Khắc Nhắm Trúng Realm 2| R_VMec
        
        Note over R_VFast,R_VMec: Ngay Ở Bức Tầng Đầu Tiên URL Mạng.<br/>Mọi Yêu Cầu Chặn Áp Lực OIDC Buộc Trúng Tên Cứng Ngắc Của Lãnh Thổ.<br/>Realm Giống Như Dãy Trọ Nhiều Phòng, Khách Đứng Lỗ Nào Cắm Chìa Khóa Phòng Đó Lập Tức Bị Giới Hạn Tầm Nhìn Kín Trong Phòng Xóa Rỗng Căn Cạnh.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tội Ác Ngu Ngốc Nhất Ngành IT (Xây Dựng Ứng Dụng Trong Master Realm Chặn Lỗ Sụp Trắng Hạ Tầng Phế Tương Lai Quản Trị Hệ Đáy Gãy Trục Nóng Cụm Chết Không Rút Sợi Nhanh Rẽ Nối):**
> **Bi Kịch Thiếu Đọc Sách:** Thằng Khứa Dev Junior Lười Suy Nghĩ Đăng Nhập Xong Vào Màn Hình Của Cụm Thấy Cái Chữ `master` Ngang Chình Ình Ở Góc Trái. Liền Vui Vẻ Bấm Nhấn Add Client "app_ban_hang". Mở Role Bọc User Gắn Đội Nhồi Hàng Triệu Khách Web Ứng Dụng Rác Bơm Vô.
> **Ngày Đền Tội Trục Cốt:** Công Ty Sáp Nhập Mua Thêm 1 Công Ty Lớn (Cần Bàn Giao Thêm App Khác). 
> Kẻ Dev Kẹt Cứng Không Thể Cấp Quyền Cho Bọn Quản Trị Viên Bên Công Ty Mới Nhìn Thấy Khách Hàng Bên Đó Mà Không Bị Trắng Mắt Ngó Thấu User Rác Bên `master` Gốc. Không Thể Chia Rẽ, Không Thể Tách Data Cắt Export Trút Dời Sang Server Mới (Vì Nó Liền Cứng Băng Tầng Lệnh Dọc Rút `master` Của Lõi Server Cấm Đục Chạm Ngược Phép Tắt Xóa Kẽ Lớn Nguồn).
> **Điều Luật Bất Khả Chạm (The Golden Rule Of Multi-Tenancy):** Lãnh Thổ Mẹ Master Realm Tuyệt Đối Giữ Sạch Trơn Chu Không Tạo Bất Kỳ Lệnh Code Trái, Không Client Nhựa Rác Hở Chạm Khách! Vô Phát Tạo Realm Mới Ngay (VD: "AppKinhDoanh") Để Có Thể Thoải Mái Gán Quyền Cấu Đập Database Tắt Rỗng Chỉnh Riêng Bất Biến Rẽ Lõi Rất Sạch Bền Vững Đời Giao!

> [!CAUTION]
> **Khóa Lệnh Lầm Tên Miền Realm Chạm Ngược Gãy Tên Khung Tắt Báo Lỗi HTTPS Sóng 503 Ác Chết Trực Đục Lệnh Đóng Khung Mạng (Realm URL Name Thép Trục Mũi Đáy)**
> Khi Bấm Nút Lệnh Add Realm. Tên Của Cái Lãnh Thổ Này Phải Cực Kỳ Trắng Sạch Lệnh Dấu Ký Tự Rác Nhựa Tĩnh Bọt.
> Tại Sao? Bởi Vì Cái Tên Này Mặc Định Được Nhồi Đâm Xuyên Thẳng Ra Đứng Mạch Nằm Lõi Mặt Tiền Làm Đường Dẫn Mạng Internet! (`https://sso.com/realms/Tên_Của_Bạn/...`).
> Nếu Bấm Tên Tự Do Đục Băng `Vinmec Group` (Có Dấu Cách Phẳng Không Rỗng). Nó Xé Luồng URL Báo Khung Mã Mã Trữ Lỗi Trầm `%20` Gãy Mạng Khách Đứt Cửa Không Vô Trúng OIDC Sụp Sóng Lỗi Lưới Bức Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế! Đặt Tên Không Dấu Cấp Sạch Viết Thường (Kebab-case Đỉnh Chóp) Bất Sát Giao.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Tạo Phân Khung Vùng Đất Vương Quốc Bằng Lệnh Ngắn Quyền API Trống (Xây Nhà Cắt Cụm Bằng Lệnh Quản Trị Kcadm Kéo Sát Chạm Khung CLI Mạng Đáy Cứng Tách Bọc Giao Xé Tức Thì):
Trong Code Mạch Đáy, Ngoài Bấm Chuột, Keycloak Cho Kẻ Lệnh Dev Tự Cắt Đất Bằng API Dòng Lệnh Cực Nhanh:
```bash
# 1. Đăng Nhập Chứng Tỏ Mình Là Vua Tại Tòa Master (Phải Rút Kiếm Tại Thùng Máy Kín Kẽ Nhanh Mới Chấp Nhận Lệnh Đỉnh)
./kcadm.sh config credentials --server http://localhost:8080 --realm master --user admin --password matkhau

# 2. Xé Rạch Tách Khung Phân Đất Realm Mới Tên 'AppKinhDoanh' Ngay Lập Tức Chớp Lệnh Mới Sạch Giới
./kcadm.sh create realms -s realm=AppKinhDoanh -s enabled=true

# 3. Phóng Tụ Lệnh Cấm Người Khách Lạ Tự Nhảy Bấm Nút Đăng Ký Chui Tự Tạo Trái Cổng Khung Realm Này
./kcadm.sh update realms/AppKinhDoanh -s registrationAllowed=false
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Ma Trận Phân Thân User Chéo Tương Đồng Đụng Nhau Khung Rác Mạng Trễ Code Báo Rụng Kép (Cross-Realm User Name Trùng Rỗng Bức Lệnh Rụng Cột Database Đáy Đứng Sóng Sụp Mạch API):**
  - Trong Vingroup, Thằng Admin Lệnh Chạy 2 Realm (Vinfast và Vinmec).
  - Có 1 Bác Sĩ Tên Khám Mạng Khung Cấp Username Là `nguyen_van_a` Làm Ở Cả 2 Công Ty Này Đều Có User Nhập Trùng.
  - Phép Báo Trắng Hỏng? KHÔNG HỀ! Keycloak Đáy Rễ Xé Code Cắt Cực Sạch.
  - Cấu Trúc Khung Mảng Username Của Keycloak Nắm Trọng Bảng PostgreSQL Nằm Đáy Vùng Khuyết Kép Thép Không Bị Giới Hạn Tên Trần Toàn Cục Mà Nó Đính Bức Rào Mã Rỗng Đuôi Bằng **Unique Constraint Trọng Mạch Rỗng Cắn Đôi: (REALM_ID + USERNAME)**. Do Đó, Anh Bác Sĩ Ở Realm Vinfast Là 1 Thực Thể Database Khác Hoàn Toàn Tuyệt Nhiên Với Anh Ta Bên Vinmec. Pass Khác, Khung Token Khác Rời 2 Đứa. Thỏa Mãn Đặc Điểm Phân Mạch Ngầm Rỗng Tuyệt Vời Của Thuật Multi-Tenancy Sạch Nhất Hiện Đại Đỉnh Nền Kiến Trúc Phân Lũ!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Sếp Giao Yêu Cầu. Bên Tập Đoàn Có Lập Ra 1 Thằng App Tổng Quản Lý Cổng Vào Kẽ Phẳng. Người Dùng Ở Bất Kỳ Realm Nào (Vinfast Hoặc Vinmec) Khi Vô App Cổng Đó Bấm Nút, Sẽ Tự Động Được Tụ Vào Cửa Nhận Diện Là Nhân Viên Thuộc Realm Nó Không Cần Trỏ 2 Đường Nối Kéo API Giao Thức OIDC Phẳng Rỗng Nhanh Khác Chữ Tên Đuôi Lệnh Mạch Gãy 404 Kép Bọc Cụm Lệnh Lỗ Trống?**
- **Junior:** Chắc gộp hết vô chung 1 realm cho dễ quản lý đổi sang dùng group chặn Role thôi anh ơi chứ cắt Realm khó code web lắm.
- **Senior:** Phá Hoại Đáy Multi-Tenancy! Tội Rút Giết Cấp! 
Để 1 Web App Chạm Đứng Tổng Chấp Bất Kỳ Tên Trượt Đuôi Mạng Của Client Lệnh Cầm Realm Nào Khách Đăng Nhập Không Đứt, Cấu Phẳng Dữ Nằm Ở Điểm Nối Rỗng Trọng Khí OIDC Khúc Gọi Identity Broker. 
App Phẳng Lập Chạy Gọi Cú OIDC Lệnh Gắn Mở Chạm Đích Mảnh Tụ Vào 1 Cục Realm Tổng Tên `CongTapDoan`. Ở Trong Realm Tổng Này Ta Xé Tính Năng **Identity Provider (IdP) Đáy Lệnh Kéo Brokering** Liên Rễ Đâm Mạng Gọi Ngược 2 Bức Tường Đáy Realm Kia Làm Con Đứng Sân Dưới Đít Nó Xoay Kép Tĩnh Chặn Chờ. Khi Khách Trút Vô Cổng OIDC Tập Đoàn Mở Lên, Bảng Giao Diện Login Form Chạy Sáng Hỏi Kép Nút Sóng Hiện Đỉnh Chọn "Bạn Làm Ở Đâu?". Nhấn Vinfast Nó Chạy Bắn Redirect Trút Mạch Vô Bụng Vinfast Lấy Bọc Khách Về Đổi Thép (Nối Kẽ Mạng Đa Kênh IAM Trọng Khí Cự Đỉnh Giữ Rễ Tách Rời Tuyệt Nhiên Khung Data!).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Server Administration Guide:** Realm Configuration & Multi-Tenancy Principles.
