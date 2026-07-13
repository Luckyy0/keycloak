> [!NOTE]
> **Category:** Practical/Lab  
> **Goal:** Mô phỏng các sự cố phổ biến của Keycloak trên môi trường thực tế và áp dụng các kỹ thuật đọc log, chuẩn đoán để khôi phục hệ thống.

## 1. Kịch bản Thực hành (Lab Scenario)

Hệ thống Keycloak của công ty bạn đột nhiên nhận được hàng loạt báo cáo lỗi từ người dùng. Bạn được cấp quyền truy cập vào máy chủ (thông qua Docker). Các lỗi được báo cáo bao gồm:
1. Người dùng không thể đăng nhập, trang web liên tục báo lỗi "Invalid Client" hoặc "Invalid Redirect URI".
2. Hệ thống bất ngờ bị treo cứng hoàn toàn, không thể vào được giao diện Admin Console, nghi ngờ lỗi tràn bộ nhớ (OOM).

Nhiệm vụ của bạn là vào vai một System Administrator, sử dụng các công cụ dòng lệnh để tìm nguyên nhân và khắc phục.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy ảo hoặc máy local có cài đặt **Docker** và **Docker Compose**.
- Tải file docker-compose giả lập lỗi do giảng viên cung cấp (nếu có), hoặc tự tạo một file `docker-compose.yml` cố tình cấu hình sai:
  - Cấu hình `-Xmx` quá nhỏ (ví dụ 128m).
  - Không thiết lập biến `KC_PROXY`.

*Tạo nhanh file `docker-compose.yml` có lỗi:*
```yaml
version: '3.8'
services:
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev
    environment:
      - KC_DB=dev-file
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
      # Cố tình set bộ nhớ cực thấp để tạo lỗi OOM
      - JAVA_OPTS_APPEND=-Xms64m -Xmx128m 
    ports:
      - "8080:8080"
```

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Kích hoạt hệ thống lỗi

1. Mở Terminal, đi đến thư mục chứa file `docker-compose.yml`.
2. Chạy lệnh: `docker-compose up -d`.
3. Đợi vài phút để Keycloak khởi động (hoặc sụp đổ).

### Bước 2: Điều tra lỗi OutOfMemoryError (OOM)

1. Mở trình duyệt và truy cập `http://localhost:8080`. Bạn sẽ thấy trang tải vô tận hoặc báo `Connection Refused`.
2. Truy xuất Log của Container bằng lệnh:
   ```bash
   docker logs --tail 200 keycloak
   ```
3. **Tìm kiếm bằng chứng:** Hãy tìm trong log các đoạn có chữ `ERROR` hoặc `Exception`. Bạn sẽ nhanh chóng bắt gặp dòng `java.lang.OutOfMemoryError: Java heap space`.
4. **Khắc phục:** Mở file `docker-compose.yml`, chỉnh lại thông số `JAVA_OPTS_APPEND`:
   ```yaml
   - JAVA_OPTS_APPEND=-Xms512m -Xmx1024m
   ```
5. Chạy lại `docker-compose up -d` để update vùng nhớ. Sau vài phút, giao diện Admin sẽ lên bình thường.

### Bước 3: Điều tra lỗi "Invalid redirect uri"

1. Truy cập Admin Console, tạo một Realm `test-realm` và một Client `test-client`.
2. Cố tình để trống mục `Valid Redirect URIs`.
3. Mở tab mới, cố gắng giả lập một request đăng nhập bằng cách nhập trực tiếp URL (thay `<IP>` bằng localhost):
   `http://localhost:8080/realms/test-realm/protocol/openid-connect/auth?client_id=test-client&response_type=code&redirect_uri=http://app.local/callback`
4. Giao diện Keycloak sẽ hiển thị cảnh báo "Invalid parameter: redirect_uri".
5. **Điều tra:** Vào Keycloak Admin Console -> Realm Settings -> Bật tính năng **Events** (Save Events: ON).
6. F5 lại trang lỗi kia một lần nữa.
7. Vào mục **Events** trên Admin Console, bạn sẽ thấy một bản ghi lỗi có type `LOGIN_ERROR` và Error là `invalid_redirect_uri`.
8. **Khắc phục:** Vào cấu hình của Client `test-client`, thêm `http://app.local/callback` vào mục Valid Redirect URIs. Quay lại reload trang lỗi, giao diện Form Login sẽ hiển thị thành công.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
- **Tiêu chí 1:** Container chạy ổn định trên 15 phút không bị khởi động lại tự động do OOM. Sử dụng lệnh `docker stats` để xác nhận bộ nhớ đang ở mức ~600MB - 800MB.
- **Tiêu chí 2:** Tab giả lập request login hiển thị được form đăng nhập Username/Password, trên thanh địa chỉ có chứa `state` và `client_id` hợp lệ.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Nếu Docker không start được sau khi sửa:** Có thể cú pháp YAML bị lỗi (thụt lề sai khoảng trắng). Hãy kiểm tra file bằng các công cụ YAML validator.
- **Vẫn lỗi "Invalid redirect uri":** Hãy chắc chắn bạn không gõ dư dấu slash (`/`) ở cuối URL. Keycloak đối chiếu chuỗi URL theo dạng "Exact match" (khớp chính xác). Nếu config là `http://app.local/callback` mà gửi `http://app.local/callback/` thì vẫn là lỗi.
- **Không thấy log Event:** Hãy chắc chắn bạn bật lưu Event cho ĐÚNG Realm (trong trường hợp này là `test-realm`), việc bật Event ở `master` realm sẽ không thu thập được log của `test-realm`.
