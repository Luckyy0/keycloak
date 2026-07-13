# Chapter 38: Mổ Bụng Lõi Lọc Mật Mã (Spring Security Fundamentals)

## Giới thiệu (Introduction)
Từ chương này trở đi Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, chúng ta sẽ rời khỏi màn hình Admin của Keycloak để bước vào chiến trường khốc liệt nhất của Lập Trình Viên Backend: Tích Hợp Bảo Mật.
Muốn Keycloak bảo vệ được Ứng Dụng Java (Spring Boot) của bạn Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh, bạn buộc phải hiểu tường tận cơ chế Bảo vệ Bẩm sinh của Spring: **Spring Security**. Nó chính là Bức Tường Thành Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp, là Kẻ Gác Cửa Đứng Chắn Trước Mọi Dòng Dữ Liệu Chảy Vào Controller của bạn Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa.

## Mục lục (Table of Contents)

### Module 1: Giải Phẫu Lõi Bảo Mật (Core Architecture)
*   **Lesson 1: Băng Chuyền Và Tâm Cụ (Filter Chain & Security Context):** Hiểu rõ cách Gói Tin Đi Qua Hàng Loạt Trạm Gác (Filter) và Nơi Lưu Giữ Dấu Vết Của Kẻ Đăng Nhập (Security Context).
*   **Lesson 2: Dòng Chảy Thẩm Định (Authentication Flow):** Hành trình Token Bị Phân Tích Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Chữ Ký Bị Kiểm Tra Qua `AuthenticationManager` Và `AuthenticationProvider`.
*   **Lesson 3: Lưỡi Gươm Trừng Phạt (Authorization):** Cách dùng Quyền (Roles) Phán Quyết Kẻ Nào Được Gọi Hàm Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa (`@PreAuthorize`) Bằng `AuthorizationManager`.
*   **Lesson 4: Cắm Chốt Kiểm Soát (Custom Filter):** Tự Tay Viết Một Trạm Gác Riêng Biệt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Gắn Vào Băng Chuyền Để Chặn Đứng Những Request Nguy Hiểm Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.

### Labs & Thực hành (Labs)
*   **Lab 1:** Sơ Cứu Băng Chuyền: Trực tiếp Code ra một Custom Filter log toàn bộ hành vi Đăng nhập giả lập Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng, chặn IP xấu và thả cửa cho User Admin Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.

## Bắt đầu từ đâu? (Where to start?)
Khám phá Băng Chuyền Sinh Tử tại [Lesson 1: Filter Chain](Module-1-Concepts/Lesson-1-Filter-Chain-and-Context.md).
