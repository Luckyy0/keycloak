# Chapter 28: Custom Themes (Biến Hình Giao Diện Đỉnh Cao)

## Giới thiệu (Introduction)
Dù Keycloak có mạnh mẽ và bảo mật đến đâu, nếu màn hình Đăng Nhập trông giống như một trang web từ năm 1990 với Logo chữ K thô kệch, khách hàng của bạn sẽ bỏ chạy ngay lập tức. Sếp của bạn sẽ yêu cầu: *"Gắn Logo công ty vào! Đổi màu nút bấm thành màu Xanh Thương Hiệu! Đổi nền thành hình Cảnh Sát Vũ Trụ! Và quan trọng nhất: Xóa chữ 'Powered by Keycloak' đi!"*

Chào mừng đến với Nghệ Thuật Custom Theme. Ở chương này, bạn không cần phải là một cao thủ Java Backend. Bạn sẽ khoác lên mình chiếc áo của một **Frontend Developer**. Keycloak sử dụng công nghệ Template Engine có tên là **Freemarker** (.ftl) kết hợp với HTML, CSS, và JS thuần túy để render (kết xuất) giao diện. Bạn sẽ học cách "ghi đè" (override) giao diện gốc, cách nhúng CSS của Bootstrap hoặc Tailwind, và cách gọi đa ngôn ngữ (i18n).

## Mục lục (Table of Contents)

### Module 1: Xây Dựng Lớp Áo Mới (Concepts)
*   **Lesson 1: Theme Architecture (Kiến Trúc Kế Thừa Giao Diện):** Khám phá cấu trúc thư mục của một Theme. Tại sao bạn không bao giờ được sửa trực tiếp vào file gốc của Keycloak mà phải tạo ra một bộ Theme mới Kế Thừa (Inheritance) từ Theme `base`.
*   **Lesson 2: Freemarker Templates (Ma Thuật Nội Suy Dữ Liệu):** Học cách đọc hiểu cú pháp `<#if>`, `<#list>`, và nội suy `${message.summary}` của Freemarker. Cách bóc tách một trang `login.ftl` gốc thành giao diện mang đậm tính nhận diện thương hiệu.
*   **Lesson 3: Account Console (Giao Diện Quản Trị Của Người Dùng):** Không chỉ có màn hình Đăng Nhập (Login), bạn còn có thể tùy biến cả màn hình Đổi Mật Khẩu, Cập nhật Profile, Quản lý Token (Account Theme) - một ứng dụng Single Page App (SPA) bằng React ẩn giấu bên trong Keycloak.

### Labs & Thực hành (Labs)
*   **Lab 1:** Tạo một Theme Đăng Nhập mang phong cách **Tối Giản Đen Tuyền (Dark Mode)**, thay đổi Logo, đổi màu Nút bấm, và thêm thông báo Tuân Thủ Cookie. Đóng gói Theme thành file `.jar` và triển khai vào Keycloak bằng Docker.

## Bắt đầu từ đâu? (Where to start?)
Tiến thẳng vào [Lesson 1: Theme Architecture (Kiến Trúc Kế Thừa Giao Diện)](Module-1-Concepts/Lesson-1-Theme-Architecture.md) để nắm bắt luật chơi của thế giới Freemarker!
