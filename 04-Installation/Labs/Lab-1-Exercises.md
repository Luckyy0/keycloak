# Lab 1: Triển khai Keycloak với PostgreSQL qua Docker Compose

> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Cài đặt và vận hành Keycloak sử dụng Docker Compose, kết nối với cơ sở dữ liệu PostgreSQL. Cấu hình các biến môi trường thiết yếu để khởi chạy chế độ Production.

## 1. Kịch bản Thực hành (Lab Scenario)
Trong môi trường doanh nghiệp (Enterprise), việc lưu trữ dữ liệu của Keycloak (Users, Clients, Sessions) không thể dùng H2 Database mặc định (In-memory) vì nguy cơ mất dữ liệu khi restart container. Bài lab này giả lập tình huống bạn là một DevOps/System Administrator được yêu cầu khởi tạo Keycloak Cluster ở chế độ tối ưu cho môi trường Production, sử dụng **PostgreSQL** làm Relational Database Backend.

## 2. Chuẩn bị Môi trường (Prerequisites)
Để thực hiện bài lab, bạn cần chuẩn bị:
- Máy chủ (Server/Local Machine) có cài đặt **Docker** và **Docker Compose**.
- Đảm bảo các port `8080` (Keycloak) và `5432` (PostgreSQL) không bị tiến trình khác chiếm dụng.
- RAM trống ít nhất 2GB để chạy mượt mà hệ thống Java application.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Tạo thư mục làm việc và tệp `docker-compose.yml`**
Tạo một thư mục mới có tên `keycloak-postgres` và di chuyển vào đó:
```bash
mkdir keycloak-postgres
cd keycloak-postgres
```

Tạo tệp `docker-compose.yml` với nội dung sau:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: keycloak-db
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - keycloak-net

  keycloak:
    image: quay.io/keycloak/keycloak:22.0.0
    container_name: keycloak-server
    command: start-dev
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - keycloak-net

networks:
  keycloak-net:
    driver: bridge

volumes:
  postgres_data:
```

**Bước 2: Khởi chạy các dịch vụ**
Chạy lệnh sau để pull các images và khởi tạo containers:
```bash
docker-compose up -d
```

**Bước 3: Xem log khởi động**
Quan sát log của Keycloak để đảm bảo kết nối DB thành công và server khởi động không gặp lỗi:
```bash
docker logs -f keycloak-server
```
Bạn sẽ thấy thông báo tương tự `Keycloak 22.0.0 on Quarkus started in XXms`.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu:**
1. Mở trình duyệt và truy cập `http://localhost:8080`.
2. Nhấp vào **Administration Console**.
3. Đăng nhập bằng `Username: admin` và `Password: admin`.
4. Truy cập **Master Realm** -> **Realm settings** và đảm bảo giao diện quản trị phản hồi tốt.

**Troubleshooting (Khắc phục sự cố):**
- **Lỗi không kết nối được PostgreSQL (`Connection refused`):** Có thể do PostgreSQL mất thời gian khởi động, trong khi Keycloak lại kết nối ngay. *Cách xử lý:* Restart lại Keycloak container bằng `docker restart keycloak-server` hoặc thêm script wait-for-it.
- **Lỗi xung đột Port (`bind: address already in use`):** Do port `8080` bị chiếm dụng. *Cách xử lý:* Thay đổi port mapping thành `"8081:8080"` trong `docker-compose.yml` và restart lại.
- **Dữ liệu bị mất khi khởi động lại (Data loss):** Kiểm tra cấu hình `volumes` của service postgres xem có map chính xác đường dẫn chứa dữ liệu trong container (`/var/lib/postgresql/data`) chưa.
