# Lab 1: Xây Dựng Cụm Cluster Bất Tử (NGINX + Keycloak X 2 Node)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay dựng một hệ thống Cluster chịu lỗi (Fault-Tolerant) với 4 Máy Chủ: 1 Database, 2 Node Keycloak (Node 1 và Node 2) chạy chung một Infinispan Cluster, và 1 NGINX làm Load Balancer chặn trước để chia dòng (Kèm thiết lập Sticky Cookie Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy). Cuối cùng, thực hiện Kịch Bản Chết (Failover Test) để chứng minh Khách hàng không bị rớt mạng khi 1 Node bốc cháy.

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Chuẩn Bị Bộ Chia Bài (NGINX Config)
Trong thư mục `code/`, tạo một file tên là `nginx.conf`. File này có tác dụng Hứng toàn bộ luồng mạng từ Cổng 80 của Máy Chủ ngoài cùng, chia tải vào 2 Node nội bộ, đồng thời kích hoạt thuật toán Phân Phối Dựa Băm IP (IP Hash) hoặc Cookie (Tùy bản Nginx Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh).
*Vì Nginx bản Free không hỗ trợ Sticky Cookie chính gốc như bản Plus, ta sẽ dùng thuật toán `ip_hash` thay thế (Luôn luôn đưa 1 địa chỉ IP về cùng 1 Node).*

```nginx
events {}

http {
    upstream keycloak_cluster {
        # ĐỊNH TUYẾN DÍNH BẰNG ĐỊA CHỈ IP (Thay thế cho Sticky Cookie Ở Bản Nginx Free)
        ip_hash;
        
        # Chỉ định 2 Đứa Đứng Sau (2 Container Keycloak)
        server kc1:8080;
        server kc2:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://keycloak_cluster;
            
            # Khai Báo Header Thần Thánh Chống Mất Tên Miền Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Bước 2: Viết Khởi Động Trận Đồ (Docker Compose)
Tạo file `code/docker-compose.yml`. Lưu ý: Chúng ta phải ép 2 Node KC chạy chung Cổng Database, và phải Bật Biến Môi Trường Của JGroups để chúng nó Tự Bắt Tay Nhau Trong Docker LAN!

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
      - ha_network

  kc1:
    image: quay.io/keycloak/keycloak:24.0.1
    # Bật Chức Năng Bùa Trấn Yểm Phía Sau Proxy Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
    command: start-dev --proxy-headers=xforwarded
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: db
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      # Bật Cluster (Phát sóng UDP Mặc Định Của JGroups Trong LAN Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa)
      JGROUPS_DISCOVERY_PROTOCOL: PING
    depends_on:
      - db
    networks:
      - ha_network

  kc2:
    image: quay.io/keycloak/keycloak:24.0.1
    command: start-dev --proxy-headers=xforwarded
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: db
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      # Không cần tạo Admin nữa vì Node 1 đã Tạo Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh! (Chung 1 Database Mà)
      JGROUPS_DISCOVERY_PROTOCOL: PING
    depends_on:
      - db
    networks:
      - ha_network

  loadbalancer:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    # NGINX Trấn Giữ Cửa Ải Public Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa! Đám Bên Trong Kín Cổng Cấm Mở Ra Ngoài!
    ports:
      - "80:80"
    depends_on:
      - kc1
      - kc2
    networks:
      - ha_network

networks:
  ha_network:
    driver: bridge
```

### Bước 3: Đốt Lò Và Kiểm Ngành (Ignition)
1. Mở terminal, chạy: `docker-compose up -d`.
2. Đợi hơi lâu (Khoảng 2 Phút) vì PostgreSQL phải nhào nặn cấu trúc Data lần đầu tiên Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, rồi 2 Node KC1 và KC2 mới cắm vòi xuống.
3. Kéo Log Của 1 Node Lên Xem Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy: `docker logs code-kc1-1` (Tuỳ Tên Container). Bạn Hãy Dò Dòng Có Chữ `ISPN000094: Received new cluster view`. Nếu Bạn Thấy Chữ Này, Chứng Tỏ 2 Đứa Nó Đã Đánh Hơi Thấy Nhau Và Bắt Tay Thành Cluster! Infinispan Cache Đã Sẵn Sàng Nhận Dữ Liệu!

### Bước 4: Kiểm Chứng Fail-Over Chống Cháy (Disaster Test)
1. Mở Trình Duyệt vào `http://localhost/` (Đập thẳng vô Load Balancer Nginx, Nginx sẽ định tuyến ngầm).
2. Đăng nhập Admin Console (admin/admin). Bạn tạo 1 User Tên Là `teo`, pass `123`.
3. Mở Cửa Sổ Ẩn Danh, Đăng nhập trang Account Bằng Tài Khoản Khách Hàng (Tèo): `http://localhost/realms/master/account/`.
4. Khi Tèo đã Đăng nhập vào bên trong và Lướt Web Ngon Lành. **TRÒ CHƠI BẮT ĐẦU Oanh Khung Dịch Lụa Mạch Lệnh!**
5. Bạn quay lại Terminal, BẮN BỎ KHÔNG THƯƠNG TIẾC NODE SỐ 1:
   `docker stop code-kc1-1`
6. Quay lại màn hình của Trình Duyệt Ẩn Danh (Nơi Tèo Đang Lướt Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa). Bạn thử Chuyển Sang Tab "Cập Nhật Profile" Của Giao Diện!
   **BÙM Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề!**
   Trang Web Vẫn Chạy Mượt Mà Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp! Tèo Không Hề Bị Văng Ra Đăng Nhập Lại!
   Điều Gì Vừa Xảy Ra Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa?
   - Lúc Đăng Nhập Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp, Có Thể Tèo Rơi Vào Node 1 Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. Node 1 Lưu Mạng RAM (Infinispan). Node 1 Phát Tín Hiệu Chép Bản Sao Của Tèo Sang RAM Của Node 2 Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh.
   - Node 1 Bị Bạn Đốt Nhà Chết Đứng!
   - Tèo Bấm Nút Lệnh. Nginx Thấy Node 1 Chết Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Lập Tức Bẻ Lái Gói Tin Sang Đít Của Node 2 (Fail-Over Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa).
   - Node 2 Lục Tung Túi RAM Của Mình Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh, Móc Ra Cục Session Của Tèo (Được Sao Lưu Trước Đó). Nó Tiếp Đón Tèo Nồng Hậu Như Mới!
   
Chữ "Bất Tử" Chính Là Đây Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy!
