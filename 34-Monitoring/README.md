# Chapter 34: Đôi Mắt Thần Quan Sát (Monitoring & Tracing)

## Giới thiệu (Introduction)
Chạy Keycloak trên Production mà không có hệ thống Giám sát (Monitoring) chẳng khác nào lái máy bay Boeing trong đêm tối mà bịt mắt. Bạn sẽ không biết bao giờ CPU quá tải, bao giờ Database bị thắt cổ chai, và không thể biết 1 request Login bị chậm là do lỗi ở NGINX, Keycloak hay Database.
Ở Chương 30 (Logging), chúng ta mới chỉ chạm tới một phần nhỏ là Đo Số Lượng. Trong Chương này, chúng ta sẽ xây dựng một Hệ Thống Quan Sát Toàn Diện (Observability) chuẩn Mây (Cloud-Native): Từ nhịp đập sức khỏe (Health Check), thu thập chỉ số (Metrics/Prometheus), vẽ biểu đồ đẹp mắt (Grafana), cho đến tuyệt kỹ Theo Dấu Vết Dòng Nhảy Phân Tán (Distributed Tracing) với OpenTelemetry và Jaeger để nhìn xuyên thấu vào từng dòng code đang chạy chậm.

## Mục lục (Table of Contents)

### Module 1: Xây Dựng Hệ Sinh Thái Giám Sát (Concepts)
*   **Lesson 1: Health Check (Nhịp Đập Sự Sống):** Cách kích hoạt và sử dụng điểm mù `/health`. Phân biệt giữa `live` (đang thở) và `ready` (sẵn sàng làm việc) để cấu hình cho Kubernetes Load Balancer.
*   **Lesson 2: Metrics (Chỉ Số Định Lượng):** Bật Micrometer. Hiểu ý nghĩa của các chỉ số quan trọng như `jvm_memory_used_bytes`, `db_pool_active`.
*   **Lesson 3: Prometheus (Kẻ Thu Thập):** Cấu hình Prometheus để đi hút dữ liệu (Scrape) từ Keycloak cứ mỗi 15 giây.
*   **Lesson 4: Grafana (Bảng Điều Khiển):** Vẽ biểu đồ (Dashboards). Cách gắn chuông báo động (Alerts) gửi tin nhắn qua Slack khi CPU Keycloak vượt 90%.
*   **Lesson 5: OpenTelemetry (Tiêu Chuẩn Theo Dấu):** Giới thiệu kiến trúc OTel. Cách bật chế độ Tracing trong lõi Quarkus của Keycloak.
*   **Lesson 6: Jaeger (Thám Tử Phân Tán):** Hình ảnh hóa dòng chảy thời gian. Nhìn thấy rõ một cú Click chuột Mất 10ms ở Nginx, 50ms ở Keycloak và 500ms kẹt ở Database.
*   **Lesson 7: Loki (Hố Đen Hút Log):** Gom log từ tất cả các Container về một chỗ để dễ dàng tìm kiếm (Grep) bằng Grafana thay vì phải chui vào từng Server đọc File.

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây dựng Trạm Giám Sát Đám Mây: Chạy một cụm hoành tráng bằng Docker Compose gồm: Keycloak, Postgres, Prometheus, Grafana, Jaeger, Promtail, Loki. Thực hành tạo lỗi quá tải và quan sát biểu đồ giật lên.

## Bắt đầu từ đâu? (Where to start?)
Học cách bắt mạch cho Keycloak tại [Lesson 1: Health Check](Module-1-Concepts/Lesson-1-Health-Check.md).
