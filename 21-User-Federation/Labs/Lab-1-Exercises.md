# Lab 1: Máy Bơm Hút Máu OpenLDAP

## 1. Mục Tiêu (Objectives)
Trong bài lab này, chúng ta sẽ Dựng Lên 2 Trạm:
- Trạm 1: `openldap` (Máy chủ chứa Rừng Dữ Liệu Nhân Viên Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt).
- Trạm 2: `keycloak` (Lãnh Chúa Đứng Ở Mây Đáy Lõi DB Trút Cắt Khung Tương Lai).
Mục tiêu là Cấu Hình Bơm Dữ Liệu Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh hút toàn bộ User từ Rừng OpenLDAP Về Đổ Vào Database PostgreSQL Của Keycloak. Bật Sync Chế Độ Read-Only.

---

## 2. Chuẩn Bị (Prerequisites)
Hệ thống Docker Compose Bài 21 Đã Chứa Sẵn Máy Chủ LDAP Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề.

```bash
cd code
docker-compose up -d
```
- Máy chủ `openldap` đang chạy Ở Mạng Kín Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ. Đã Nạp Sẵn 1 Khách Hàng Có Sẵn Dưới Đó:
  - Tên: `nguyen-van-b`
  - Pass: `admin`
- Truy cập Admin Console Keycloak: `http://localhost:8080` (admin/admin).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Thiết Lập Cáp Nối Mạng (User Federation)
1. Trong Admin Console Keycloak, Bấm Menu **User federation**.
2. Bấm Add Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa -> Chọn **LDAP**.
3. Khai Báo Các Thông Số Đỉnh Cao Của Máy Bơm Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:
   - **UI display name**: `my-company-ldap`
   - **Vendor**: Chọn `Other` Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng (Vì Mình Đang Chạy Hình Nộm OpenLDAP).
   - **Connection URL**: `ldap://openldap:389` (Chọc Qua Thằng Docker Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa).
   - Bấm Nút **Test connection** Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm. Báo Thành Công Xanh Lè Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa!
   
### Task 2: Cấp Quyền Đào Bới Rừng (Bind Credentials)
1. Cuộn Xuống Đáy Bọc Lệnh Cũ Đỉnh Chóp Mục **LDAP searching and updating** Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh:
   - **Users DN** (Tọa Độ Tìm Người): `dc=example,dc=org`
   - **Username LDAP attribute**: `cn`
   - **RDN LDAP attribute**: `cn`
   - **UUID LDAP attribute**: `entryUUID`
   - **User object classes**: `inetOrgPerson, organizationalPerson`
2. Sang Mục **Authentication settings** Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh:
   - **Bind type**: Chọn `simple`.
   - **Bind DN**: `cn=admin,dc=example,dc=org`
   - **Bind credentials**: `admin`
3. Bấm Nút **Test authentication** Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị. Báo Thành Công Xanh Lè Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt!
4. Cuộn Xuống Mục **Synchronization settings** Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề.
   - Bật Công Tắc **Import Users** LÊN! (Đồng Bộ Về DB Local Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh).
   - Chọn **Edit mode**: `READ_ONLY`.
   - Bấm SAVE Lại Toàn Bộ Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy.

### Task 3: Kích Hoạt Máy Bơm Động Cơ Khủng Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích
1. Ngay Sau Khi Bấm Save Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp. Bạn Sẽ Thấy Ở Đáy Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Hiện Lên 2 Nút Bấm Thép Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa: `Synchronize all users` và `Synchronize changed users`.
2. Bấm Nút **Synchronize all users**.
3. Bạn Sẽ Thấy Hiện Thông Báo Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa: *"Success! Sync of users finished successfully. 1 imported users, 0 updated users"*.
4. MÁY CHỦ ĐÃ KÉO CỤC MÁU TỪ LDAP LÊN TRỜI THÀNH CÔNG Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh!

### Task 4: Kiểm Chứng Thẩm Thấu Database Local Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm
1. Vô Menu **Users** Của Admin Console Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.
2. Bấm Chữ *View all users* Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng. BẠN SẼ THẤY CÓ THẰNG KHÁCH Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy **`nguyen-van-b`** HIỆN HÌNH Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh!
3. Mở Trình Duyệt Ẩn Danh Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Truy Cập Trang Account: `http://localhost:8080/realms/master/account/`.
4. Đăng Nhập Với Account Vừa Rút Máu Bọt Mạch Kéo API Dữ Lụa Lỗ Bọt Cắt Trắng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích: `nguyen-van-b` / `admin`.
5. Bùm! Keycloak Đập Pass "admin" Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy Xuống Bind Thử Dưới LDAP Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy. Báo Thành Công Đỉnh Đáy Oanh Mạng Bắt Lụa! Đẩy Khách Vào Account Console Mạch Cắt Oanh Trọng Lõi Tự Trị Ngay Lập Tức Oanh Khung Dịch Lụa Mạch Lệnh! Hoàn Mỹ Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa!

---

## 4. Dọn Dẹp (Cleanup)
Hủy Các Cỗ Máy Khủng Long Tránh Tràn RAM Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:
```bash
docker-compose down -v
```
