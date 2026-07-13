# Chương 05: Nghệ thuật Khai hoang Vương Quốc (Realms & Multi-Tenancy)

> [!NOTE]
> Chào mừng đến với Chương 5. Ở các chương trước, bạn đã Xây xong Hạ tầng Lõi, đã Dựng Cụm HA Đám mây. Giờ là lúc bạn chia cắt Cỗ máy đó thành các "Vương quốc" độc lập. Keycloak sinh ra từ trong máu để phục vụ Khái niệm Đa thuê bao (Multi-Tenancy). Mỗi Realm là một Ốc đảo biệt lập hoàn toàn. Không ai biết ai, không ai chạm vào data của ai, dù dùng chung một Cụm Server Lõi và chung một Database Bề Đáy.

## Mục tiêu của chương
- Thấu hiểu Khái niệm Phân cắt Đất đai (Realm) và Tội ác Tày trời Trảm Thủ khi cho phép Dân thường/App thường chui vào `Master Realm`.
- Khám phá các Điểm Nóng Setting Sống Còn ở Mức Realm (Tokens Lifespan, Sessions timeout, Trận Đồ Bát Quái Brute Force Protection).
- Quyền Năng Xóa Bỏ Dấu Vết: Đa ngôn ngữ (Localization) và Tùy biến Giao diện (Themes) - Thay áo cho Keycloak để không ai nhận ra nó là Keycloak, Biến Nó Thành Thương Hiệu Tự Trọng Công Ty.
- Nghệ thuật Di Dời Xuyên Bang: Import và Export Realms. (Cảnh báo tại sao Export lại là mồi lửa cực kỳ nguy hiểm nếu Bị Lộ Mật Khẩu Database JSON Trắng).

## Cấu trúc bài học
Chương này đi sâu vào Bảng Cấu Hình Admin của Cụm:

- **Nhóm 1: Cơ sở Pháp Lý Vương Quốc (Quản Trị Bề Mặt)**
  - `Lesson-1-Realm-Configuration.md`: Trận Chiến Multi-Tenancy. Tại sao Master Realm lại Cô Độc Dành Riêng Cho Quỷ Thần Quản Trị?
  - `Lesson-2-Realm-Settings.md`: Kìm Hãm Sức Mạnh Token, Định Ranh Giới Sống Session Nhớ Mạng, Và Phép Đuổi Khách Sai Pass Brute Force.
  - `Lesson-3-Localization.md`: Giao Tiếp Tiếng Người. Dịch Thuật Đa Ngôn Ngữ i18n Chọc Vỡ Giao Diện Mặc Định Trúc Trắc.
- **Nhóm 2: Quyền Năng Thay Áo Vận Tiêu (Tùy Biến Lõi)**
  - `Lesson-4-Themes.md`: Thay Áo Giáp Bằng Freemarker/HTML. Lột Bỏ Logo RedHat Đỉnh, Nhét Hình Công Ty Trọng Tâm Khách Hàng.
  - `Lesson-5-Realm-Import.md`: Phép Màu Nhồi Database Cấp Tốc Bằng File Khung Lệnh Json Khởi Nguồn Kẽ Nhanh Rụng.
  - `Lesson-6-Realm-Export.md`: Phép Rút Củi Đáy Nồi Xé Mã Trọng Mạng Chạy Bỏ Lõi (Sao Lưu Rút JSON Ngầm).

## Hướng dẫn thực hành (Labs)
- Bài Lab cuối chương sẽ Dẫn Bạn Cầm Cờ Viết Một Cái Mạng Theme Custom Siêu Ngầu Bằng CSS Nhẹ, Chèn Vào Trình Khởi Động Đóng Gói Nhựa Ảo Và Nhổ Tận Gốc Chữ Keycloak Trên Khung Login. Sau Đó Xuất Data Vương Quốc Trút Ngược Thành Tờ Giấy Lệnh JSON Lưu Lại Đời Sau.
