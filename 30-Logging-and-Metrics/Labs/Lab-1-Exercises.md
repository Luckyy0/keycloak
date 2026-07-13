# Lab 1: Cấu hình Logging và Tích hợp Metrics trong Keycloak

> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Cấu hình hệ thống Ghi nhật ký (Logging) thành định dạng JSON và bật hệ thống Metrics để Prometheus có thể thu thập số liệu giám sát.

## 1. Kịch bản Thực hành (Lab Scenario)
Hệ thống Keycloak của công ty bạn vừa được phê duyệt đưa lên môi trường Production. Đội ngũ DevOps yêu cầu hai việc:
1. Toàn bộ Server Logs phải được in ra màn hình console dưới định dạng JSON để hệ thống ELK (Elasticsearch, Logstash, Kibana) dễ dàng phân tích.
2. Endpoint `/metrics` phải được mở trên một port riêng biệt để cấu hình Prometheus tiến hành thu thập hiệu suất hoạt động của Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Docker và Docker Compose đã được cài đặt.
- Hệ thống có khả năng pull image `quay.io/keycloak/keycloak`.
- Các port `8080` (Traffic người dùng) và `9000` (Management/Metrics Traffic) khả dụng.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Tạo cấu trúc thư mục**
Tạo thư mục làm việc và di chuyển vào trong:
```bash
mkdir keycloak-monitoring-lab
cd keycloak-monitoring-lab
```

**Bước 2: Viết tệp Docker Compose với cấu hình theo yêu cầu**
Tạo tệp `docker-compose.yml` có cấu hình truyền tham số khởi động vào Quarkus engine:
```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:22.0.0
    container_name: keycloak-metrics-server
    command: start-dev --metrics-enabled=true --http-management-port=9000
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_LOG_CONSOLE_OUTPUT: json
      KC_LOG_LEVEL: INFO,org.keycloak.events:DEBUG
    ports:
      - "8080:8080"
      - "9000:9000"
```
*Ghi chú:*
- `--metrics-enabled=true`: Bật hệ thống Micrometer.
- `--http-management-port=9000`: Đẩy endpoint `/metrics` ra port riêng.
- `KC_LOG_CONSOLE_OUTPUT=json`: Chuyển định dạng log console sang chuẩn JSON thay vì plain text.

**Bước 3: Chạy hệ thống**
Khởi động container ở chế độ background:
```bash
docker-compose up -d
```

**Bước 4: Tạo ra một số sự kiện (Events)**
- Mở trình duyệt truy cập `http://localhost:8080/admin/` và đăng nhập bằng admin/admin.
- Cố tình nhập sai mật khẩu vài lần để tạo Event lỗi.
- Đăng nhập thành công và truy cập vào Master Realm để thao tác.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu Logging (JSON Format):**
Kiểm tra log của container để xem định dạng JSON có chính xác không:
```bash
docker logs keycloak-metrics-server
```
Bạn phải nhìn thấy các dòng log được in ra dưới dạng `{"timestamp":"...", "level":"INFO", ...}` thay vì các dòng văn bản liền mạch.

**Nghiệm thu Metrics (Prometheus Endpoint):**
Sử dụng curl hoặc trình duyệt để gọi vào management port:
```bash
curl http://localhost:9000/metrics
```
Bạn sẽ nhận được danh sách dài các chỉ số (ví dụ: `jvm_memory_used_bytes`, `http_server_requests_seconds_count`, `keycloak_logins_total`), theo định dạng Plaintext của Prometheus.

**Troubleshooting (Khắc phục sự cố):**
- **Không truy cập được endpoint /metrics:** Kiểm tra xem bạn có đang nhầm lẫn gọi port 8080 thay vì 9000 hay không. Cấu hình `--http-management-port=9000` đã tách biệt endpoint khỏi port 8080.
- **Log không hiển thị JSON:** Đảm bảo biến môi trường `KC_LOG_CONSOLE_OUTPUT: json` được viết đúng chính tả, và Keycloak của bạn là phiên bản Quarkus (từ version 17 trở đi).
