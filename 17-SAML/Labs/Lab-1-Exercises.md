# Lab 1: Tích Hợp SAML Bằng Metadata Lõi Bọc Thép

## 1. Mục Tiêu (Objectives)
Thực hành đóng vai Admin Của Tập Đoàn kết nối 1 Hệ Thống Legacy (Code Bằng tool SAML Online) Vào Keycloak Oanh Cáp Trọng Lõi.
- **Task 1:** Đọc và Xuất File IdP Metadata.
- **Task 2:** Dùng File IdP Nạp Cho Trang Web SP Online Sinh Lệnh Oanh Rút.
- **Task 3:** Lấy Tọa Độ SP Nhập Ngược Lại Cho Keycloak Chữ Khớp Lệnh Oanh Rỗng.
- **Task 4:** Chạy Luồng Đăng Nhập SAML Bắn XML Chữ Ký Đáy Lụa.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Mở Trình Duyệt Truy Cập Admin Console Keycloak tại `http://localhost:8080/` (admin/admin).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Tải Bản Đồ Tọa Độ Của Lãnh Chúa (IdP Metadata)
1. Trong Admin Console Keycloak, Bấm Vào **Realm Settings**.
2. Cuộn Sang Tab Cùng **Endpoints** Ở Cuối Mạch Lệnh Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống.
3. Kích Vào Cửa Sổ **`SAML 2.0 Identity Provider Metadata`**.
4. Trình Duyệt Bật Ra 1 File XML. Click Chuột Phải -> **Save As...** Thành File `idp-metadata.xml` Lên Desktop.

### Task 2: Chế Tạo Tàu Sân Bay SP Online Oanh Tĩnh Lụa Thép
Chúng Ta Dùng Trang Web Hỗ Trợ SAML Test Đỉnh Chóp Trọng Khóa Tĩnh: `https://sptest.iamshowcase.com/`
1. Truy Cập Vào `https://sptest.iamshowcase.com/`. 
2. Ở Bảng Bên Trái, Bạn Sẽ Thấy Cái Cục **SP Metadata** Của Nó Đáy Lụa. Hãy Bấm Nút **Download** Tải Cái Cục Của Kẻ Địch Về Máy Tính `sp-metadata.xml`.
3. Bấm Lên Tab **Instructions** Của Trang Web Đó. Nó Cho Bạn 1 Chỗ Để Nạp File IdP Của Bạn. Dán Nội Dung Tờ Khai `idp-metadata.xml` Vào Khung Đáy Oanh Mạch Rút Trọng. (Đây Gọi Là Khớp Giao Ước Bằng Tay).

### Task 3: Import Bản Đồ Của Kẻ Địch Vào Keycloak Bọt Lụa
1. Quay Lại Keycloak Console. Vào Menu **Clients**.
2. Bấm Nút Trút Lụa Bọt Kẽ Mã Đáy **Import client**.
3. Chọn Tải Lên File `sp-metadata.xml` Của Thằng Trang Web Test Nãy Tải Về!
4. Keycloak Xử Lý Bọt Khung Oanh Cáp Cực Kỳ Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng. Nó Tự Tạo Ra 1 Thằng Client Tên Là `https://sptest.iamshowcase.com/testsp`. Tọa Độ ACS Tự Điền Chuẩn Mực Băng Tần Khung Kẽ Bọt Cắt Mạch! Save Lại Khớp Lệnh Oanh Rỗng.

### Task 4: Khai Hỏa Tên Lửa XML Lệnh Đáy Oanh Mạng Bọc Thép
1. Qua Lại Mạch Oanh Giao Dịch Của Trang `https://sptest.iamshowcase.com/`.
2. Bấm Nút Đáy Bọc Lệnh Cũ Trút Cáp Mạch Máu Cắt: **Log In**.
3. Bùm! Nó Bắn Lệnh Văng Ngược Về Localhost Keycloak Trút Lụa Mã Oanh Cáp Trọng Lõi Tự Trị. Nhập `admin`/`admin`.
4. Bùm Lần 2! Keycloak Sinh Assertion Bơm Chữ Ký XML-DSIG Văng Khách Ngược Lại Trang Web Test. Trang Web Test Vỡ Tung Cảm Xúc Oanh Dữ Lụa, Mở Khóa Đáy Bọc Cấp Phiên Lệnh Oanh Rút Thành Công! Kéo Kẽ Bọt Cắt Lệnh Giao Thức!

---

## 4. Dọn Dẹp (Cleanup)
Sau khi hoàn thành SAML Lab, hủy Docker tránh lãng phí RAM:
```bash
docker-compose down -v
```
