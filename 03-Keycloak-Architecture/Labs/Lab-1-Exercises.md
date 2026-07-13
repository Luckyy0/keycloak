# Lab 1: Giải phẫu Lõi Database và Các Cấu trúc Cách ly

> [!NOTE]
> Bài Lab này sẽ giúp bạn Dựng một Cụm Keycloak hoàn chỉnh chạy trên PostgreSQL. Sau đó, chúng ta sẽ chui thẳng vào Lõi Database (Bằng công cụ DBeaver/pgAdmin) để soi xem Keycloak tổ chức Realms, Users, Roles như thế nào dưới dạng các Bảng RDBMS. Mắt thấy Tai nghe mới ngộ đạo.

## Chuẩn bị
- Đã cài đặt Docker và Docker Compose.
- Đã cài đặt một phần mềm quản trị Database (khuyến nghị DBeaver, TablePlus, hoặc pgAdmin).

## Bước 1: Chạy Cụm Keycloak + PostgreSQL

1. Mở Terminal, di chuyển vào thư mục `03-Keycloak-Architecture/code/`.
2. Quan sát file `docker-compose.yml`. Bạn sẽ thấy chúng ta cấu hình 2 dịch vụ (Keycloak và Postgres), kết nối bằng biến `KC_DB_URL`. Chú ý Keycloak đang chạy bằng lệnh `start-dev` cho tiện lab.
3. Khởi động bằng lệnh:
```bash
docker-compose up -d
```
4. Đợi khoảng 10 giây để Keycloak tự động khởi tạo Bảng dữ liệu thông qua Liquibase.

## Bước 2: Tương tác qua Giao diện để Tạo Mầm Dữ liệu

1. Truy cập: `http://localhost:8080/admin` (Admin: `admin` / Pass: `admin`).
2. **Tạo Realm Mới:** Bấm góc trái trên, tạo Realm tên là `Vingroup`.
3. **Tạo Client:** Trong Realm Vingroup, tạo 1 Client tên `Ketoan-App`. (Loại OpenID Connect).
4. **Tạo Role:** Trong Realm Vingroup, tạo Realm Role tên là `Giam_Doc`.
5. **Tạo User:** Tạo User tên `Alice`. Ở tab Credentials, đặt pass `123456` (Nhớ tắt nút Temporary).
6. Ở tab Role Mapping của Alice, gán Role `Giam_Doc` cho cô ấy.

## Bước 3: Mổ Xẻ Dưới Đáy PostgreSQL

1. Mở DBeaver. Kết nối vào PostgreSQL:
   - Host: `localhost`
   - Port: `5432`
   - Database: `keycloak`
   - User: `keycloak`
   - Password: `password`
2. Mở Schema `public` -> `Tables`. Bạn sẽ thấy hơn 100 bảng được Keycloak quản lý. (Không được tự ý INSERT/UPDATE trực tiếp vào đây kẻo hỏng Cache).

### Khám phá 1: Sự vĩ đại của Multi-tenancy (Bảng `REALM`)
- Chạy lệnh `SELECT * FROM REALM;`
- Bạn sẽ thấy 2 dòng: `master` và `Vingroup`. Chú ý cột `ID` là một mã UUID dài. Hãy copy mã UUID của `Vingroup` ra Notepad (Ví dụ: `1a2b3c...`). Cột ID này chính là Khóa Cách Ly cho mọi bảng khác.

### Khám phá 2: Bản thể User và Thuộc tính (Bảng `USER_ENTITY` và `USER_ATTRIBUTE`)
- Chạy lệnh `SELECT * FROM USER_ENTITY WHERE REALM_ID = 'mã-uuid-của-Vingroup';`
- Bạn sẽ thấy dòng chứa tên `Alice`. Hãy chú ý các cột chỉ có Email, Username. KHÔNG HỀ CÓ CỘT PASSWORD ở đây.
- Bây giờ bạn về lại Giao diện Admin, gắn 1 Attribute `chuc_vu` = `CEO` cho Alice.
- Chạy lệnh `SELECT * FROM USER_ATTRIBUTE;` để thấy Data được lưu dạng Key-Value (Dọc) trỏ về ID của Alice ra sao.

### Khám phá 3: Cỗ Máy Mã Hóa Mật Khẩu (Bảng `CREDENTIAL`)
- Chạy lệnh `SELECT * FROM CREDENTIAL;`
- Nhìn vào dòng của Alice. Cột `SECRET_DATA` chứa một chuỗi JSON băm bằng thuật toán `pbkdf2-sha256`. Rõ ràng Keycloak không lưu Pass trần (Plaintext) và tách hẳn nó ra một bảng riêng (Bảng Nhạy cảm) để chống Dump Full DB lộ pass.

### Khám phá 4: Mạng Lưới Nhện của Phiên (Bảng `USER_SESSION`)
- Đăng nhập thử bằng tài khoản Alice ở Account Console (http://localhost:8080/realms/Vingroup/account).
- Chạy lệnh `SELECT * FROM USER_SESSION;`
- KẾT QUẢ SẼ LÀ RỖNG!!! (0 dòng). 
- **Lý giải bài học lý thuyết:** Chẳng phải ta đã học Session được lưu ở RAM (Infinispan) sao? PostgreSQL không thèm lưu Session hiện tại để né Nút thắt Cổ chai Băng thông.
- **Để phá vỡ lý thuyết:** Đăng nhập lại, nhưng lần này cấu hình App phát sinh 1 cái **Offline Session** (Kéo dài vĩnh viễn). Chạy xem lệnh `SELECT * FROM OFFLINE_USER_SESSION;`. Giờ thì Session đã Đâm Thẳng Xuống Đĩa SSD Postgres Trú Ẩn an toàn qua mùa đông.

## Bước 4: Dọn Dẹp
Kết thúc quan sát, tắt cụm Docker:
```bash
docker-compose down -v
```

> [!TIP]
> **Bài học rút ra:** Không bao giờ gõ lệnh `ALTER TABLE` vào DB của Keycloak, vì Liquibase sẽ cắn nát hệ thống khởi động ở lần sau do Checksum không khớp. DB Keycloak sinh ra là để ĐỌC TRỘM, KHÔNG PHẢI ĐỂ GHI ĐÈ BẰNG TAY. Mọi tương tác phải qua Admin REST API.
