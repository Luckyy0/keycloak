# Hướng Dẫn Chạy Môi Trường Monitoring (Keycloak + Prometheus + Grafana)

Thư mục này chứa cấu hình Docker Compose để khởi chạy một cụm Giám Sát Hiệu Suất cấp độ Enterprise (Monitoring Stack). Bao gồm Máy chủ Keycloak đã mở khóa Metrics, Hệ cơ sở dữ liệu chuỗi thời gian Prometheus, và Công cụ vẽ biểu đồ Grafana.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động 3 Máy Chủ
├── prometheus.yml         # Cấu hình "chỉ điểm" mục tiêu cho Prometheus (Nằm chung thư mục code)
└── README.md              # File hướng dẫn này
```

## 2. Cách Vận Hành Và Khám Phá

1. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
2. Đợi 1 phút cho 3 máy chủ khởi động hoàn toàn.
3. Truy cập **Keycloak Metrics (Cấp độ Thô)**: 
   Mở trình duyệt vào `http://localhost:9000/q/metrics`. Bạn sẽ thấy hàng ngàn dòng Text Format (OpenMetrics) đổ ra. Cứ mỗi lần bạn Refresh trang (F5), các con số trong đó sẽ thay đổi nhẹ (Do bộ nhớ sinh ra, CPU chạy...).
4. Truy cập **Prometheus (Trung Tâm Lưu Trữ Thời Gian Thực)**:
   Mở trình duyệt vào `http://localhost:9090`. Vào thanh menu trên cùng chọn **Status -> Targets**. Bạn phải nhìn thấy dòng chữ xanh lá cây `UP` ở mục `keycloak:9000`. Điều đó chứng tỏ Vòi hút của Prometheus đã cắm thẳng vào mạch máu của Keycloak!
5. Truy cập **Grafana (Bảng Điều Khiển Lãnh Đạo)**:
   Mở trình duyệt vào `http://localhost:3000` (Đăng nhập admin/admin, bỏ qua đổi pass).
   - Menu trái -> Data Sources -> Add Prometheus -> Điền URL là `http://prometheus:9090` -> Save.
   - Menu trái (Dấu +) -> Import -> Nhập ID Dashboard là `19226` (Quarkus Micrometer) -> Chọn Data Source Prometheus -> Import!
   - Thưởng thức thành quả Biểu đồ sống động. Thử Spam F5 Màn hình đăng nhập Keycloak (Cổng 8080) và quay lại Grafana để xem cột khói Request Rate vút lên trời!

**Lưu ý Về Phân Tách Mạng:**
Trong file Docker, Cổng Management `9000` được mở ra ngoài `9000:9000` LÀ CHỈ ĐỂ CHO BẠN (Lập trình viên) ĐỌC THỬ Ở BƯỚC 3 THÔI!
Khi Đưa Lên Môi Trường Thật, BẠN PHẢI XÓA DÒNG `9000:9000` BÊN TRONG FILE DOCKER (Ở cục cấu hình `keycloak`).
Vì Prometheus (`http://prometheus:9090`) và Keycloak Cùng Chung Một Mạng Docker Network (Tên là `my_monitor_network`), Nên Prometheus vẫn tự gọi ngầm bằng DNS Container `keycloak:9000` được mà KHÔNG CẦN Cổng 9000 phải chọc lủng ra Firewall Public của máy chủ! Bọn Hacker Đứng Ngoài Internet Sẽ KHÔNG BAO GIỜ chạm được vào cổng `/q/metrics` Cực Kỳ Nhạy Cảm Này.
