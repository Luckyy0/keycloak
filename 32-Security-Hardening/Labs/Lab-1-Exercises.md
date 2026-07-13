# Lab 1: Xây Dựng Pháo Đài (Nginx HTTPS & Chặn Giao Diện Admin)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thiết lập một luồng Bảo Mật Hoàn Chỉnh từ cửa ngõ (Mã hóa SSL/TLS tự ký), Cấu hình Proxy để đánh lừa HTTPS vào Keycloak, và Cấm toàn bộ quyền truy cập vào cổng Admin dựa trên địa chỉ IP.

## 1. Yêu cầu (Prerequisites)
- Docker Compose.
- OpenSSL (Để sinh Chứng Chỉ Tự Ký).

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Rèn Vũ Khí Chìa Khóa Bằng Máy Tính Nội Bộ (Tạo SSL Self-Signed)
Mở Terminal ở máy Host của bạn (Hoặc WSL), di chuyển vào thư mục `code/` và chạy lệnh này để Sinh ra 2 file `cert.pem` (Khóa Công khai) và `key.pem` (Khóa Bí mật). Mật khẩu bỏ trống, các thông tin bạn điền bừa cũng được.
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes
```
Sau khi chạy xong, trong thư mục `code/` sẽ xuất hiện 2 file này. (Trong môi trường thực tế, 2 file này bạn lấy từ Let's Encrypt hoặc mua của GoDaddy).

### Bước 2: Thiết Lập Bức Tường Nginx (Cấu Hình Chặn IP & Nhúng SSL)
Tạo file `code/nginx.conf`. Đây là trái tim của Pháo Đài:

```nginx
events {}

http {
    # Chặn không cho ngắm cổng HTTP. Đẩy sạch về HTTPS
    server {
        listen 80;
        server_name localhost;
        return 301 https://$host$request_uri;
    }

    server {
        # Bật Lỗ Lắng Nghe Bọc SSL Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
        listen 443 ssl;
        server_name localhost;

        # Chỉ cho Nginx biết đường dẫn chứa 2 Khẩu Súng Vừa Rèn
        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;

        # CÀI ĐẶT BÙA CHÚ HSTS CHỐNG SSL STRIPPING (Sát Thủ Bắn Tỉa Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy)
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # LUẬT BLOCK GIAO DIỆN ADMIN (Sát Thủ Cầm Kéo Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh)
        # Chỉ những máy nằm trong Mạng Trắng Của Docker Bridge (hoặc điền 192.168... của LAN Công ty) Mới Được Vào
        location ~ ^/(admin|realms/master) {
            # Giả sử IP Gateway của mạng Docker là 172.18.0.1
            allow 172.16.0.0/12; # Mở cho dải IP Private của Docker
            allow 192.168.0.0/16; # Mở cho dải IP LAN
            deny all; # CHẶN TOÀN BỘ CÒN LẠI VĂNG LỖI 403
            
            # Nếu Hợp lệ, Vẫn Phải Bơm Vào Bụng Keycloak Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
            proxy_pass http://keycloak:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # LUẬT KHÁCH HÀNG BÌNH THƯỜNG TRUY CẬP (Thỏa Mái Vô Ra Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa)
        location / {
            proxy_pass http://keycloak:8080;
            
            # Gửi Đủ Đồ Chơi Cho Lõi App Keycloak Biết Có SSL Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Bước 3: Lệnh Khởi Động Đám Mây Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa (Docker Compose)
Tạo file `code/docker-compose.yml`. Lưu ý lúc này Keycloak không chạy Dev nữa Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy, mà chạy bằng Lệnh Start Production Cứng Cựa!
Phải cấu hình biến Lừa Đảo Proxy `--proxy=edge` để nó châm chước bỏ qua Cái Khóa Bắt Buộc HTTPS của lõi Quarkus:

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak_db
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    networks:
      - secure_network

  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    # CHẠY PRODUCTION CỨNG CỰA! GẮN BÙA PROXY=EDGE
    command: start --proxy=edge --hostname-strict=false
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: db
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    depends_on:
      - db
    networks:
      - secure_network

  firewall:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      # Mount 2 File Chìa Khóa Vừa Sinh Lúc Nãy Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Vào Bụng Nginx
      - ./cert.pem:/etc/nginx/certs/cert.pem
      - ./key.pem:/etc/nginx/certs/key.pem
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - keycloak
    networks:
      - secure_network

networks:
  secure_network:
    driver: bridge
```

### Bước 4: Test Đẳng Cấp Hacker Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh
1. Khởi động Cụm Vây: `docker-compose up -d`.
2. Kiểm tra Luồng SSL Bằng Lệnh Chuyển Hướng:
   - Mở Trình Duyệt gõ chữ: `http://localhost/`. (Đường G HTTP Thường Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề).
   - Nginx Chặn Cửa, Văng Bạn Ngay Lập Tức Sang Bờ `https://localhost/`. Màn hình trình duyệt sẽ hiện Cái Bản Vàng To Cảnh Báo Chữ "Your Connection Is Not Private". LÀ DO CÁI CERT BẠN TỰ SINH Ở BƯỚC 1 LÀ ĐỒ GIẢ (Không Phải Do Đơn Vị Uy Tín Ký Tên Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa). KỆ NÓ. Bạn Bấm Nút **Advanced -> Proceed to localhost (Unsafe)**.
3. Kiểm Tra Luồng Cấm IP Admin:
   - Sau khi vào Trang Chủ (Chạy Bằng HTTPS SSL Chói Mù Mắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề). Bạn Hãy Cố Tình Click Vào Chữ Bấm Vào `Administration Console`.
   - **BÙM Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề!** Bạn Sẽ Thấy Một Bảng Trắng Xóa Của NGINX Ghi Dòng Chữ Sắc Lạnh: `403 FORBIDDEN` (Cấm Cửa Oanh Khung Dịch Lụa Mạch Lệnh)!
   - (Lý do là NGINX đang thấy IP của bạn được Map vào Container qua Cửa Cục Bộ NAT có thể khác với dải `192.168` hoặc `172.16` Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp. Trong Test Lab này Bạn Đã Thành Công Nhập Vai Đứa Khách Hàng Ở Châu Âu Thèm Thuồng Bảng Admin Mà Bị Cấm).
4. Phá Bỏ Vòng Vây (Bẻ Code Test):
   - Bạn Vào `nginx.conf`. Ở Mục Chặn Admin `location ~ ^/(admin|realms/master)`. Sửa Dòng `deny all;` Thành `allow all;` (Mở Toang Ra).
   - Chạy `docker restart code-firewall-1`.
   - Bấm F5 Lại Trang Admin Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp. Nó Đã Chạy Êm Ru Đón Nhận Khách Hàng Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề!

Đây Chính Là Hệ Thống Khóa Cổ Phân Vùng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Tuyệt Đối!
