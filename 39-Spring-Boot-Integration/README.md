# Chapter 39: Vũ Khí Tối Thượng Java (Spring Boot Integration)

## Giới thiệu (Introduction)
Chào mừng bạn đến với Mảnh Ghép Quan Trọng Nhất Của Khóa Học! Keycloak có xịn đến mấy cũng vô dụng nếu Code Java của bạn không biết cách kết nối với nó.
Trong chương này Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần, chúng ta sẽ bắt tay vào Code Thực Tế Bằng Spring Boot Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy. Bạn sẽ biến một ứng dụng Java trống rỗng thành một Cỗ Máy Bọc Thép Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa: Vừa biết Đẩy Người Dùng sang Keycloak Đăng Nhập Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Vừa biết Kiểm Tra Chữ Ký JWT Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp, Vừa biết Truyền Token Xuyên Qua Các Microservices.

## Mục lục (Table of Contents)

### Module 1: Bí Kíp Tích Hợp Spring Boot
*   **Lesson 1: OAuth2 Login & BFF:** Biến Spring Boot thành một Client thực thụ (Web Ứng dụng Server-Side) với khả năng bật màn hình Đăng Nhập Oanh Khung Dịch Lụa Mạch Lệnh, và kiến trúc Backend-For-Frontend bọc Token vào Cookie.
*   **Lesson 2: Resource Server & JWT Validation:** Chuyển hóa Spring Boot thành Backend API Không Trạng Thái Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Cách Spring xác thực chữ ký JWT và Cơ chế Caching JWKS siêu tốc.
*   **Lesson 3: Nhào Nặn Quyền Lực (RBAC & ABAC):** Map Role của Keycloak vào Security Context của Spring. Kỹ thuật Phân quyền Động dựa trên Thuộc Tính (ABAC).
*   **Lesson 4: Chuyền Bóng Giữa Các Vì Sao (Token Relay):** Giao tiếp giữa 2 Microservices. Dùng FeignClient và WebClient để Bế Nguyên Cục Token Của Khách Hàng Chuyền Chéo Sang Dịch Vụ Khác Mà Không Bị Rơi Rớt Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh.
*   **Lesson 5: Giả Lập Trận Đánh (Testing):** Code xịn thì phải có Test Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Cấu hình MockMvc và Testcontainers để giả lập Keycloak ngay trong lúc chạy Unit Test.

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây Dựng Resource Server Bất Khả Xâm Phạm Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề. Cấu hình File `application.yml` Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng, Tự Tay Code Hàm Giải Mã Token Bóc Role Từ Keycloak Nhét Vào Hệ Thống Của Spring Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy.

## Bắt đầu từ đâu? (Where to start?)
Mở màn trận chiến Code tại [Lesson 1: OAuth2 Login](Module-1-Concepts/Lesson-1-OAuth2-Login-and-Client.md).
