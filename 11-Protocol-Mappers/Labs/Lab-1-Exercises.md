# Lab 1: Máy Bơm Động & Javascript Injection

> [!NOTE]
> Lab này yêu cầu bạn trực tiếp nhào nặn cấu trúc Data của JWT Token. Bạn sẽ dựng Built-in Mapper và sau đó thử bật tính năng Javascript Engine (vốn bị khóa kín) để bơm Cột Tính Toán Động trực tiếp vào payload, mô phỏng nghiệp vụ cực khó của Khách Hàng.

## Chuẩn bị
- Máy có Docker và Docker-Compose.
- JWT Decoder (trang `jwt.io`).

## Bước 1: Ráp Khung Đáy Rễ Căn Cứ Và Bật Mã Nguồn Javascript

1. Đi vào thư mục `11-Protocol-Mappers/code`. 
2. Mở file `docker-compose.yml`. 
   > **Quan trọng:** File này đã được tôi cài sẵn biến `KC_FEATURES=scripts` để đánh thức Động cơ GraalVM JS vốn bị RedHat phong ấn từ bản 18+.

## Bước 2: Bật Cụm Khung Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch Cắt Rò Rụng

```bash
docker-compose up -d
```
Mở UI: `http://localhost:8080/admin` (admin/admin).
Tạo 1 Realm mới: `Vingroup_Mapper`.

## Bước 3: Đưa Đạn Dữ Liệu (Data Attribute) Vào DB

1. Vô Bảng `Users`. Bấm Add user.
   - Username: `khach_vip`. Bấm Save.
   - Tab Credentials: Đặt mật khẩu `pass`.
2. **Khai báo Cột NoSQL (Attribute):**
   - Vô Tab `Attributes` của user `khach_vip`.
   - Nhập Key: `ma_vach`
   - Nhập Value: `BARCODE-888999`
   - Bấm Save.

## Bước 4: Chế Tạo Ống Bơm Built-in Chọc DB

1. Vô Bảng `Clients`. Tạo 1 Client tên `app-mua-sam`. (Tắt Auth, Bật Direct access grants để xài `curl` cho lẹ).
2. Chuyển Qua Cột Menubar Bên Trái, Vô `Client scopes`.
3. Bấm **Create client scope**. Tên là `scope_thong_tin_ma_vach`. Loại Default. Save.
4. Bấm Vô Thằng Scope Mới Này -> Chuyển Qua Tab **Mappers**.
5. Bấm `Configure a new mapper` -> Chọn loại **User Attribute**.
6. Điền Các Thông Số Sống Còn:
   - Name: `bom_ma_vach`.
   - User Attribute (Nguồn): `ma_vach`.
   - Token Claim Name (Đích Json): `barcode_secret`.
   - Add to access token: `ON`.
   - Save Lại Bức Cắt Khung!

## Bước 5: Bắn Mã Javascript Vô Động Cơ Engine

1. Đứng nguyên ở tab Mappers của Scope `scope_thong_tin_ma_vach`.
2. Bấm tiếp `Configure a new mapper` -> Lần Này Chọn **Script Mapper**. (Nếu không có `KC_FEATURES=scripts` ở Docker, bạn sẽ không bao giờ thấy nút này!).
3. Điền Thông Số Bắn JS:
   - Name: `bom_js_tinh_toan`.
   - Script (Chép Nhờ Mã Này Vào):
     ```javascript
     var ma = user.getFirstAttribute("ma_vach");
     if (ma != null && ma.startsWith("BARCODE-")) {
         exports = true;
     } else {
         exports = false;
     }
     ```
   - Token Claim Name: `is_vip_barcode`.
   - Claim JSON Type: Chọn `boolean`. (Để in ra JSON không có dấu nháy kép `""`).
   - Add to access token: `ON`.
   - Bấm Save!

## Bước 6: Kích Hoạt API Trút Bão Mạng Sinh JWT

Bật Terminal, ép Keycloak nhả JWT bằng lệnh:

```bash
curl -X POST \
  http://localhost:8080/realms/Vingroup_Mapper/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password' \
  -d 'client_id=app-mua-sam' \
  -d 'username=khach_vip' \
  -d 'password=pass'
```

Copy chuỗi `access_token` cực dài, đem dán lên `jwt.io`. BÙM! Nhìn vào bụng JSON:
```json
{
  "barcode_secret": "BARCODE-888999",
  "is_vip_barcode": true,
  ...
}
```
Bạn Đã Hoàn Toàn Làm Chủ Tuyệt Kỹ Bơm Mạch Của Xưởng Đúc Dữ Liệu Keycloak!

## Bước 7: Dọn Lệnh Rác

```bash
docker-compose down -v
```
