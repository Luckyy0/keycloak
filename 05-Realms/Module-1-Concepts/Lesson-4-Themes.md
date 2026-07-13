# Lesson 4: Quyền Năng Thay Áo Vận Tiêu (Themes & UI Customization)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Khi bạn đem SSO đi Bán hoặc Chạy Nội Bộ. Khách Hàng Sẽ Cảm Thấy Vô Cùng Cảnh Giác Nếu Form Đăng Nhập Lại Hiện Cái Logo Lạ Hoắc Của "Keycloak". Bài Học Này Khai Mở Bức Màn FreeMarker Để Giúp Bạn Tẩy Trắng Toàn Bộ Giao Diện, Đắp Logo Công Ty Mình Lên Và Biến Nó Thành Thương Hiệu Độc Quyền (White-labeling).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Kiến Trúc Lớp Cắt Theme (Theme Inheritance)
Keycloak Không Phải Dạng "Hard-code" Cứng HTML Bằng Java Gây Đau Khổ. Nó Xây Dựng Khung Giao Diện Dựa Trên Cấu Trúc Kế Thừa Giai Cấp (Inheritance) Bằng Máy Dịch HTML (FreeMarker Templating - `.ftl`):
- **Base Theme (Lõi Đáy Trắng):** Gồm Toàn Code HTML Gốc Cứng Ngắc Xương Xẩu Không CSS. RedHat Cấm Không Cho Ai Sửa Cái Này Khỏi Lỗi Mạch.
- **Keycloak Theme (Áo Giáp Mặc Định):** Kế Thừa Lõi Base Đáy. Đắp Thêm Đống CSS Đẹp Mắt, Chèn Logo Keycloak. (Là Cái Bạn Nhìn Thấy Mỗi Ngày).
- **Vingroup Theme (Theme Tùy Biến Của Bạn):** Đỉnh Cao Là Bạn Tự Khai Báo 1 Cục Theme Của Mình. Bạn Báo Với Server: *"Cho Tao Kế Thừa Thằng Base Đáy. Tao Chỉ Thay Đúng Cái Hình Chữ Nhật Ở Giữa (Thêm Logo Vingroup, Thêm Background Tòa Nhà Landmark 81). Còn Bọn Mã Code OIDC JWT Đằng Sau Tao Giữ Y Nguyên Của Thằng Base"*. Lập Tức Sóng Sạch Kép, HTML Được Máy Render Xé Ra Trọn Vẹn Cực Bền Vững Đời Giao.

### 1.2. 4 Vùng Đất Giao Diện Cho Phép Thay Áo (Theme Types)
Bạn Không Chỉ Thay Mỗi Màn Hình Login! Bạn Có Quyền Tùy Biến:
1. **Login:** Mọi Form OIDC Đăng Nhập, Đăng Ký, Quên Pass, 2FA OTP.
2. **Account:** Trang Cá Nhân Của Khách (Đổi Password, Xem Log Lịch Sử Đăng Nhập Thiết Bị Nào).
3. **Admin:** Giao Diện Dành Cho Sếp (Tuy Nhiên Khuyên Bỏ Qua Trút Giao Diện Này Cho Đỡ Nhọc Vì Sếp Nào Quan Tâm Giao Diện Admin OIDC Trắng Hay Đỏ Đâu, Sống Bền Là Được).
4. **Email:** Giao Diện Đáy Mạch Thư Chúc Mừng, Nhắc Lại Pass Bắn Đi Bằng Lõi SMTP Lên Khung Mail Của Khách Cần Phải Có CSS Đẹp Tôn Cấp Tập Đoàn.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Máy Trộn Render FreeMarker Kéo Dữ Liệu Ép Nhựa Thành Web:

```mermaid
graph TD
    subgraph "Cách Keycloak Ráp Code FreeMarker Tĩnh Kéo Động Nền HTML Phẳng"
        Client[Khách Bấm Trang Đăng Nhập]
        
        KC_Java[Lõi Keycloak Java Lấy Mạch Session Gắn Mã OIDC Gốc Oauth2]
        Data_Context[Túi Dữ Liệu: url.loginAction, realm.name, client.name]
        
        FTL_File[File Lệnh Tĩnh: login.ftl (Của Theme Vingroup)]
        
        KC_Java-->|Tiêm Dòng Động Nhựa Context Vào File| FTL_File
        Data_Context-->|Tiêm Mạch Chữ| FTL_File
        
        FTL_File-->|Máy Máy Trộn FreeMarker Sinh Hủy Xong Lệnh| HTML_Result[Mã HTML Đã Được Render Kéo Chữ Biến Thành Link Code Oauth Đáy Chống CSRF Thép]
        
        HTML_Result-->>Client: Trả Khung Web Hoàn Hảo Cho Người Dùng Kéo.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Kỹ Gắn Theme Nóng (Hot-Deploy Bằng Docker Volume Tránh Đứt Cụm Kéo Chậm Server Cấu Cháy Image Bê Tông)**
> **Tội Ác Viết UI Của Dân Docker Lệch Cột:** Bạn Dev Sửa Cái Nút Đăng Nhập Thành Màu Đỏ Thay Vì Xanh. Bạn Liền Chạy Gõ Lệnh Cấu `Dockerfile` Build Lại Toàn Bộ Nguyên Cục Quarkus Mất 5 Phút. Kéo Cắm Docker Mới Khởi Động K8s Nhanh. BÙM! Quá Trình Làm Font-end Web Mà Phải Build Java Backend Thì Gây Tức Cười Nạn Nhân Ức Chế Bức Tự Rỗng Lệnh 404 Đuôi!
> **Tuyệt Đỉnh Tách Theme Frontend Riêng:** Theme Về Căn Bản Là File Text `.css` và `.ftl` Rỗng Mạch Máu. 
> Trên Môi Trường Dev Local. Bạn Cầm Lệnh Docker Compose **Gắn Volume Thẳng Cột Đáy Nhựa Chạm Máy Tính Mình Vô Đít Tường Kín Của Container**:
> `-v ./my-theme:/opt/keycloak/themes/my-theme`. 
> Lúc Này Bật Thêm Chế Độ Tắt Bộ Nhớ Đệm Lõi Java (Disable Theme Caching Ở File Conf).
> Bạn Code File CSS Đổi Chữ Đỏ Ở Màn Hình Window VSCode, Quay Trái Bấm F5 Chrome Trình Duyệt Bên Cạnh Lập Tức Chữ OIDC Bọc Lõi Keycloak Trở Thành Đỏ Chót Cực Nhanh! (Hot Reload 0 Giây Cấp Bậc Oanh Liệt Code Frontend Dân Sự).

> [!CAUTION]
> **Nỗi Ôm Khối Hận HTML Bị Gãy Form Token Khi Đổi Đuôi Phiên Bản OIDC (Thép Chặn Copy Paste Rác)**
> Dev Làm Theme Thường Có Tật Lấy File `login.ftl` Của Thằng Mẹ Base Copy Hết Cả Nghìn Dòng Về Dán Vào Theme Mình Rồi Sửa Rất Ác Mạch Lõi Code OIDC Ngầm Của Thằng RedHat Sinh CSRF (Token Chống Chặn Form).
> Keycloak Nâng Cấp Từ V22 Lên V24. Bụng Lệnh API Trong Lõi Oauth2 Của Thằng Mẹ Thay Đổi Dữ Liệu Biến Thép Ở 1 Chỗ Tĩnh Nào Đó Đáy Rễ. 
> Nhưng File Theme `.ftl` Của Bạn Nằm Phẳng Dưới Theme Copy Y Nguyên Bản V22 Cũ Khung Mệnh. KHÁCH BẤM LOGIN NHẬP ĐÚNG PASS NHƯNG TOÀN BÁO LỖI ĐỎ SỤP SERVER (Gãy Token Biến CSRF Gắn Dữ HTML Cũ).
> **Luật Thép Sống Còn Giữ Code UI:** Hạn Chế Tối Đa Việc Copy Những File Có Lệnh Kéo Token Khởi Nguồn Sát Đáy OIDC Trọng Nếu Không Cần Thiết. Hãy Sửa CSS Ở File Tách Rời (Nằm Ở Ô `styles=` Trong Tệp `theme.properties`). Chỉ Khi Nào Buộc Phải Dời Cột HTML Phẳng Mới Dám Đè Copy Override Thằng Mẹ Đít File Lệnh `.ftl` Bọc.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Cấu Trúc Tối Giản Nhất Của 1 Cục Khối Theme Kéo Đội Lệnh Giấu Áo:
Thư mục Tự Động Kéo: `themes/vingroup-theme/`
1. Tệp Pháp Lý: `login/theme.properties`
```properties
parent=keycloak
import=common/keycloak
styles=css/my-style.css
```
(Dòng Lệnh Tuyệt Vời Ép Cỗ Máy: Tự Động Kế Thừa Cục Mẹ Keycloak, Tự Lấy Form Mẹ, Chỉ Nhét Chèn Kép Thêm Cái File CSS Riêng Của Tao Đề Vô Thùng Nước Bảng Để Dập Chữ Xóa Màu!).

2. Tệp Áo Giáp Tự Thêu: `login/resources/css/my-style.css`
```css
/* Đục Sóng Dữ Liệu Ném Xóa Hình Khung Trắng Logo OIDC Vô Hình Mạch Base Cũ Của Đít Khung RedHat Ngắn Mạch */
#kc-header-wrapper {
    background-image: url('../img/vingroup-logo.png');
    font-size: 0; /* Xóa Lệnh Chữ Rỗng Đi Kéo Bằng Không Thấy Tên Cũ Nữa Khách Ảo Mờ OIDC Mạng */
}
body {
    background-color: #f1f2f5;
}
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Đâm Nút Lỗ Hổng Nén Thép Theme Nằm Trong Vùng Image Bê Tông Quarkus Không Ăn Trọng Lõi Start (Theme Kẹp Bọc Cấp K8S File Rỗng Mở API Không Nổ Java Build Lệnh Gãy Trúc Đứt Kẽ Đóng Tĩnh):**
  - Dev Build Theme Gói Kéo Thành File `.jar` Kẹp Rỗng Ném Vào Nút Thư Mục Rìa Tĩnh Đuôi Của Image Docker Bọc `providers/`.
  - Nhưng Khi Nắm Cụm Đưa K8s Chạy `kc.sh start`, Server Khung Cứ Nhận Cục Theme Mặc Định Cũ Mèm Không Thấy Theme Lõi Mình Tự Code Nằm Đâu Trong Giao Diện.
  - Vấn Đề Lớn Bọc Ảo Đứt Gãy Quarkus: Vì Thư Mục Providers Chứa Lệnh Theme Jar Bọc. BẮT BUỘC Thằng Build Phải Chạy Lệnh Rút Gắn Code Cứng Thời Gian Đỉnh Oanh (kc.sh build). Nếu Không ÉP Quarkus Tiêu Hóa Theme Ở Lúc Compile Nồi Cháo Image Khung Chạy Nhanh Đáy, Nó Sẽ Không Biết Mạch Ở Đáy Kéo Tĩnh Lệnh Có Cục Theme Rỗng Để Khai Báo Cho Form OIDC Sóng Nút 8080 Kéo Ra Sóng! Mọi Cục Thêm Gắn SPI Giao Đều Phải Qua Vòng Xưởng Đúc Giao Kẹp Lệnh Tái Trọng Start Nằm Ảo Rỗng Đáy!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Vinfast. Cậu DevOps Đã Gắn Đội Theme Vingroup Lên. Nhưng Khách Mở Web Kéo Đít Form Đăng Nhập Điện Thoại Thì Lại Muốn Theme Phải Nằm Cụm Nhỏ Gọn CSS Rút Mảnh Tách Oanh Theo Bản Giao Điện Thoại Dọc Lệch (Mobile Theme Riêng). Nhưng Bảng Realm Chỉ Cho Phép Chọn 1 Theme Login Duy Nhất Cục Bọc Ở Tab Settings. Làm Cách Nào Để Keycloak Lõi Bẻ Luồng Nhận Biết Khách Khung Giao Động Mobile Trút Nhanh Ép Kẽ Trả Form OIDC Sóng Khác Rỗng Trắng Nằm API Chéo Lệch Nhau?**
- **Junior:** Chắc dùng CSS Media Query @media responsive cho nó bóp lại là được rồi anh cần gì theme riêng code cho cực rỗng sóng.
- **Senior:** Lỗi Mất Kiểm Soát Lõi Bọc Sóng Giao Trọng Khi Theme PC Quá Nặng Lệnh Cứu Hộ Đáy RAM. 
Để OIDC Keycloak Đổi Lệ Trục Khung Nén Theme Linh Động Hoàn Toàn Tĩnh (Dynamically Override Theme Per Request Tách Lõi Tốc Sóng Oanh Tạc). Keycloak Có Chừa Khẩu Kẻ Giao Lệnh Ẩn Tên Là `kc_theme`.
Bạn Cầm Client Điện Thoại Web Gọi Lệnh Login Đâm OIDC Trút URL Bắn Mũi. Nhét Kép Nằm Rìa Dọc: `/auth?client_id=xxx&kc_theme=vingroup-mobile`. Lập Tức OIDC Tĩnh Đáy Vứt Khung Theme Mặc Định Chữ Realm Đang Cài Xóa Kẽ Trọng Nhựa. Nó Xé Tốc Phục Vụ Đáy Bắn Mạch Giao Khung Vingroup-Mobile Cực Mỏng Nhẹ Ra Thay Thế Trực Diện Màn Hình. Giúp Trải Nghiệm Khách Mobile Cắt Tải 10MB Dư Rác Background Cháy Tĩnh Kép Giữ Luồng Rất Sạch Oanh Liệt Rút Trọng Lệnh Dữ DB Trống Bất! (Tuyệt Diệu Kép Gọi Động OIDC Bọc Lệnh API Nhựa).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Themes:** Customizing the Login and Account Console.
