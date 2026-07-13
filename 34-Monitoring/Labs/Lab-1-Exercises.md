# Lab 1: Xây Dựng Cỗ Máy Quan Sát Mọi Thứ (Observability Stack)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thiết lập một Cụm Hệ Thống Mắt Thần vĩ đại bằng Docker Compose bao gồm: Postgres DB, Keycloak (Cấp phép Bật Health, Metrics, OTel), Prometheus (Máy Nhặt Số), Grafana (Máy Vẽ Bảng), Jaeger (Máy Dò Bức Xạ OTel). Thực hành tạo 1 Biểu đồ Tải cơ bản.

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Chuẩn Bị Cấu Hình Prometheus
Tạo file `code/prometheus.yml`. File này để chỉ cho Prometheus biết phải đi cắm vòi hút vào miệng con Keycloak lúc nào:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'keycloak'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['keycloak:8080']
```

### Bước 2: Thiết Lập Quái Vật Đám Mây (Docker Compose)
Tạo file `code/docker-compose.yml`. Lưu ý lúc này Keycloak Mở Ban Banh Các Lỗ Cắm:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
      POSTGRES_DB: keycloak
    networks:
      - obs_network

  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    # BẬT TÁNG LOẠN LÊN Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề! Bật Metrics Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy, Bật Health Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Bật Tracing (Phóng Xạ OTel Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh)
    command: >
      start-dev 
      --metrics-enabled=true 
      --health-enabled=true 
      --tracing-enabled=true
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: postgres
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      # Trỏ Lỗ Phóng Xạ OTel Của Keycloak Bay Thẳng Vào Miệng Máy Dò Jaeger Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa (Endpoint Của Jaeger OTLP Port 4317 Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp)
      KC_TRACING_ENDPOINT: "http://jaeger:4317"
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - obs_network

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - obs_network

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    networks:
      - obs_network

  jaeger:
    image: jaegertracing/all-in-one:latest
    environment:
      # Bật Chế Độ Hứng Bức Xạ OTLP Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
      COLLECTOR_OTLP_ENABLED: "true"
    ports:
      # Giao Diện Xem Bảng Của Jaeger Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh
      - "16686:16686"
      # Lỗ Cắm Hút Trực Tiếp (gRPC) Của OTel Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần
      - "4317:4317"
    networks:
      - obs_network

networks:
  obs_network:
    driver: bridge
```

### Bước 3: Khởi Động Và Trải Nghiệm Mắt Thần Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
1. Gõ `docker-compose up -d`. Đợi 1 phút.
2. **Kiểm Tra Sinh Tử Của Keycloak (Health Check Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh):**
   Mở Trình duyệt gõ: `http://localhost:8080/health/ready`. Cười Đắc Thắng Khi Thấy Chữ JSON: `{"status": "UP", ...}`.
3. **Thưởng Thức Mớ Hỗn Độn (Metrics Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa):**
   Gõ `http://localhost:8080/metrics`. Thấy Chữ Chạy Dọc Màn Hình Mịt Mù Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng. Kệ Đó Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa!
4. **Grafana Oai Phong Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:**
   - Vào Grafana `http://localhost:3000` (Pass: `admin`/`admin`).
   - Chọn Connection -> Add Data Source -> Prometheus -> Điền Cổng Vừa Mở `http://prometheus:9090` -> Save!
   - Chọn Dashboards -> Import -> Nhập số thần kỳ **`19248`** -> Chọn Data Nguồn Vừa Thêm Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy. BÙM Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa! Bạn Nhìn Thấy Bảng Điều Khiển Y Hệt Đĩa Bay NASA Về Tình Trạng Của Keycloak Oanh Khung Dịch Lụa Mạch Lệnh!
5. **Dấu Chân Thám Tử (Jaeger Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề):**
   - Về Màn Hình Chính Keycloak Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh `http://localhost:8080`. Bấm Login Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Tạo Realm (Tạo Dấu Vết Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa).
   - Vô Giao Diện Jaeger Ở Máy Đếm Thời Gian: `http://localhost:16686/`. Ở Khúc Search Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh, Chọn Service Là `keycloak`. Bấm Trực Tiếp "Find Traces" Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp. Bạn Nhìn Cái Thác Nước Chỉ Ra Cú Login Vừa Nãy Bạn Tốn Bao Nhiêu Milisec Của Thanh Quản DB Và Thanh Lõi Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề! Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị! Tột Đỉnh DevOps Ở Đây Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa!
