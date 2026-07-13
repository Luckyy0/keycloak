# Lab 1: Nối Mạng Cò Mồi Keycloak - Keycloak (OIDC Federation)

## 1. Mục Tiêu (Objectives)
Trong bài lab này, chúng ta sẽ không nối với Google vì cần Đăng ký Developer Account phức tạp. Thay vào đó, chúng ta sẽ Dựng Lên 2 Trạm Keycloak Chạy Song Song Cùng Lúc:
- Trạm 1: `kc-idp` (Đóng vai trò như Google - Nơi chứa dữ liệu Khách Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt).
- Trạm 2: `kc-broker` (Đóng vai trò Cò Mồi Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp - Phục vụ App Kế Toán Lệnh Đáy DB).
Mục tiêu là Bật Nút "Login bằng KC-IDP" trên màn hình của KC-Broker Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị!

---

## 2. Chuẩn Bị (Prerequisites)
Hệ thống Docker Compose Bài 20 Đã Chứa Sẵn 2 Cỗ Máy Keycloak Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ.

```bash
cd code
docker-compose up -d
```
- Trạm 1 `kc-idp` Chạy Ở Cổng Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa: **`http://localhost:8081`** (admin/admin).
- Trạm 2 `kc-broker` Chạy Ở Cổng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh: **`http://localhost:8080`** (admin/admin).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Mở Cửa Trạm IdP (Khởi Tạo OAuth2 Client Trên `kc-idp`)
1. Truy Cập `kc-idp` tại `http://localhost:8081`. Đăng Nhập.
2. Vô Menu **Clients**. Bấm Create client.
3. **Client ID**: `broker-client` Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm.
4. Bật Công Tắc **Client authentication = ON** Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh. Save Lại.
5. Ở Tab Settings, Nhập Tạm **Valid redirect URIs**: `http://localhost:8080/*` (Tí Sửa Lại Chính Xác Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa). Save.
6. Sang Tab **Credentials**, Copy Khối **`Client Secret`**.
7. Tạo 1 User Mới Ở `kc-idp` Tên Là **`nguyen-van-teo`**, Cấp Password `123`.

### Task 2: Dựng Cáp Mạng Tại Trạm Cò Mồi (Identity Provider Trên `kc-broker`)
1. Truy Cập `kc-broker` tại `http://localhost:8080`. Đăng Nhập Cấu Trúc Khung Rỗng XML Nặng Nề Lệnh Khớp Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa.
2. Vô Menu **Identity Providers** Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp.
3. Bấm **Keycloak OpenID Connect** (Đấu Nối OIDC Chuẩn).
4. Khai Báo Dữ Liệu Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh:
   - **Alias**: `kc-idp` (Đây Sẽ Là Tên Cái Nút Hiện Lên Màn Hình Khúc Tới Ngay Lệnh).
   - Cuộn Xuống Đáy Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. Cấu Hình Endpoints Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ:
     - **Authorization URL**: `http://localhost:8081/realms/master/protocol/openid-connect/auth`
     - **Token URL**: `http://localhost:8081/realms/master/protocol/openid-connect/token`
   - Cấu Hình Client Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép:
     - **Client ID**: `broker-client`
     - **Client Secret**: Dán Cái Secret Bạn Vừa Copy Ở Task 1 Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa.
5. Kéo Lên Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Lấy **`Redirect URI`** Của Keycloak Broker (Ví Dụ Mạch: `http://localhost:8080/realms/master/broker/kc-idp/endpoint`). Copy Nó Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa! Bấm ADD Save Lại Cấu Cắt Khung Đứt Băng Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ.
6. (Quay Lại Máy `kc-idp` Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị, Paste Cái Dòng Vừa Copy Đè Lên Ô Valid Redirect URIs Cho Chuẩn Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần).

### Task 3: Chạy Thử Cửa Ải First Broker Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
1. Bạn Mở 1 Trình Duyệt Ẩn Danh (Incognito Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần).
2. Gọi URL Lấy Code Mồi Của Thằng Cò Mồi `kc-broker`: 
   `http://localhost:8080/realms/master/protocol/openid-connect/auth?client_id=account-console&response_type=code&redirect_uri=http://localhost:8080/realms/master/account/`
3. Màn Hình Đăng Nhập Bật Lên Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. BUM! BẠN SẼ THẤY CÓ THÊM 1 NÚT **`kc-idp`** Ở BÊN PHẢI MÀN HÌNH Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh!
4. Bấm Vào Nút Đó Cấu Trúc Khung Rỗng XML Nặng Nề Lệnh Khớp Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Trình Duyệt Văng Sang Màn Hình Của Máy Chủ Khác `localhost:8081`!
5. Nhập `nguyen-van-teo` Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh / `123`.
6. Đăng Nhập Thành Công Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy! Máy Tự Văng Ngược Lại `localhost:8080` Vô Trang Account Console Đỉnh Cao Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!
7. **Kiểm Chứng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh:** Tắt Tab Ẩn Danh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích. Vô Admin Console Của `kc-broker`. Mở Menu Users Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. BẠN SẼ THẤY CÓ 1 THẰNG `nguyen-van-teo` VỪA ĐƯỢC MÁY CHỦ TỰ ĐỘNG TẠO RA Oanh Khung Dịch Lụa Mạch Lệnh! Vẻ Đẹp Hoàn Mỹ Của Brokering Lệnh Đáy Oanh Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!

---

## 4. Dọn Dẹp (Cleanup)
Hủy 2 Cỗ Máy Docker Tránh Nặng RAM:
```bash
docker-compose down -v
```
