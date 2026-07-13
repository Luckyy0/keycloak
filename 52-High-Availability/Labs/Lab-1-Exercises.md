> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn thực hành xây dựng một cụm (Cluster) Keycloak với độ Sẵn sàng Cao (High Availability), bao gồm việc thiết lập Load Balancer (HAProxy) hỗ trợ Phiên dính (Sticky Sessions) và sử dụng cơ chế Tìm kiếm Node bằng Cơ sở dữ liệu (JDBC_PING).

## 1. Kịch bản Thực hành (Lab Scenario)

Trong môi trường Production doanh nghiệp, để tránh sự cố hệ thống sụp đổ hoàn toàn (Single Point of Failure), bạn cần chạy nhiều máy chủ Keycloak đồng thời. Bài thực hành này mô phỏng một môi trường với:
- 1 Bộ cân bằng tải (HAProxy) phân phối lưu lượng người dùng.
- 2 Node Keycloak chạy ở chế độ Cụm (Cluster) và phân tán phiên bộ nhớ bằng Infinispan.
- Các Node sử dụng giao thức JGroups `JDBC_PING` để tự động dò tìm nhau mà không cần cấu hình IP tĩnh.
- 1 Cơ sở dữ liệu PostgreSQL chia sẻ chung cấu hình.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Máy chủ Linux, Mac hoặc Windows (WSL).
- **Docker** và **Docker Compose** đã cài đặt.
- Tạo một thư mục cho bài Lab: `keycloak-ha-lab`.
- Cấu trúc file:
```text
keycloak-ha-lab/
├── docker-compose.yml
└── haproxy.cfg
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Viết cấu hình HAProxy (Load Balancer)
Tạo file `haproxy.cfg`. File này đảm nhiệm cân bằng tải và thiết lập **Sticky Session** theo thuật toán phân tích cookie của Keycloak (`AUTH_SESSION_ID`).

```haproxy
global
    log stdout format raw local0

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend http_front
    bind *:80
    default_backend keycloak_cluster

backend keycloak_cluster
    balance roundrobin
    # Đọc giá trị cookie do Keycloak sinh ra, ví dụ cookie kết thúc bằng "node1"
    cookie AUTH_SESSION_ID prefix nocache
    
    # Check health để loại node chết. Cookie khai báo tên Node (Routing Suffix)
    server keycloak-node1 kc1:8080 cookie node1 check
    server keycloak-node2 kc2:8080 cookie node2 check
```

### Bước 2: Thiết lập Docker Compose
Tạo file `docker-compose.yml`. Lưu ý rằng thay vì tự viết file XML JDBC_PING phức tạp, từ phiên bản Keycloak Quarkus, ta có thể kích hoạt nhanh thông qua một số biến môi trường.

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
      - ha-net

  kc1:
    image: quay.io/keycloak/keycloak:24.0.2
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_PROXY_HEADERS: xforwarded
      KC_HOSTNAME_STRICT: "false"
      
      # Kích hoạt Infinispan Cache Mode
      KC_CACHE: ispn
      # Cấu hình JGroups sử dụng JDBC_PING thay vì UDP Multicast
      KC_CACHE_STACK: tcp
      JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING
      
      # Cấu hình Routing Suffix cho Sticky Session
      KC_SPI_STICKY_SESSION_ENCODER_INFINISPAN_ROUTE: node1
    command: start-dev
    depends_on:
      - postgres
    networks:
      - ha-net

  kc2:
    image: quay.io/keycloak/keycloak:24.0.2
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KC_PROXY_HEADERS: xforwarded
      KC_HOSTNAME_STRICT: "false"
      KC_CACHE: ispn
      KC_CACHE_STACK: tcp
      JGROUPS_DISCOVERY_PROTOCOL: JDBC_PING
      KC_SPI_STICKY_SESSION_ENCODER_INFINISPAN_ROUTE: node2
    command: start-dev
    depends_on:
      - postgres
    networks:
      - ha-net

  haproxy:
    image: haproxy:2.8
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    ports:
      - "80:80"
    depends_on:
      - kc1
      - kc2
    networks:
      - ha-net

networks:
  ha-net:
```

### Bước 3: Khởi chạy Cụm (Cluster)
Sử dụng dòng lệnh để chạy hệ thống:
```bash
docker-compose up -d
```

Để theo dõi quá trình các Node khám phá và ghép cụm với nhau, hãy kiểm tra log của một trong hai Node (ví dụ `kc1`):
```bash
docker logs -f keycloak-ha-lab-kc1-1
```
Bạn phải nhìn thấy dòng log có chữ `ISPN000094: Received new cluster view...`. Nó sẽ liệt kê tên máy tính của cả 2 node trong một mảng `[...]`, chứng tỏ Cluster đã được thiết lập thành công.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Nghiệm thu 1: Kiểm tra Sticky Session
1. Mở trình duyệt ẩn danh, truy cập trang quản trị: `http://localhost/admin/`.
2. Đăng nhập bằng `admin` / `admin`.
3. Nhấn `F12` mở Developer Tools -> Tab `Application` (hoặc Storage) -> `Cookies`.
4. Tìm cookie có tên `AUTH_SESSION_ID`. Giá trị của nó phải có hậu tố cấu hình của bạn, ví dụ: `xxxxxxxxx.node1` hoặc `xxxxxxxxx.node2`. Trình duyệt đã bị "dính" vào Node đó.

### Nghiệm thu 2: Kiểm tra High Availability & Failover (Chuyển đổi dự phòng)
1. Trong khi vẫn đang đăng nhập ở Admin Console, xác định xem bạn đang dính vào node nào qua Cookie (Giả sử là `node1`).
2. Mở Terminal, mô phỏng sự cố bằng cách "giết" (kill) thẳng tay node đó:
   ```bash
   docker-compose stop kc1
   ```
3. Quay lại trình duyệt, thử click vào mục `Users` hoặc tải lại trang Admin Console.
4. **Kết quả kỳ vọng:** Giao diện mất khoảng 1-2 giây để tải lại, nhưng bạn **KHÔNG BỊ ĐẨY VỀ MÀN HÌNH ĐĂNG NHẬP**. Phiên đăng nhập (Session) đã được HAProxy tự động định tuyến sang `node2`, và Infinispan ở `node2` đã cung cấp lại trạng thái Session (vì nó đã lưu bản sao dự phòng từ trước đó). Cookie sẽ tự động đổi đuôi thành `.node2`.

### Khắc phục sự cố (Troubleshooting)
- **Node không Join Cluster (Log chỉ báo view có 1 node):** Mặc dù cấu hình JDBC_PING, nhưng Docker network có thể gặp lỗi phân giải tên hoặc chưa kịp tạo Database (do Node chạy quá nhanh). Hãy thử khởi động lại cả cụm: `docker-compose down` và chạy lại.
- **Lỗi hiển thị "Invalid username or password" hoặc bị mất phiên liên tục:** Đây là biểu hiện của việc cấu hình **Sticky Session thất bại**. HAProxy ném bạn qua lại giữa 2 Node. Kiểm tra lại tham số môi trường `KC_SPI_STICKY_SESSION_ENCODER_INFINISPAN_ROUTE` xem đã khớp với khai báo `cookie node1 / node2` trong `haproxy.cfg` chưa.
