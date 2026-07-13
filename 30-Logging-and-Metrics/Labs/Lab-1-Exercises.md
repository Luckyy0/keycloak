# Lab 1: Giăng Lưới Bắt Sóng Hiệu Suất (Prometheus & Grafana)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay dựng một hệ thống Mắt Thần (Monitoring Stack) với 3 Node: Keycloak (Máy phát tín hiệu), Prometheus (Máy Hút Dữ Liệu), và Grafana (Bảng Vẽ Biểu Đồ).

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Mở Cổng Tín Hiệu Keycloak
Tạo file `code/docker-compose.yml` như sau. Lưu ý cấu hình Keycloak cực kỳ quan trọng:

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    # BẮT BUỘC BẬT --metrics-enabled=true
    command: start-dev --metrics-enabled=true
    environment:
      KC_DB: dev-file
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - 8080:8080
      - 9000:9000 # Mở Cổng Management Metrics Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
    networks:
      - my_monitor_network

  prometheus:
    image: prom/prometheus:latest
    ports:
      - 9090:9090
    volumes:
      # Mount file cấu hình nhắm mục tiêu vào Keycloak
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - my_monitor_network

  grafana:
    image: grafana/grafana:latest
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - my_monitor_network

networks:
  my_monitor_network:
    driver: bridge
```

### Bước 2: Chỉ Điểm Mục Tiêu Cho Máy Hút (Prometheus Config)
Tại thư mục `code/`, bạn tạo một file tên là `prometheus.yml`. File này chỉ cho Prometheus biết phải chạy đi đâu để hút máu:

```yaml
global:
  scrape_interval: 5s # Cứ 5 giây qua nhà thằng kia chọc 1 lần Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy

scrape_configs:
  - job_name: 'keycloak_metrics'
    metrics_path: '/q/metrics' # Nơi giấu Dữ Liệu Của Quarkus Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh
    static_configs:
      # Do chạy chung Docker Network nên gọi thẳng tên Container "keycloak" và cổng "9000" Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
      - targets: ['keycloak:9000']
```

### Bước 3: Đốt Lò!
1. Mở terminal, chạy: `docker-compose up -d`.
2. Kiểm tra Mắt Thần Cấp 1 (Prometheus): Mở `http://localhost:9090`. Vào mục **Status -> Targets**. Nếu bạn thấy Dòng Chữ `keycloak_metrics` màu Xanh Lá Cây (`UP`), nghĩa là Máy Hút đã gắn vòi thành công vào Lõi Keycloak!
3. Kiểm tra Dữ Liệu Sống: Ở màn hình chính của Prometheus (Biểu tượng cái Kính Lúp), gõ tên biến `jvm_memory_used_bytes` rồi bấm Execute. Bạn sẽ thấy con số RAM của Java đang nhảy nhót!

### Bước 4: Vẽ Biểu Đồ Bằng Grafana
1. Mở `http://localhost:3000`. Đăng nhập bằng `admin` / `admin`. (Được hỏi đổi pass thì bấm Skip).
2. Kết Nối Nguồn Dữ Liệu (Data Source): 
   - Bấm `Data Sources` -> Add data source -> Chọn `Prometheus`.
   - Chỗ URL, nhập: `http://prometheus:9090` (Gọi tên Container trong mạng nội bộ).
   - Bấm `Save & Test`. Thấy nút tích Xanh là thành công.
3. Tạo Biểu Đồ Thần Thánh (Import Dashboard):
   - Mở Trình Duyệt Bấm Chỗ Ô Vuông Dấu Cộng `+` Trái Cùng Màn Hình, Chọn `Import Dashboard`.
   - Grafana có sẵn Hàng Triệu cái Bảng Vẽ do Cộng đồng cúng dường. Với Quarkus/Keycloak, hãy nhập ID: `19226` (Đây là mã ID của cái Bảng Vẽ Quarkus Micrometer Cực Chuẩn Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề). Bấm Load.
   - Ở mục Prometheus Data Source (Dưới đáy màn hình), chọn cái Nguồn bạn vừa cấu hình. Bấm Import!
4. Há Mồm Ngạc Nhiên! Hàng chục biểu đồ Xanh Đỏ Tím Vàng hiện ra: JVM Heap, CPU Usage, Garbage Collection Pause Time, HTTP Request Rate!
5. Bạn thử mở một tab khác, Load Trang Admin Keycloak 100 lần liên tiếp (F5 x 100). Sau đó Quay lại Grafana, bạn sẽ thấy Cột Khói HTTP Request dựng ngược vút lên trời xanh! 

Tuyệt Đỉnh Giám Sát Là Đây Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh!
