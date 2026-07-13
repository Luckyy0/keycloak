# Lab 1: Thực Hành Nâng Cấp Sinh Tử (Migration Lab)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Giả lập trường hợp bạn đang vận hành một Keycloak bản Cũ (v23.0.0). Bạn sẽ tạo dữ liệu, sau đó thực hiện quá trình Dừng Máy, Backup, đổi Image sang Bản Mới (v24.0.1) và theo dõi log để thấy quá trình Database được nâng cấp tự động bởi Liquibase.

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Khởi động Quá Khứ (Bản 23.0.0)
Tạo file `code/docker-compose.yml` (Đã được chuẩn bị sẵn ở thư mục `code/`). Chú ý dòng Image đang trỏ tới bản 23:
```yaml
  keycloak:
    image: quay.io/keycloak/keycloak:23.0.0
```
Khởi động hệ thống:
```bash
docker-compose up -d
```
Đăng nhập vào `http://localhost:8080/` bằng `admin`/`admin`. 
Tạo một Realm tên là `V23-Realm`.
Tạo một User tên là `old-user`. 
(Đây chính là dữ liệu vàng của bạn).

### Bước 2: Chuẩn Bị Backup (Đừng bao giờ quên bước này)
Trái Tim DB Đang Chạy, Bạn phải Rút Dữ Liệu Ra Cất Vào Két Sắt!
Mở terminal, chạy lệnh DUMP trực tiếp vào container Postgres để lấy file backup:
```bash
docker exec -t code-db-1 pg_dump -U keycloak -d keycloak_db -F c -f /tmp/backup.dump
docker cp code-db-1:/tmp/backup.dump ./backup.dump
```
Bây giờ ở máy bạn đã có file `backup.dump`. 

### Bước 3: Đưa Hệ Thống Vào Trạng Thái Gây Mê (Tắt Máy)
Dừng Keycloak để chuẩn bị Phẫu Thuật (Không được tắt Postgres nhé):
```bash
docker-compose stop keycloak
```

### Bước 4: Đánh Thức Tương Lai (Sửa Code Sang Bản 24.0.1)
Mở file `code/docker-compose.yml`. Sửa dòng Image của Keycloak:
```yaml
  keycloak:
    # Đổi chữ 23.0.0 thành 24.0.1
    image: quay.io/keycloak/keycloak:24.0.1 
```
Bật lại Keycloak Mới lên:
```bash
docker-compose up -d keycloak
```

### Bước 5: Xem Liquibase Mổ Xẻ DB
Ngay sau khi bật lên, HÃY NHANH TAY XEM LOG CỦA NÓ:
```bash
docker logs -f code-keycloak-1
```
Bạn sẽ thấy màn hình chạy chữ Vàng khét lẹt:
```text
Updating database. This may take a while.
[org.keycloak.connections.jpa.updater.liquibase.LiquibaseJpaUpdaterProvider] (main) Initializing database schema. Using changelog META-INF/jpa-changelog-master.xml
```
Và sau vài giây:
```text
[org.keycloak.connections.jpa.updater.liquibase.LiquibaseJpaUpdaterProvider] (main) Database upgrade is complete.
```
Liquibase đã làm xong việc của nó!

### Bước 6: Kiểm tra Sự Sống
Truy cập lại `http://localhost:8080/`. Đăng nhập `admin`/`admin`.
Mở danh sách Realm. Bạn sẽ Thấy cái `V23-Realm` vẫn Nằm Chình Ình Ở Đó Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp! User `old-user` vẫn sống nhăn răng!
Bạn đã Thực Hiện Cuộc Đại Phẫu Thuật Thay Máu Nâng Đời Keycloak Thành Công Mỹ Mãn! Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp!
