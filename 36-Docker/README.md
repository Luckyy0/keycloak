# Chapter 36: Đóng Tàu Không Gian (Docker)

## Giới thiệu (Introduction)
Trong suốt 35 chương vừa qua, chúng ta đã liên tục gõ lệnh `docker-compose up -d`. Tuy nhiên, đó mới chỉ là việc chúng ta "mượn chiếc xe của người khác (quay.io) để chạy". 
Khi bạn bước vào môi trường Doanh nghiệp, bạn không thể cứ lấy cái Image gốc của Red Hat đem đi chạy Production. Bạn cần phải cài thêm chứng chỉ SSL của công ty, nhét thêm các file cấu hình giao diện (Theme) tiếng Việt, và cài các Custom SPI (Ví dụ như Gửi OTP qua SMS) mà bạn đã viết.
Chương này sẽ dạy bạn cách "Build" (Đúc) một chiếc Tàu Không Gian Keycloak (Docker Image) mang thương hiệu của riêng bạn, tối ưu dung lượng và sẵn sàng cất cánh trên mọi môi trường.

## Mục lục (Table of Contents)

### Module 1: Đúc Tàu Và Quản Lý Bến Cảng (Concepts)
*   **Lesson 1: Đúc Tàu (Custom Image):** Cách viết `Dockerfile` chuẩn Quarkus. Quá trình chia làm 2 bước: Build (Tối ưu hóa Database, gom Theme) và Run (Chạy siêu nhẹ).
*   **Lesson 2: Khoang Hàng & Băng Chuyền (Volumes & Networks):** Làm sao để khi Tàu Vỡ (Restart Container), dữ liệu và chứng chỉ trong khoang không bị bốc hơi?
*   **Lesson 3: Lái Tàu Vào Bão (Production Docker):** Những tiêu chuẩn sống còn khi chạy Docker trên Production: Không chạy bằng quyền root, giới hạn RAM (OOM Killer), và ghi Log an toàn.

### Labs & Thực hành (Labs)
*   **Lab 1:** Xưởng Đúc Tàu: Thực hành viết Dockerfile, nhét một cái Custom Theme và một chứng chỉ Tự Tạo vào trong Image, sau đó `docker build` ra một con Keycloak mang tên công ty bạn.

## Bắt đầu từ đâu? (Where to start?)
Trở thành Kỹ sư Đóng Tàu tại [Lesson 1: Custom Image](Module-1-Concepts/Lesson-1-Custom-Image.md).
