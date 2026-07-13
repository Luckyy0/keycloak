> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Hướng dẫn thực hành nâng cấp hệ thống Keycloak từ một phiên bản cũ lên phiên bản mới an toàn, thực hiện sao lưu (backup) cơ sở dữ liệu và xử lý các vấn đề phát sinh trong quá trình di chuyển dữ liệu (migration).

## 1. Kịch bản Thực hành (Lab Scenario)

Trong bài lab này, chúng ta sẽ mô phỏng một môi trường doanh nghiệp đang sử dụng Keycloak ở phiên bản cũ (ví dụ: `21.1.2`) với cơ sở dữ liệu PostgreSQL. Yêu cầu đặt ra là phải nâng cấp hệ thống lên một phiên bản mới hơn (ví dụ: `24.0.2`). 

Quá trình nâng cấp (Upgrade) trong Keycloak không chỉ đơn thuần là thay đổi phiên bản phần mềm (binary/container image) mà còn bao gồm quá trình di chuyển dữ liệu (Database Migration). Nếu không thực hiện đúng quy trình, cơ sở dữ liệu có thể bị hỏng và gây ra gián đoạn hệ thống. Bài lab sẽ trang bị cho bạn kỹ năng sao lưu, thử nghiệm và cập nhật an toàn.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Linux (Ubuntu/CentOS) hoặc WSL trên Windows.
- **Docker** và **Docker Compose** đã được cài đặt.
- Cấu trúc thư mục Lab như sau:
```text
keycloak-upgrade-lab/
├── docker-compose.yml
└── data/
```

- Khởi tạo hệ thống hiện tại (phiên bản cũ) bằng file `docker-compose.yml`:

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
      - ./data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  keycloak:
    image: quay.io/keycloak/keycloak:21.1.2
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
```

Khởi chạy hệ thống cũ bằng lệnh: `docker-compose up -d`. Đợi hệ thống chạy và tạo một vài người dùng ảo trong Admin Console để có dữ liệu thực hiện nâng cấp.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Đọc Release Notes và Migration Guide
Trước bất kỳ quy trình nâng cấp nào, bạn BẮT BUỘC phải đọc tài liệu [Keycloak Upgrading Guide](https://www.keycloak.org/docs/latest/upgrading/). Keycloak thường có những thay đổi đột phá (Breaking Changes) giữa các phiên bản lớn (Major versions). Hãy ghi chú lại nếu phiên bản mới yêu cầu cấu hình mới hoặc thay thế (Deprecated) tính năng cũ.

### Bước 2: Sao lưu (Backup) Database
Tuyệt đối không nâng cấp mà không sao lưu cơ sở dữ liệu.
Sử dụng công cụ `pg_dump` để sao lưu dữ liệu từ container PostgreSQL:

```bash
docker exec -t keycloak-upgrade-lab-postgres-1 pg_dump -U keycloak -F c keycloak > keycloak_backup.dump
```
> [!IMPORTANT]
> File `keycloak_backup.dump` chính là phao cứu sinh của bạn. Trong môi trường Production, bạn cũng nên sao lưu cả thư mục cấu hình và các custom Provider (file `.jar`).

### Bước 3: Dừng hệ thống hiện tại (Downtime)
Để tránh dữ liệu mới được ghi vào trong quá trình nâng cấp, hãy dừng container Keycloak:

```bash
docker-compose stop keycloak
```

### Bước 4: Cập nhật cấu hình và Khởi chạy phiên bản mới
Sửa file `docker-compose.yml`, thay đổi thẻ (tag) của image Keycloak sang phiên bản mới:

```diff
-   image: quay.io/keycloak/keycloak:21.1.2
+   image: quay.io/keycloak/keycloak:24.0.2
```

Lưu ý: Đối với môi trường Production, Keycloak không khuyến khích tự động migration khi khởi động để tránh rủi ro. Bạn nên thiết lập tham số môi trường `--spi-connections-jpa-default-migration-strategy=manual` để sinh ra file SQL script và review bằng tay, hoặc giữ mặc định `update` trong môi trường phát triển (mặc định của `start-dev`).

Khởi chạy lại hệ thống:
```bash
docker-compose up -d keycloak
```

Kiểm tra log theo thời gian thực để quan sát quá trình Database Migration:
```bash
docker logs -f keycloak-upgrade-lab-keycloak-1
```

Bạn sẽ thấy các dòng log liên quan đến Liquibase thực thi cập nhật các bảng:
`Updating database...`
`Successfully acquired change log lock...`
`Reading from database...`

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Nghiệm thu hệ thống
1. Truy cập vào giao diện quản trị `http://localhost:8080`.
2. Đăng nhập bằng tài khoản `admin`.
3. Kiểm tra thông tin phiên bản ở góc dưới bên phải hoặc trong phần **Server Info**.
4. Kiểm tra xem các Realm, Client, và Users tạo từ phiên bản cũ có còn tồn tại và hoạt động ổn định hay không.
5. Thử đăng nhập bằng một tài khoản người dùng bình thường để kiểm tra flow xác thực (Authentication Flow) xem có bị phá vỡ (break) không.

### Khắc phục sự cố (Troubleshooting)

- **Lỗi Migration Failed (Liquibase error):**
  Nếu quá trình nâng cấp thất bại và container bị sập, cơ sở dữ liệu có thể đang ở trạng thái không nhất quán (inconsistent state). 
  **Cách xử lý:** 
  1. Dừng và xóa các container.
  2. Khôi phục lại Database từ file backup bằng lệnh `pg_restore`:
     ```bash
     docker exec -i keycloak-upgrade-lab-postgres-1 pg_restore -U keycloak -d keycloak < keycloak_backup.dump
     ```
  3. Quay lại sử dụng phiên bản cũ hoặc phân tích log lỗi để tìm nguyên nhân gốc (ví dụ: xung đột custom SPI).

- **Thiếu thuộc tính hoặc cảnh báo Deprecated:**
  Sau khi cập nhật, hãy theo dõi cẩn thận System Logs. Nếu thấy các dòng log `WARN` về việc một tham số môi trường hoặc cấu hình bị loại bỏ (deprecated), bạn cần lên kế hoạch chỉnh sửa cấu hình hệ thống sớm để chuẩn bị cho phiên bản Major tiếp theo.
