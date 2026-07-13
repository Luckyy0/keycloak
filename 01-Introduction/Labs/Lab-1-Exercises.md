# Lab 1: Khởi chạy Máy chủ Keycloak Đầu tiên (First Boot)

> [!NOTE]
> **Category:** Labs (Thực hành)
> **Goal:** Áp dụng Lý thuyết Chương 1. Chấm dứt việc nói chay. Tự tay thiết lập (Provision) Cỗ máy Keycloak bằng Docker Compose, gắn kèm Database PostgreSQL chuẩn Enterprise và bước những bước đầu tiên vào Lâu đài Quản trị.

## 1. Chuẩn bị Môi trường (Prerequisites)

Để thực thi Lab này, trên máy tính cá nhân (Laptop/PC) của bạn bắt buộc phải cài đặt:
1. **Docker Engine** (Chạy Nền tảng Ảo hóa Container).
2. **Docker Compose** (Công cụ quản lý cụm Container).
3. Mở sẵn Port `8080` (Dành cho Web) và `5432` (Dành cho Database) trên máy.

## 2. Phân tích File Docker Compose (Infrastructure as Code)

Hãy mở file `code/docker-compose.yml`. Đây là bản thiết kế Hạ tầng của chúng ta.
Là một Kỹ sư, đừng Copy chạy mù quáng, hãy phân tích 3 Mảnh ghép Sinh tử:

```yaml
version: '3.8'

services:
  # KHỐI 1: TẦNG LƯU TRỮ VĨNH CỬU (DATABASE)
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data # Gắn ổ cứng Volume để chống mất Data khi tắt máy
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password

  # KHỐI 2: TẦNG LÕI THỰC THI (KEYCLOAK QUARKUS)
  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1 # Ghim version chuẩn, cấm xài :latest
    command: start-dev # Chạy chế độ Dev (Bỏ qua cấu hình SSL/HTTPS phức tạp)
    environment:
      # Nối mạch máu từ Keycloak sang Database
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
      
      # Tự động tạo Account Chúa tể (Super Admin) lúc khởi động lần đầu
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8080:8080" # Đục tường lửa mở Port 8080 ra Laptop của bạn
    depends_on:
      - postgres # Ra lệnh Docker: Đợi DB chạy lên xong mới được phép gọi Keycloak

volumes:
  postgres_data:
```

## 3. Các bước Thực thi (Execution)

**Bước 1: Chạy lệnh khởi động**
Mở Terminal/Powershell, đi chuyển vào thư mục `01-Introduction/code/`.
Chạy lệnh:
```bash
docker compose up -d
```
*(Cờ `-d` là detached mode, giúp nó chạy ngầm trong background để bạn không bị treo Terminal).*

**Bước 2: Soi Log xem Cỗ Máy Hoạt Động (Quan trọng)**
Kiến trúc sư không bao giờ chạy lệnh xong là ngồi bấm Web luôn. Bắt buộc phải nhìn Log xem Cỗ máy Boot như thế nào:
```bash
docker compose logs -f keycloak
```
Bạn sẽ thấy Keycloak tự động thực hiện tiến trình **Migration (Tạo bảng CSDL)**. Nó chạy hàng trăm lệnh SQL để tạo Bảng User, Role vào trong PostgreSQL. Cuối cùng bạn sẽ thấy dòng chữ Vàng:
`Keycloak 24.0.1 on JVM (powered by Quarkus ...) started in 8.5s`

**Bước 3: Bước qua Cửa Khẩu**
- Mở Trình duyệt (Chrome/Firefox).
- Truy cập vào: `http://localhost:8080/`
- Bạn sẽ thấy Giao diện Welcome to Keycloak. Click vào **"Administration Console"**.
- Đăng nhập bằng Account siêu đặc quyền (Super Admin) mà chúng ta đã truyền ở Khối 2: Username: `admin` / Password: `admin`.

**CHÚC MỪNG BẠN ĐÃ CHIẾM ĐƯỢC QUYỀN ĐIỀU KHIỂN CỖ MÁY IDENTITY MẠNH NHẤT HÀNH TINH!**

---

## 4. Bắt lỗi Hệ thống (Troubleshooting Challenges)

Trong môi trường thực tế, mọi thứ hiếm khi chạy mượt mà ngay lần đầu.

**Thử thách 1: Chết non vì đụng Port**
- **Triệu chứng:** Khi gõ `docker compose up -d`, báo lỗi chữ đỏ: `Bind for 0.0.0.0:8080 failed: port is already allocated`.
- **Nguyên nhân:** Laptop của bạn đang chạy một máy chủ Tomcat, VueJS hoặc Jenkins khác Đang Chiếm Cổng 8080.
- **Cách Fix (Sửa Code):** Mở `docker-compose.yml`, tìm dòng `ports: - "8080:8080"`. Sửa con số BÊN TRÁI thành `8081` (Ví dụ: `8081:8080`). Con số bên trái là Port Laptop, bên phải là Port Lõi Container (Tuyệt đối không sửa số bên phải). Sau đó gõ truy cập `http://localhost:8081`.

**Thử thách 2: Keycloak văng Lỗi KếT NỐI (Connection Refused)**
- **Triệu chứng:** Xem log thấy báo: `org.postgresql.util.PSQLException: Connection to postgres:5432 refused`. Keycloak tự động Crash chết sấp mặt.
- **Nguyên nhân:** Lệnh `depends_on` trong Docker Compose chỉ bắt Keycloak "chờ cái Container DB BẬT LÊN" chứ nó KHÔNG ĐỢI DB KHỞI ĐỘNG XONG (DB Postgres chạy lần đầu phải tạo phân vùng mất 10 giây). Thế là Keycloak nhào dô cắn DB khi DB chưa sẵn sàng.
- **Cách Fix chuẩn Enterprise:** Sửa lệnh `depends_on` thành dạng Check Health nghiêm ngặt:
  ```yaml
  depends_on:
    postgres:
      condition: service_healthy
  ```
  *(Buộc phải viết thêm block `healthcheck` cho DB). Đây là lý do trong Production, người ta thường Setup DB riêng lẻ trước, chạy ổn định cả ngày, rồi mới bơm Máy chủ App vào sau.*

---

> [!NOTE] 
> **HOÀN THÀNH CHƯƠNG 01:**
> Bạn đã làm chủ hoàn toàn Định nghĩa về IAM, Kiến trúc Phân tầng, Sự khác biệt của các Bản Build (Editions) và có trong tay một Máy chủ Identity thực thụ đang chạy.
> Ở Chương sau (Chương 02), chúng ta sẽ Khoanh tay Cất Code đi, và học các Khái Niệm Trừu Tượng Tối Cao Của IAM (Identity, Federation, Zero Trust) - Những Ngôn Ngữ Bắt Buộc trong Bàn Cờ Đàm Phán.
