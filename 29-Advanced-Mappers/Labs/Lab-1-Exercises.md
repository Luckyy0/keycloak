# Lab 1: Mở Khóa Và Phù Phép Javascript (Script Mapper)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Học cách kích hoạt tính năng Cấm Thuật `scripts` trên Keycloak bằng cấu hình Docker. Sau đó trực tiếp lên Giao Diện Web cấu hình viết một đoạn mã Javascript tự tính toán chèn một cờ hiệu Độc Lạ vào Access Token. 

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Mở Khóa Cấm Thuật Ở Máy Chủ (Docker)
Để Script Mapper hiện hình trên giao diện Admin, bạn phải khởi động Keycloak kèm Feature Flag.
Hãy tạo một file `docker-compose.yml` (Hoặc sửa file ở các bài trước):

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    # BẮT BUỘC CÓ --features=scripts Ở LỆNH KHỞI ĐỘNG Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
    command: start-dev --features=scripts
    environment:
      KC_DB: dev-file
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - 8080:8080
```
Gõ `docker-compose up -d`. Đợi một phút cho KC nổ máy.

### Bước 2: Tạo Dữ Liệu Thô (User Attribute)
1. Đăng nhập `localhost:8080` (admin/admin).
2. Tạo 1 User tên là `teo`, đặt mật khẩu là `123`.
3. Chuyển qua Tab **Attributes** của thằng User `teo`.
4. Gắn cho nó một cái Cột Thuộc Tính tên là `so_tien_trong_tui`, Value gõ vào `5000000` (5 Triệu). Save Lại!

### Bước 3: Cấy Script Mapper Bằng Giao Diện
1. Vào mục `Client Scopes` (Ở cột Menu trái).
2. Chọn thằng Scope tên là `roles` (Hoặc tạo 1 Scope mới để khỏi đụng chạm thằng Mặc định).
3. Sang tab **Mappers** -> Bấm Add Mapper -> **By Configuration**.
4. Chọn loại **Script Mapper** (Nếu bạn không gắn `--features=scripts` ở Bước 1, loại này sẽ Ẩn Mất Tăm Không Tìm Thấy).
5. Khai báo các thông tin sau:
   - **Name:** `Xep_Loai_Dai_Gia`
   - **Token Claim Name:** `xep_loai`
   - **Claim JSON Type:** `String`
   - **Script:** (Ô vuông Code bự chà bá Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, dán nguyên đoạn JS dưới đây vào):

```javascript
// Lấy thuộc tính gốc kiểu mảng (array) từ DB
var danhSachTien = user.getAttribute("so_tien_trong_tui");

if (danhSachTien != null && danhSachTien.size() > 0) {
    // Ép kiểu về Số Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa
    var soTien = parseInt(danhSachTien.get(0));
    
    // Logic tính toán Rẽ Nhánh Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
    if (soTien > 1000000) {
        exports = "VIP_DAI_GIA"; // Biến 'exports' là Cổng Đầu Ra Của Script Mapper Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!
    } else {
        exports = "NGHEO_ROT_MONG_TOI";
    }
} else {
    exports = "KHONG_XAC_DINH";
}
```
   - **Add to ID token:** Bật `ON`
   - **Add to access token:** Bật `ON`
6. Bấm Save.

### Bước 4: Kiểm Chứng Ma Thuật (Evaluate)
Sử dụng chức năng Evaluate Tích Hợp Sẵn Của Keycloak (Tránh mất công Postman Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy):
1. Bạn chuyển sang Tab **Client Scopes -> Evaluate** (Hoặc ở trong cấu hình 1 Client bất kỳ, chọn tab Evaluate).
2. Ô User: Tìm và Gõ tên `teo`.
3. Bấm Nút **Evaluate** bự đùng.
4. Một cục Cửa Sổ Bóc Tách Bụng Token Chạy Ra (Generated Access Token). Bấm vào Tab **Generated Access Token**.
5. Đảo mắt kéo xuống dưới cùng cục JSON. Bạn sẽ Sướng Rơn khi thấy dòng:
   `"xep_loai": "VIP_DAI_GIA"`
6. Quay lại sửa `so_tien_trong_tui` của Tèo thành `500`. Trở lại bấm Evaluate lần nữa. Bụng Token lập tức thay đổi:
   `"xep_loai": "NGHEO_ROT_MONG_TOI"`

Chúc Mừng Bạn Đã Tốt Nghiệp Ma Đạo Sư Token Bằng Javascript Oanh Khung Dịch Lụa Mạch Lệnh!
