# Chapter 30: Logging & Metrics (Con Mắt Của Thượng Đế)

## Giới thiệu (Introduction)
Keycloak đã lên Production và đang phục vụ 100,000 Khách hàng. Một buổi sáng đẹp trời, Giám đốc bảo mật gọi điện cho bạn hét lớn: *"Hôm qua có một thằng Hacker đã dùng Tool dò Pass (Brute-force) để bẻ khóa tài khoản của Kế Toán Trưởng! Nó đã làm chuyện đó từ địa chỉ IP nào? Lúc mấy giờ? Nó có thành công không? Và tại sao CPU của máy chủ Keycloak lại tăng vọt lên 100% trong suốt 3 tiếng đồng hồ?"*

Nếu bạn không cấu hình Logging & Metrics cho Keycloak, bạn sẽ hoàn toàn Mù Lòa. Bạn không có bất cứ một bằng chứng nào để trả lời Sếp. 
Chương này sẽ hướng dẫn bạn biến Keycloak thành một "Camera An Ninh" siêu cấp. Ghi lại mọi hành động (Ai đăng nhập, Ai tạo User mới, Ai xóa Client). Đồng thời mở cổng Metrics (Đo lường) để đẩy dữ liệu CPU, RAM, Số lượng Token phát hành sang hệ thống Prometheus & Grafana để vẽ biểu đồ theo dõi trực tiếp (Real-time).

## Mục lục (Table of Contents)

### Module 1: Hệ Thống Giám Sát Phân Tầng (Concepts)
*   **Lesson 1: Event Logging (Theo Dõi Hành Vi Đăng Nhập):** Hướng dẫn bật tính năng ghi Log các sự kiện của Khách Hàng (User Events) như: `LOGIN`, `LOGIN_ERROR`, `LOGOUT`. Làm thế nào để lưu trữ nó vào Database hoặc xuất ra file Log để tích hợp với ElasticSearch (ELK).
*   **Lesson 2: Audit Logs (Theo Dõi Hành Vi Quản Trị):** Khách hàng đăng nhập thì lưu ở User Events. Nhưng nếu một Thằng Admin nào đó (hoặc chính Bạn) bấm nhầm Nút "Xóa" một cái Realm, thì tìm thủ phạm ở đâu? Bài này hướng dẫn cấu hình Admin Events (Audit Logs) để truy vết những kẻ nắm quyền sinh sát.
*   **Lesson 3: Prometheus Metrics (Đo Lường Hiệu Suất Bằng Số Liệu):** Mở cổng `/metrics` của Quarkus. Hướng dẫn cách dùng Prometheus để "hút" dữ liệu hiệu suất của Keycloak và vẽ biểu đồ đẹp mắt trên Grafana. (Bao nhiêu RAM đang chạy, Có bao nhiêu kết nối Database đang mở, Tốc độ phản hồi API là bao nhiêu Mili-giây).

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây dựng Cụm Giám Sát Hoàn Chỉnh (Monitoring Stack). Viết file Docker Compose khởi động 3 Máy chủ: Keycloak, Prometheus, Grafana. Đấu nối chúng lại với nhau để tạo ra một Bảng Điều Khiển (Dashboard) Giám Sát Hiệu Suất Tuyệt Đẹp.

## Bắt đầu từ đâu? (Where to start?)
Bắt đầu với [Lesson 1: Event Logging (Theo Dõi Hành Vi Đăng Nhập)](Module-1-Concepts/Lesson-1-Event-Logging.md) để bật ngay Camera an ninh bảo vệ tài khoản Khách hàng!
