> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Triển khai Keycloak và cơ sở dữ liệu PostgreSQL lên môi trường thực tế bằng Docker Compose, đồng thời thực hành tối ưu hóa image (Custom Image) và thiết lập mạng nội bộ.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là Kỹ sư Hệ thống (DevOps Engineer) tại một công ty công nghệ. Dự án mới yêu cầu thiết lập một hệ thống định danh trung tâm bằng Keycloak. Yêu cầu đặt ra là hệ thống không được dùng cơ sở dữ liệu H2 mặc định (chỉ dùng cho Dev), mà phải dùng PostgreSQL để đảm bảo tính bền vững (Persistence). Ngoài ra, hệ thống cần được cấu hình theo tiêu chuẩn Production bằng cách sử dụng Docker Compose với các volume để lưu trữ dữ liệu database và mạng nội bộ để bảo mật.

Trong bài Lab này, bạn sẽ tự tay viết Dockerfile cho Keycloak, viết `docker-compose.yml` và khởi chạy toàn bộ stack.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Máy tính đã cài đặt **Docker** và **Docker Compose** (phiên bản V2 trở lên).
- Có kết nối Internet ổn định để kéo (pull) các Docker images (`quay.io/keycloak/keycloak` và `postgres`).
- Có quyền Administrator/Root hoặc user thuộc nhóm `docker` để chạy các lệnh quản trị vùng chứa.
- Sử dụng Terminal (Bash/PowerShell) và một Text Editor (VS Code hoặc Nano/Vim).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo cấu trúc thư mục dự án
Mở Terminal và tạo một thư mục mới cho bài Lab:
```bash
mkdir keycloak-docker-lab
cd keycloak-docker-lab
```

### Bước 3.2: Viết Dockerfile tối ưu cho Keycloak (Custom Image)
Tạo một tệp tin tên là `Dockerfile` bằng editor của bạn:
```bash
touch Dockerfile
```
Dán nội dung sau vào `Dockerfile`:
```dockerfile
# Sử dụng base image mới nhất của Keycloak
FROM quay.io/keycloak/keycloak:latest as builder

# Cấu hình biến môi trường Build-time
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_DB=postgres

# Chạy build để tối ưu hóa Quarkus
RUN /opt/keycloak/bin/kc.sh build

# Stage 2 (Tùy chọn cho Runtime)
FROM quay.io/keycloak/keycloak:latest
COPY --from=builder /opt/keycloak/ /opt/keycloak/

# Đặt Entrypoint
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
```

### Bước 3.3: Tạo tệp cấu hình Docker Compose
Tạo tệp `docker-compose.yml` trong cùng thư mục:
```bash
touch docker-compose.yml
```
Dán nội dung sau vào:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: kc_postgres
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password123
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - kc_network

  keycloak:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: kc_server
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password123
      KC_HOSTNAME: localhost
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    command: start-dev # Chạy mode dev để bỏ qua cấu hình HTTPS rườm rà trong Lab này, nhưng cấu trúc vẫn là Production ready
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - kc_network

volumes:
  postgres_data:

networks:
  kc_network:
    driver: bridge
```

### Bước 3.4: Khởi chạy hệ thống
Tại thư mục chứa hai tệp trên, chạy lệnh sau để build và khởi động các container:
```bash
docker compose up -d --build
```
Hệ thống sẽ mất khoảng 1-2 phút để tải images, build Keycloak và khởi chạy PostgreSQL. 

Theo dõi log của Keycloak để biết khi nào hệ thống sẵn sàng:
```bash
docker logs -f kc_server
```
Chờ đến khi bạn thấy dòng chữ tương tự như: `Keycloak x.x.x (Quarkus) started in Xms`.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu (Verification):**
1. Mở trình duyệt web và truy cập: `http://localhost:8080`
2. Bạn sẽ thấy màn hình Welcome của Keycloak. Nhấp vào "Administration Console".
3. Đăng nhập bằng thông tin đã khai báo trong file compose:
   - Username: `admin`
   - Password: `admin`
4. Nếu đăng nhập thành công vào màn hình Admin, bạn đã cấu hình đúng!

**Kiểm tra tính bền vững (Persistence Check):**
1. Đăng nhập vào Keycloak, tạo một Realm mới tên là `TestRealm`.
2. Dừng và xóa toàn bộ container:
   ```bash
   docker compose down
   ```
3. Khởi động lại hệ thống:
   ```bash
   docker compose up -d
   ```
4. Đăng nhập lại vào `http://localhost:8080`. Bạn sẽ thấy `TestRealm` vẫn tồn tại do dữ liệu đã được lưu an toàn trong Docker Volume (`postgres_data`).

**Các lỗi thường gặp (Troubleshooting):**
- **Lỗi cổng 8080 đã được sử dụng:** Nếu bạn nhận thông báo `bind: address already in use`, hãy dừng dịch vụ đang chiếm cổng 8080 (ví dụ Tomcat hoặc một container Keycloak khác), hoặc đổi cổng trong file compose (`ports: - "8081:8080"`).
- **Keycloak Crash vì không kết nối được Database:** Chạy lệnh `docker logs kc_postgres` để xem DB có khởi động lỗi không. Đảm bảo tham số `KC_DB_URL` khớp với tên service (tên container) của Database là `postgres`.
