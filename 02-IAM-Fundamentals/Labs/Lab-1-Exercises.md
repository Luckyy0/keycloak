# Lab 1: Ánh xạ Lý thuyết IAM vào Thực tế Máy chủ Keycloak

> [!NOTE]
> **Category:** Labs (Thực hành)
> **Goal:** Học Lái Xe Trên Máy Tập Bắn. Khởi động Máy Chủ Keycloak và dùng chuột "Sờ tận tay" các Khái Niệm Tối Cao của Chương 2: Identity, Môi Giới Federation, MFA Cây Quyết Định, và Hạt Giống của Least Privilege.

## 1. Chuẩn bị Môi trường và Khởi Động Động Cơ

Sử dụng nguyên file `docker-compose.yml` có sẵn trong thư mục `02-IAM-Fundamentals/code/` để dựng lại Máy Chủ Giống Chương 1.
```bash
cd 02-IAM-Fundamentals/code/
docker compose up -d
```
Truy cập `http://localhost:8080/`, Đăng nhập Admin Console (Username `admin` / Pass `admin`).

---

## 2. Truy Tìm Các Khái Niệm Lõi Trên Bảng Điều Khiển

Vào Giao diện Admin, hãy Tự tay truy tìm và Thực Hiện các hành động sau:

### Tọa Độ 1: Identity vs Account (Bản Thể Khác Trái Mật Khẩu)
1. Ở Menu trái, Chọn **Users** -> Bấm Nút **Add user**. Tạo một Identity: Username `alice`, Email `alice@cyber.com`. Bấm Save.
2. Lúc này ALICE ĐÃ TỒN TẠI TRONG VŨ TRỤ (Identity Sinh Ra). Nhưng cô ấy KHÔNG THỂ ĐĂNG NHẬP (Chưa Có Account/Credential).
3. Sang Táp **Credentials** của cô Alice. Bấm **Set Password**. Nhập Pass `123456`, TẮT CỜ `Temporary` đi. Bấm Save.
*(Lý thuyết Chứng Minh: Mật Khẩu Chỉ Là Lớp Áo Khoác Đắp Lên Bản Thể Bất Tử Ở Dưới).*

### Tọa Độ 2: MFA - Xác Thực Đa Yếu Tố
1. Vẫn ở tài khoản của cô Alice. Chuyển sang Táp **Details**.
2. Tìm hộp thoại **Required User Actions** (Các Hành Động Ép Buộc Khách Phải Làm).
3. Chọn tính năng **Configure OTP**. Bấm Save.
4. Mở Tab Trình duyệt ẩn danh. Vào `http://localhost:8080/realms/master/account/`.
5. Đăng Nhập Bằng Nick `alice / 123456`.
6. BÙM! Màn hình KHÔNG VÀO TRONG, Bị Khóa Đứng Bằng 1 Mã QR Bắt Quét Google Authenticator. (MFA Cưỡng Chế 100% Hoàn Tất Dài Chưa Tới 1 Phút).

### Tọa Độ 3: Bàn Đàm Phán Ngoại Giao (Federation / Identity Provider)
1. Ở Menu Trái, Tắt tab User. Mở tab **Identity Providers**.
2. Bấm Nút **Add provider**. Bạn sẽ thấy sự Hiện Diện Vĩ Đại Của OIDC, SAML, Google, GitHub, Facebook.
3. Nếu bạn click vào GitHub, Keycloak chỉ đòi 2 Thông tin Sinh tử: `Client ID` và `Client Secret`. 
4. Nếu điền xong. Ra lại trang Login của Master Realm. Nút "ĐĂNG NHẬP BẰNG GITHUB" Tự Động Mọc Lên Một Cách Thần Thánh. (IdP Brokering Hoạt động hoàn hảo).

### Tọa Độ 4: Tách Bạch Rạch Ròi AuthN và AuthZ (Clients và Roles)
1. Menu Trái, Chọn **Clients** (Đây chính là Tọa Độ Của Service Provider - SP / App Khách Gõ Cửa).
2. Khi Bấm Tạo Client, bạn đang Phân Vùng Quyền Lực Chứ Không Phải Xác Thực.
3. Chọn Menu **Realm Roles**. Tạo Role `SUPER_VIP`.
4. Gán Role `SUPER_VIP` cho cô Alice. (Authorization Phân Quyền). JWT Của Alice từ Giờ Mọc Thêm Đôi Cánh SUPER_VIP Bên Trong Bụng.

---

## 3. Tắt Máy Dọn Dẹp Chiến Trường

Thực hành xong, trở lại Terminal và Hủy Diệt Vũ Trụ Ảo Để Tiết Kiệm RAM Máy Tính:
```bash
docker compose down -v
```

> [!NOTE] 
> **HOÀN THÀNH CHƯƠNG 02 (THE HARDEST PART):**
> Chúc Mừng Bạn Đã Tốt Nghiệp Phần Lõi Tư Duy Nặng Nhất. Kể từ giờ phút này, Bạn không còn là Một Cậu Thợ Gõ Lệnh. Bạn đã sỡ hữu Hệ Ngôn Từ và Triết Lý Thượng Thừa Của Một Kiến Trúc Sư An Ninh (Security Architect).
> Ở Chương sau (Chương 03), Chúng ta Sẽ Bổ Đôi Con Quái Vật Keycloak, Chui Vào Ruột Nó, Xem Từng Bánh Răng Nội Tại Chạy Bằng Cái Gì. Chuẩn Bị Tinh Thần Đi!
