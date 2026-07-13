> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Cấu hình và chạy Keycloak trong chế độ Production tích hợp với PostgreSQL (External Database) và Nginx (Reverse Proxy với TLS Termination).

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn yêu cầu triển khai Keycloak phục vụ hàng ngàn người dùng. Thay vì sử dụng cơ sở dữ liệu H2 nội trú không an toàn cho Production và chạy bằng giao thức HTTP bản rõ, bạn cần thiết lập một môi trường chuẩn:
- Một container PostgreSQL đóng vai trò làm External Database.
- Một container Keycloak được build tối ưu (Optimized Build) kết nối vào PostgreSQL.
- Một container Nginx cấu hình SSL/TLS (Tự ký - Self-signed Certificate) để làm Reverse Proxy, chuyển tiếp (Forward) các Header cho Keycloak xử lý.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Docker và Docker Compose đã được cài đặt trên máy.
- Một chứng chỉ SSL tự ký (Self-signed certificate) dùng cho Nginx.
- Trình duyệt web và công cụ dòng lệnh `curl` để kiểm thử.

Tạo cấu trúc thư mục như sau:
```text
keycloak-prod-lab/
├── docker-compose.yml
└── nginx/
    ├── nginx.conf
    └── certs/
        ├── tls.crt
        └── tls.key
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Tạo chứng chỉ SSL tự ký cho Nginx**
Mở Terminal, đi tới thư mục `nginx/certs/` và chạy lệnh sau:
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=auth.local.com"
```

**Bước 2: Cấu hình Nginx (Reverse Proxy)**
Tạo tệp `nginx/nginx.conf` với nội dung:
```nginx
events {}
http {
    server {
        listen 443 ssl;
        server_name auth.local.com;

        ssl_certificate /etc/nginx/certs/tls.crt;
        ssl_certificate_key /etc/nginx/certs/tls.key;

        location / {
            proxy_pass http://keycloak:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $host;
        }
    }
}
```

**Bước 3: Tạo tệp cấu hình Docker Compose**
Tạo tệp `docker-compose.yml` ở thư mục gốc `keycloak-prod-lab/`:
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  keycloak:
    image: quay.io/keycloak/keycloak:latest
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KC_HOSTNAME: auth.local.com
      KC_PROXY_HEADERS: xforwarded
      KC_HTTP_ENABLED: "true"
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    command: start
    depends_on:
      - postgres

  nginx:
    image: nginx:latest
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/certs:/etc/nginx/certs
    depends_on:
      - keycloak

volumes:
  postgres_data:
```

**Bước 4: Cập nhật file Hosts**
Thêm dòng sau vào tệp `/etc/hosts` (hoặc `C:\Windows\System32\drivers\etc\hosts` trên Windows):
```text
127.0.0.1 auth.local.com
```

**Bước 5: Khởi chạy cụm dịch vụ**
Chạy lệnh:
```bash
docker-compose up -d
```
Xem log của Keycloak để đảm bảo nó đã kết nối thành công tới PostgreSQL và khởi động hoàn tất:
```bash
docker-compose logs -f keycloak
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)
- **Kiểm tra Truy cập HTTPS:** Mở trình duyệt và truy cập `https://auth.local.com`. Trình duyệt sẽ cảnh báo chứng chỉ không an toàn (do chứng chỉ tự ký). Bấm "Proceed" (hoặc "Advanced -> Continue").
- **Kiểm tra Đăng nhập:** Đăng nhập vào Administration Console với tài khoản `admin`/`admin`. Nếu giao diện hiển thị đúng, tức là Reverse Proxy đã forward các tiêu đề đúng cách.
- **Kiểm tra Database:** Kết nối vào container PostgreSQL:
  ```bash
  docker exec -it <postgres_container_id> psql -U keycloak -d keycloak
  ```
  Chạy lệnh `\dt;` để liệt kê các bảng. Bạn sẽ thấy hàng loạt bảng nội bộ của Keycloak (ví dụ: `USER_ENTITY`, `REALM`).

**Lỗi thường gặp (Troubleshooting):**
- *Lỗi "Mixed Content" hoặc "Invalid Redirect URI":* Do Nginx không truyền đủ `X-Forwarded-Proto` hoặc Keycloak thiếu biến `KC_PROXY_HEADERS: xforwarded`. Kiểm tra lại config Nginx.
- *Lỗi "Connection Refused" từ Keycloak:* PostgreSQL có thể mất nhiều thời gian để khởi động. Keycloak thử kết nối quá sớm. Hãy khởi động lại container Keycloak (`docker-compose restart keycloak`).
