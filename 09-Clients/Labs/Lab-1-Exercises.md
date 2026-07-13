# Lab 1: Vương Quốc Chư Hầu (Clients & Hợp Đồng OIDC)

> [!NOTE]
> Bài Lab này đưa bạn vào vai Quản Trị Hệ Thống OIDC. Bạn sẽ khởi tạo các Ứng Dụng Chư Hầu (Clients), phân quyền chúng thành Public (Thằng Hề) và Confidential (Tướng Quân). Sau đó thử sức gọi API Xin Token bằng Cơ chế Service Account không cần dùng form Web.

## Chuẩn bị
- Máy có Docker và Docker-Compose.
- Có cài đặt Postman hoặc dùng lệnh `curl` trên Terminal.

## Bước 1: Ráp Khung Áo Giáp Lõi Tĩnh OIDC Database

1. Đi vào thư mục `09-Clients/code`. 
2. Mở file `docker-compose.yml`. Postgres và Keycloak 24+. 

## Bước 2: Bật Cụm Động Cơ OIDC Kéo Nhựa Giao Mạng

1. Khởi động OIDC bằng lệnh Thép Tĩnh Nền:
```bash
docker-compose up -d
```
2. Đăng Nhập Chỉnh Sửa Tại Admin Console: `http://localhost:8080/admin` (admin/admin).
3. Tạo 1 Lãnh Thổ Realm Mới: `Vingroup_Clients`.

## Bước 3: Đăng Ký Public Client (App Di Động / React)

1. Vô Bảng `Clients`. Nhấn `Create client`. 
2. Nhập Client ID: `app-react`. Bấm Next.
3. Ở Capability Config:
   - **Client authentication**: `OFF` (Đây là Public Client).
   - **Standard flow**: `ON`.
   - Bấm Save.
4. Kéo Xuống Access Settings Của Client Vừa Tạo:
   - **Valid redirect URIs**: Điền `http://localhost:3000/*`
   - **Web origins**: Điền `*` (Chỉ dùng học tập để bypass CORS). Bấm Save.

## Bước 4: Đăng Ký Confidential Client (App Java Backend)

1. Vô Bảng `Clients`. Nhấn `Create client`. 
2. Nhập Client ID: `app-java-backend`. Bấm Next.
3. Ở Capability Config:
   - **Client authentication**: `ON` (Confidential Client bắt buộc bật cái này).
   - **Service accounts roles**: `ON` (Để nó tự gọi API xin Token).
   - Bấm Save.
4. Chuyển sang Tab `Credentials`:
   - Copy cục mã chuỗi ở dòng `Client Secret` ra Notepad (Ví dụ: `1a2b3c...`).

## Bước 5: Gọi Token Bằng Cơ Chế Machine-To-Machine (Service Account)

Bây giờ bạn đóng vai thằng App Java. Bạn cần xin Token để móc Data báo cáo mà không cần gọi Khách Hàng lên bấm Đăng Nhập.

1. Bật Terminal Lên, Chạy Lệnh `curl` Hoặc Gõ Vào Postman:
```bash
curl -X POST \
  http://localhost:8080/realms/Vingroup_Clients/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' \
  -d 'client_id=app-java-backend' \
  -d 'client_secret=<Dán Cục Secret Của Bạn Vào Đây>'
```

2. BÙM! Keycloak sẽ văng thẳng 1 Cục JSON Trả Về. Bên Trong Chứa `access_token` Xanh Mượt Mạch Kẽ Rỗng Nhựa Mệnh Cắt Lệch Mạch OIDC Cũ Mệnh Ngắn Gọn.
Không Có Form HTML. Không Có Trình Duyệt Nào Cần Chạy Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng! Lõi Mạch OIDC Giao Khung API Hoạt Động Cực Kỳ Gọn Gàng Đáy Database UUID Không Gãy Chỗ.

## Bước 6: Phân Tích Sự Thất Bại Của Kẻ Hở (Public Error)

Bây giờ bạn lấy Thằng App React (Public) Đi Gọi Cơ Chế Service Account.

1. Chạy Lệnh Rỗng Tuếch Khung Lệnh Đuôi Ác Xé Form Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách:
```bash
curl -X POST \
  http://localhost:8080/realms/Vingroup_Clients/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' \
  -d 'client_id=app-react'
```

2. Lõi OIDC Sẽ Bắn Lỗi HTTP Lệnh Đỏ:
`{"error":"unauthorized_client","error_description":"Client not authorized for client credentials grant"}`. 
Tại Sao Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung? Vì Lõi Đáy Public Client Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Cấp K8s Oanh KHÔNG CÓ BÍ MẬT Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!. Cỗ Máy Keycloak Chỉ Bơm `Client Credentials` Cho Thằng Nào Đưa Được Tấm Thẻ `Secret` Ra Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là Resource_Access Oanh Khách Nhanh Sóng!

## Bước 7: Dọn Lệnh Rác Sóng Lưới Mạng OIDC Khép Kín Cấu Cắt
```bash
docker-compose down -v
```

> [!TIP]
> Ranh Giới Giữa Các Dòng App (Public vs Confidential) Cắt Lệnh Rỗng Phun Sinh Data Là Điểm Yếu Chết Người Của Backend Dev. Việc Nắm Rõ Và Trút Bão Mạng Sạch Bot Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Cấu Hình Đúng Flow Sẽ Cứu Hệ Thống Của Bạn Khỏi Hàng Loạt Hacker Bắn Token Trực Diện Lưới Mạng Cửa Đít Của TomCat/Quarkus Oanh Khách Nhanh Sóng Lỗ Trống Mạng!
