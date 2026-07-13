> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thành thạo cách khởi chạy độc lập Keycloak cùng với External Database (PostgreSQL) sử dụng Docker, qua đó kiểm chứng mô hình dữ liệu bên trong cấu trúc lưu trữ của Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)

Keycloak mặc định (khi chạy ở chế độ dev) sẽ sử dụng cơ sở dữ liệu in-memory H2. Tuy nhiên, trong môi trường sản xuất (Production), H2 không được phép sử dụng. Thay vào đó, chúng ta phải sử dụng một RDBMS chuyên dụng như PostgreSQL hoặc MySQL.
Trong bài lab này, bạn sẽ thiết lập môi trường hoàn chỉnh bao gồm một container PostgreSQL và một container Keycloak được kết nối nối tiếp thông qua mạng Docker, sau đó tiến hành phân tích các bảng (tables) quan trọng sinh ra bên trong Database.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Công cụ **Docker** và **Docker Compose** đã cài đặt.
- Công cụ **DBeaver** (hoặc pgAdmin) để kết nối và kiểm tra Database từ máy Host.
- Terminal / PowerShell.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Viết tệp Docker Compose
Tạo một thư mục mới có tên `keycloak-postgres-lab`. Trong thư mục này, tạo tệp `docker-compose.yml` với nội dung sau:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: kc_postgres
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - kc_network

  keycloak:
    image: quay.io/keycloak/keycloak:latest
    container_name: kc_server
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    command: start-dev
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - kc_network

networks:
  kc_network:
    driver: bridge

volumes:
  pgdata:
```

### Bước 3.2: Khởi động hệ thống
1. Mở terminal, đi vào thư mục vừa tạo.
2. Chạy lệnh: `docker-compose up -d`.
3. Đợi vài phút để Keycloak khởi động. Kiểm tra logs để đảm bảo Keycloak đã nhận được Database:
   `docker logs -f kc_server`
   Khi bạn thấy dòng `Listening on: http://0.0.0.0:8080`, tức là dịch vụ đã chạy.

### Bước 3.3: Tạo dữ liệu mẫu trên UI
1. Mở trình duyệt, truy cập `http://localhost:8080/admin`.
2. Đăng nhập bằng `admin` / `admin`.
3. Tạo một Realm mới tên là `company-realm`.
4. Đi vào **Users** của `company-realm`, tạo một user tên là `johndoe`, điền First Name và Last Name.

### Bước 3.4: Kiểm tra cấu trúc CSDL bằng DBeaver
1. Mở **DBeaver**. Tạo kết nối mới (PostgreSQL).
2. Thông số kết nối: 
   - Host: `localhost`, Port: `5432`
   - Database: `keycloak`
   - Username: `keycloak`, Password: `password`
3. Sau khi kết nối thành công, mở danh sách Tables.
4. Tìm và Mở bảng `REALM`. Truy vấn (`SELECT * FROM REALM;`), bạn sẽ thấy 2 dòng là `master` và `company-realm`.
5. Tìm bảng `USER_ENTITY`. Truy vấn, bạn sẽ thấy thông tin của tài khoản `admin` và tài khoản `johndoe` bạn vừa tạo (kèm theo password hash, không lưu plain-text).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

- **Xác nhận sự bền vững của dữ liệu:** Để chứng minh Postgres đang hoạt động thay cho H2, bạn hãy tắt Keycloak (`docker stop kc_server`) rồi xóa cả container đi (`docker rm kc_server`). Sau đó chạy lại `docker-compose up -d`. Truy cập giao diện, nếu `company-realm` vẫn tồn tại, chúc mừng, dữ liệu đã được Persistent thành công.
- **Troubleshooting - Lỗi Timeout kết nối:** Đôi khi Keycloak khởi động quá nhanh và cố kết nối đến DB trước khi Postgres kịp chạy xong (mặc dù đã có `depends_on`). Nếu `kc_server` báo lỗi kết nối từ chối (Connection Refused), bạn chỉ cần khởi động lại tiến trình Keycloak bằng lệnh: `docker restart kc_server`.
- **Lỗi Port Conflict:** Nếu hệ thống báo Port 5432 đã được sử dụng, có thể máy ảo của bạn đã chạy sẵn Postgres local. Bạn cần đổi Port trong tệp docker-compose thành `5433:5432` (Ánh xạ từ 5433 trên host vào 5432 trong container).
