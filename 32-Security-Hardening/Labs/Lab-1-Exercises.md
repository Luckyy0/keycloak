> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn thực hành thiết lập một môi trường Keycloak an toàn, bao gồm việc sinh chứng chỉ TLS, chạy Keycloak qua giao thức HTTPS, cấu hình Reverse Proxy (Nginx) chặn các đường dẫn nhạy cảm, và kích hoạt cơ chế chống đoán mật khẩu (Brute Force Protection).

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là chuyên gia bảo mật (Security Engineer) được giao nhiệm vụ "vá" (harden) một máy chủ Keycloak đang chạy bằng cấu hình mặc định thiếu an toàn. 
Trong bài Lab này, hệ thống sẽ được nâng cấp từ trạng thái chạy HTTP với cổng `8080` phơi bày ra bên ngoài lên thành một kiến trúc an toàn: Keycloak giao tiếp an toàn nội bộ qua lớp Proxy, sử dụng giao thức HTTPS, bị chặn quyền truy cập Admin Console từ các địa chỉ IP bên ngoài, và được bật cơ chế tự động khóa tài khoản để chống các cuộc tấn công Brute Force.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Ubuntu/Debian hoặc hệ thống WSL.
- **Docker** và **Docker Compose** đã cài đặt.
- Cài đặt công cụ OpenSSL: `sudo apt install openssl -y`.
- Cấu trúc thư mục bài Lab:
```text
keycloak-security-lab/
├── docker-compose.yml
├── nginx.conf
└── certs/
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Sinh Chứng chỉ số Tự ký (Self-Signed Certificates)
Trong môi trường thực tế, bạn sẽ mua chứng chỉ từ CA thật. Tuy nhiên, trong Lab này, ta dùng OpenSSL để tạo tự động cặp Khóa bảo mật (Keypair) lưu trong thư mục `certs`:

```bash
mkdir -p certs
openssl req -x509 -newkey rsa:4096 -keyout certs/keycloak.key -out certs/keycloak.crt -days 365 -nodes -subj "/CN=localhost"
```

### Bước 2: Cấu hình hệ thống Docker Compose
Tạo file `docker-compose.yml` để chạy Keycloak và Nginx (với tư cách Reverse Proxy).

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:24.0.2
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin_password
      KC_PROXY_HEADERS: xforwarded
      KC_HOSTNAME: localhost
    volumes:
      - ./certs:/etc/x509/https
    command: start-dev --https-certificate-file=/etc/x509/https/keycloak.crt --https-certificate-key-file=/etc/x509/https/keycloak.key
    # Chỉ mở cổng nội bộ cho Nginx, không expose trực tiếp ra Host
    networks:
      - kc-net

  nginx:
    image: nginx:latest
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - keycloak
    networks:
      - kc-net

networks:
  kc-net:
```

### Bước 3: Cấu hình Nginx chặn đường dẫn Admin Console
Tạo file `nginx.conf` với luật (rule) chặn các truy cập từ ngoài vào `/admin`.

```nginx
events {}

http {
    upstream keycloak {
        server keycloak:8443;
    }

    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /etc/nginx/certs/keycloak.crt;
        ssl_certificate_key /etc/nginx/certs/keycloak.key;

        # Chặn toàn bộ quyền truy cập vào Admin Console trừ dải IP nội bộ giả định
        location /admin/ {
            deny all; # Trong thực tế thay bằng allow <internal_IP>; deny all;
            return 403;
        }

        # Cho phép các request xác thực người dùng bình thường
        location / {
            proxy_pass https://keycloak;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Bước 4: Khởi chạy và bật tính năng chống Brute Force
1. Chạy hệ thống bằng lệnh: `docker-compose up -d`
2. Để truy cập Admin Console nhằm cấu hình (vì Nginx đã chặn ở ngoài), bạn phải chuyển tiếp cổng (port forwarding) qua đường vòng, hoặc tạm thời vào bằng IP Docker. Đối với Lab, bạn có thể comment lại dòng `deny all;` trong Nginx, tải lại Nginx, truy cập `https://localhost/admin/`, sau đó đăng nhập bằng `admin/admin_password`.
3. Khi đã vào Admin Console, chuyển tới `Realm Settings` -> Tab `Security Defenses` -> Tab `Brute Force Detection`.
4. Gạt **Enabled** sang vị trí `ON`. Cấu hình thông số:
   - Max Login Failures: `3`
   - Wait Increment: `1 Minute`
   - Nhấn **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Nghiệm thu
1. **Kiểm tra HTTPS:** Mở trình duyệt web truy cập `https://localhost`. Trình duyệt sẽ hiện cảnh báo bảo mật (do tự ký). Chấp nhận rủi ro và tiếp tục, bạn sẽ thấy giao diện Keycloak hiển thị với khóa bảo mật (ổ khóa).
2. **Kiểm tra luồng Proxy:** Kiểm tra file log Nginx (`docker logs <nginx_container_id>`) để đảm bảo các request được forward thành công và header được giữ lại.
3. **Kiểm tra Nginx chặn Admin:** Kích hoạt lại đoạn mã `deny all;` trong `nginx.conf` và nạp lại cấu hình Nginx (`docker exec nginx nginx -s reload`). Truy cập `https://localhost/admin/`. Hệ thống phải trả về lỗi **403 Forbidden** sinh ra bởi Nginx.
4. **Kiểm tra Brute Force Protection:** 
   - Đăng xuất khỏi Keycloak. Thử vào Account Console bằng một tài khoản thường (ví dụ: tạo user test).
   - Nhập sai mật khẩu liên tiếp **3 lần**.
   - Tại lần thứ 4, bạn nhập đúng mật khẩu. Giao diện Keycloak sẽ hiển thị "Invalid username or password" hoặc "Account is temporarily disabled". Tài khoản đã bị khóa thành công trong 1 phút.

### Xử lý sự cố (Troubleshooting)
- **Lỗi 502 Bad Gateway:** Nginx không kết nối được tới Keycloak. Kiểm tra xem cấu hình `proxy_pass https://keycloak` có chính xác không (lưu ý dùng giao thức `https` ở Upstream thay vì `http` vì Keycloak đang được bật chế độ TLS `start-dev`).
- **Lỗi Keycloak báo Invalid Redirect URI khi truy cập qua Proxy:** Đảm bảo biến môi trường `KC_PROXY_HEADERS=xforwarded` đã được truyền cho container Keycloak, và Nginx đang gửi các header `X-Forwarded-For`, `X-Forwarded-Proto` một cách chính xác.
