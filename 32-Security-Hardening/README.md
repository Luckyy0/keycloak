# Chapter 32: Gia Cố Bảo Mật Kép (Security Hardening)

## Giới thiệu (Introduction)
Keycloak sinh ra để bảo vệ các ứng dụng khác, nhưng BẢN THÂN Keycloak có được bảo vệ không?
Nếu một Ngân hàng dùng két sắt cực xịn (Keycloak) nhưng lại để Chìa Khóa Két Sắt ngoài hiên nhà, thì két xịn cũng vứt đi! 
Việc "Gia cố bảo mật" (Hardening) là quy trình BẮT BUỘC TRƯỚC KHI LÊN PRODUCTION. Chương này sẽ trang bị cho bạn tư duy của một Chuyên Gia An Ninh Mạng (SecOps): Từ việc Mã hóa luồng truyền tải dữ liệu (TLS/HTTPS), đến việc dựng Tường Lửa (Firewall/WAF) chặn IP độc hại, và cuối cùng là Kích hoạt Hệ thống phòng thủ tự động chống Dò Mật Khẩu (Brute-Force Protection) bên trong ruột của Keycloak.

## Mục lục (Table of Contents)

### Module 1: Xây Dựng Tường Đồng Vách Sắt (Concepts)
*   **Lesson 1: SSL/TLS Certificates (Lá Chắn Mạng Cốt Lõi):** Tại sao Keycloak từ chối chạy ở Môi trường Production nếu không có HTTPS? Cách tạo Chứng chỉ số tự ký (Self-Signed) để test và Cấu trúc cài đặt Let's Encrypt qua NGINX Reverse Proxy. Bít lỗ hổng MITM (Man-in-the-Middle).
*   **Lesson 2: Firewall & WAF (Tường Lửa Web):** Đừng bao giờ phơi thân Keycloak ra Internet! Cấu hình NGINX/Cloudflare WAF để chặn các đòn tấn công SQL Injection vào API của Keycloak, cách chặn truy cập vào cổng Quản Trị `/auth/admin` từ ngoài Internet.
*   **Lesson 3: Brute-Force Protection (Chống Dò Mật Khẩu):** Hacker dùng Tool thử 10.000 mật khẩu trong 1 phút? Hướng dẫn kích hoạt tính năng Tự động Khóa Tài Khoản (Lockout) khi nhập sai quá số lần quy định, và Cơ chế Tăng thời gian phạt chờ (Wait Increment).

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây Dựng Lô Cốt Hoàn Chỉnh: Chạy cụm Keycloak nấp sau một NGINX Reverse Proxy Bị Khóa Kín. Cấu hình HTTPS tự chế và thực hành cấu hình NGINX cấm tiệt truy cập giao diện Admin từ mọi địa chỉ IP bên ngoài, chỉ cho phép Mạng Nội Bộ (LAN) mở Admin Console.

## Bắt đầu từ đâu? (Where to start?)
Học cách Đóng Dấu Niêm Phong Dữ Liệu tại [Lesson 1: SSL/TLS Certificates](Module-1-Concepts/Lesson-1-TLS-Certificates.md).
