> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thiết lập và cấu hình cụm Keycloak High Availability (HA) với cơ sở dữ liệu dùng chung và bộ cân bằng tải (Load Balancer).

## 1. Kịch bản Thực hành (Lab Scenario)
Trong môi trường Enterprise, việc chỉ chạy một instance Keycloak duy nhất (Single Point of Failure - SPOF) là rủi ro cực lớn. Nếu instance đó gặp sự cố, toàn bộ hệ thống xác thực của tổ chức sẽ ngừng hoạt động. 
Kịch bản của bài Lab này yêu cầu triển khai một cụm (cluster) Keycloak High Availability (HA) cơ bản sử dụng Docker Compose. Cụm sẽ bao gồm:
- Một cơ sở dữ liệu PostgreSQL dùng chung.
- Hai instances Keycloak chạy song song ở chế độ cluster (sử dụng Infinispan để phân tán cache/sessions).
- Một Nginx làm Load Balancer (Reverse Proxy) phân phối lượng truy cập tới hai Keycloak instances này.

## 2. Chuẩn bị Môi trường (Prerequisites)
- **Hệ điều hành:** Linux/macOS hoặc Windows có WSL2.
- **Công cụ:** Đã cài đặt Docker và Docker Compose.
- **Kiến thức:** Hiểu biết cơ bản về các lệnh Docker, khái niệm Load Balancer và mô hình HA.
- **Port:** Đảm bảo các port `80`, `8080`, `8081`, và `5432` không bị trùng lặp hay bị chiếm dụng bởi dịch vụ khác trên máy.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo cấu trúc thư mục
Tạo một thư mục mới cho bài lab và di chuyển vào đó:
```bash
mkdir keycloak-ha-lab
cd keycloak-ha-lab
mkdir nginx
```

### Bước 3.2: Cấu hình Nginx Load Balancer
Tạo tệp cấu hình Nginx để phân phối tải (Round Robin) cho các Keycloak nodes.
Tạo tệp `nginx/nginx.conf` với nội dung:
```nginx
events {}

http {
    upstream keycloak_cluster {
        # Load balancing giữa 2 nodes
        server kc1:8080;
        server kc2:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://keycloak_cluster;
            
            # Forward headers quan trọng để Keycloak hiểu nó đang chạy sau Proxy
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Bước 3.3: Viết Docker Compose File
Tạo tệp `docker-compose.yml` trong thư mục gốc `keycloak-ha-lab`:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    networks:
      - kc-network

  kc1:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev --cache=ispn
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KC_PROXY: edge
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING # Cấu hình cho Infinispan Cluster
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - kc-network

  kc2:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev --cache=ispn
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KC_PROXY: edge
      JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING
    ports:
      - "8081:8080"
    depends_on:
      - postgres
    networks:
      - kc-network

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    depends_on:
      - kc1
      - kc2
    networks:
      - kc-network

networks:
  kc-network:
    driver: bridge
```

### Bước 3.4: Khởi chạy cụm HA
Chạy lệnh sau để khởi động toàn bộ dịch vụ:
```bash
docker-compose up -d
```
Chờ khoảng 30-60 giây để PostgreSQL khởi động, sau đó 2 Keycloak nodes sẽ khởi động và tự động phát hiện nhau (auto-discovery) thông qua Infinispan (JGroups) để tạo thành một cluster.

Kiểm tra log của một node để xác nhận đã tạo cluster:
```bash
docker-compose logs -f kc1
```
Bạn sẽ tìm thấy dòng log báo hiệu tham gia cụm Infinispan, ví dụ: `ISPN000094: Received new cluster view...`

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu kết quả
1. **Kiểm tra truy cập qua Load Balancer**:
   Mở trình duyệt và truy cập: `http://localhost`. Bạn sẽ thấy giao diện Admin Console của Keycloak. Nginx đã định tuyến Request của bạn đến một trong hai Node `kc1` hoặc `kc2`.

2. **Kiểm tra Session Replication (Phân tán phiên)**:
   - Truy cập Admin Console bằng `http://localhost` với tài khoản `admin`/`admin`.
   - Tạo một Realm mới tên là `HA-Test-Realm` và tạo một User mới `testuser`.
   - Stop một node (ví dụ: `docker-compose stop kc1`).
   - Tải lại trang (F5). Bạn vẫn truy cập bình thường và vẫn ở trạng thái đăng nhập. Nginx đã tự động định tuyến lại Request sang `kc2` và `kc2` có đầy đủ Session/Cache do Infinispan đã phân tán.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Lỗi Keycloak không thể kết nối tới Database**: Đảm bảo Postgres container khởi động xong. Nếu Keycloak restart, Docker sẽ tự động thử lại do cấu hình `depends_on`.
- **Lỗi Infinispan Cluster không hình thành (No cluster view)**: Kiểm tra cấu hình `--cache=ispn`. Trong môi trường Production với Docker Swarm hay Kubernetes, `JGROUPS_DISCOVERY_PROTOCOL` (như DNS_PING, JDBC_PING, KUBE_PING) cần được cấu hình chính xác để các node phát hiện lẫn nhau. Ở lab này, chúng ta sử dụng `JDBC_PING` (hoặc mặc định Multicast trong bridge network tùy version).
- **Lỗi Invalid Redirect URI hoặc HTTPS Required**: Khi chạy sau Reverse Proxy, Keycloak cần được cấu hình `KC_PROXY: edge` (hoặc các tuỳ chọn tương đương) và Nginx phải forward đúng headers (`X-Forwarded-For`, `X-Forwarded-Proto`). Nếu bị lỗi chuyển hướng, kiểm tra lại tệp `nginx.conf`.
