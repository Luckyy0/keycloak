# Chapter 29: Advanced Mappers (Bậc Thầy Nhào Nặn Dữ Liệu Token)

## Giới thiệu (Introduction)
Chào mừng bạn quay lại với thế giới của Protocol Mappers - Trái tim của việc phát hành Token! Ở các chương cơ bản, bạn đã biết cách đính kèm một thuộc tính đơn giản như "Số Điện Thoại" hay "Phòng Ban" của User vào trong cục Access Token. Nhưng trong môi trường Enterprise, chừng đó là chưa đủ.
Khách hàng của bạn sẽ đưa ra những yêu cầu cực kỳ lắt léo: *"Tôi muốn đổi tên Role 'admin' thành 'super_admin' khi nó xuất hiện trong Token, vì hệ thống cũ của tôi chỉ hiểu chữ 'super_admin'! Tôi muốn chèn cứng (Hardcode) một mã số Bí Mật vào tất cả Token của một Client cụ thể! Và nếu có logic nào quá dị biệt, tôi muốn tự viết một đoạn code Javascript chạy ngay trong Keycloak để tính toán ra dữ liệu ném vào Token!"*

Chương này sẽ biến bạn thành một **Protocol Mapper Master**. Chúng ta sẽ khai thác triệt để các Mapper Nâng Cao có sẵn trong Giao diện Admin của Keycloak (Không cần viết code Java SPI). Và đặc biệt, bạn sẽ được học cách kích hoạt tính năng **Script Mapper** (bị khóa mặc định) để viết Javascript nhúng thẳng vào luồng phát hành Token!

## Mục lục (Table of Contents)

### Module 1: Kỹ Thuật Đúc Khuôn Token (Concepts)
*   **Lesson 1: Role Mapping (Đánh Tráo Danh Tính):** Kỹ thuật `Role Name Mapper`. Đổi tên Role từ `user` thành `customer_role`, hoặc lọc bớt Role không cần thiết để giảm bớt độ bự (size) của Token. Rất hữu ích khi tích hợp với các hệ thống không cùng ngôn ngữ phân quyền.
*   **Lesson 2: Hardcoded Claims (Đóng Dấu Tử Bất Biến):** Kỹ thuật `Hardcoded Claim`. Ép cứng một cặp Key-Value tĩnh vào Token của một Client cụ thể. Ví dụ đóng dấu `app_source = "web_portal"` để Backend biết Token này đến từ nguồn nào.
*   **Lesson 3: Script Mappers (Phép Thuật Javascript):** Mở khóa Tính Năng Kín (Preview Feature) của Keycloak. Viết một đoạn mã Javascript trực tiếp trên Giao Diện Admin. Đoạn code này sẽ chạy ngầm mỗi khi Token được sinh ra, tự do tính toán, if-else, nối chuỗi để tạo ra một Claim độc nhất vô nhị!

### Labs & Thực hành (Labs)
*   **Lab 1:** Tạo một **Javascript Mapper**. Viết code lấy năm sinh của Khách Hàng, tính toán tuổi. Nếu Tuổi >= 18, ném vào Token chuỗi `"is_adult": true`. Nếu Tuổi < 18, ném vào `"is_adult": false`.

## Bắt đầu từ đâu? (Where to start?)
Chúng ta sẽ khởi động nhẹ nhàng với [Lesson 1: Role Mapping (Đánh Tráo Danh Tính)](Module-1-Concepts/Lesson-1-Role-Mapping.md) để hiểu cách đổi trắng thay đen các Role trong Token!
