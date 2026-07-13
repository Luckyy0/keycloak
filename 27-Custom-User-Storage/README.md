# Chapter 27: Custom User Storage (Trộm Long Tráo Phụng Dữ Liệu Khách Hàng)

## Giới thiệu (Introduction)
Từ đầu khóa học tới giờ, mọi User bạn tạo ra đều được lưu gọn gàng bên trong Database nội bộ của Keycloak (PostgreSQL/MySQL của riêng nó). Ở chương 20 (User Federation), bạn đã biết cách móc Keycloak vào LDAP hoặc Active Directory để đồng bộ dữ liệu. Nhưng ở Thế Giới Thực của các Doanh Nghiệp Cổ Đại (Legacy Enterprises), dữ liệu khách hàng không nằm trong LDAP, mà nằm chỏng chơ ở một cái Database Oracle 10g 20 năm tuổi, hoặc một hệ thống CRM đóng kín chỉ cho phép giao tiếp qua REST API (không hỗ trợ chuẩn OIDC hay SAML gì sất!).

Sếp ra lệnh: *"Hệ thống cũ không được đụng vào, không được phép chuyển dữ liệu sang Database của Keycloak (Vì hàng ngàn App khác vẫn đang xài DB cũ). Nhưng khi Khách hàng Login vào màn hình Keycloak, Keycloak phải biết đường chạy sang cái DB Cổ Đại kia để kiểm tra mật khẩu. Pass đúng thì mới cho vào!"*

Đây chính là đỉnh cao của sự Lươn Lẹo Dữ Liệu: **User Storage SPI**. Keycloak cho phép bạn viết Code Java để đánh lừa lõi của nó. Mỗi khi Keycloak tìm kiếm một thằng User tên là "teo_nguyen", thay vì lục tìm trong DB nội bộ, Code Java của bạn sẽ đánh lái luồng tìm kiếm đó sang Database Oracle hoặc bắn API sang CRM để lôi thông tin "teo_nguyen" về rồi nhào nặn thành đối tượng `UserModel` của Keycloak. Keycloak hoàn toàn KHÔNG HỀ BIẾT User đó đến từ đâu, nó vẫn vui vẻ cấp phát Token như bình thường! Thật ma giáo!

## Mục lục (Table of Contents)

### Module 1: Ma Thuật Đánh Lái Dữ Liệu (Concepts)
*   **Lesson 1: External Database (Kết Nối Trực Tiếp DB Ngoài):** Hướng dẫn dùng JDBC/JPA cắm thẳng vào Database MySQL/Oracle cũ rích bên ngoài. Khi User đăng nhập, tự động chạy lệnh `SELECT * FROM old_users WHERE username = ?` để kiểm tra mật khẩu. (Bài học về `UserStorageProvider`).
*   **Lesson 2: REST Storage (Bọc API Thành User):** Không có DB trực tiếp? Chỉ có một đường link REST API `GET /api/users/teo_nguyen`? Không sao cả! Bài này dạy cách biến các lệnh gọi HTTP thành dữ liệu User của Keycloak. (Bài học về Cache Cục Bộ để tránh Spam API).
*   **Lesson 3: Legacy System Integration (Tích Hợp Hệ Thống Cổ Đại):** Các chiến thuật đỉnh cao khi tích hợp với Hệ Thống Cũ: Vừa đọc từ DB ngoài, vừa nâng cấp Mật khẩu cũ (Ví dụ: MD5) lên thuật toán mã hóa hiện đại (PBKDF2) của Keycloak một cách mượt mà không cần khách hàng đổi Pass!

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây dựng một **User Storage Provider (Liên Kết Cơ Sở Dữ Liệu Ngoại Trú)** Bằng Java. Đấu nối Keycloak (chạy PostgreSQL) sang đọc dữ liệu đăng nhập từ một Database MySQL (giả lập hệ thống cũ).

## Bắt đầu từ đâu? (Where to start?)
Hãy đọc ngay [Lesson 1: External Database (Kết Nối Trực Tiếp DB Ngoài)](Module-1-Concepts/Lesson-1-External-Database.md) để thấu hiểu cách Giao Diện Java `UserStorageProvider` biến phép màu "Trộm Long Tráo Phụng" thành hiện thực!
