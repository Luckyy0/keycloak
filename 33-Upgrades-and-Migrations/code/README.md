# Hướng Dẫn Chạy Môi Trường Nâng Cấp Tự Động (Migration Lab)

Thư mục này chứa cấu hình Docker Compose để thực hành kỹ năng Đỉnh Cao của Dân Opperation (Vận Hành): Đổi Đời Phần Mềm (Upgrade/Migrate).

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File thiết lập chứa Keycloak Phiên Bản 23 (Cũ)
└── README.md              # File hướng dẫn này
```

## 2. Các Bước Thực Hành Đại Phẫu Thuật

### Giai Đoạn 1: Sống Trong Quá Khứ
1. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
2. Mở trình duyệt vào `http://localhost:8080`. Đăng nhập bằng `admin` / `admin`.
3. Tạo ra một vài dữ liệu giả (Tạo 1 Realm tên là `OIDC-Old`, tạo 1 User...). Đây là Dữ liệu Gốc của bạn.

### Giai Đoạn 2: Trạng Thái Gây Mê (Chuẩn Bị Lên Bàn Mổ)
1. Dừng con Máy Chủ Cũ lại (KHÔNG xóa Database nhé, chỉ tắt Keycloak thôi): 
   Gõ lệnh: `docker-compose stop keycloak`
2. Mở file `docker-compose.yml` bằng VSCode hoặc Text Editor.
3. Tìm dòng này:
   `image: quay.io/keycloak/keycloak:23.0.0`
   Sửa chữ `23.0.0` thành `24.0.1` (Hoặc phiên bản mới nhất hiện tại). Lưu file lại.

### Giai Đoạn 3: Tiến Hành Nâng Cấp (Liquibase Hoạt Động)
1. Bật máy chủ Mới lên (Với Image V24) bằng lệnh:
   `docker-compose up -d keycloak`
2. **CỰC KỲ QUAN TRỌNG:** Phải xem Log ngay để thấy phép màu:
   `docker logs -f code-keycloak-1`
   Bạn sẽ thấy những dòng chữ tuyệt đẹp hiện ra:
   `Updating database. This may take a while.`
   `Database upgrade is complete.`
   Chính Con Robot Liquibase đã tự động lặn xuống Postgres, bẻ cong cấu trúc Bảng, cập nhật toàn bộ Hệ Thống Dữ Liệu Lên Đời V24 cho bạn!

### Giai Đoạn 4: Trở Lại Với Tương Lai
1. Bấm F5 lại trang Web `http://localhost:8080`.
2. Đăng nhập lại. Bạn sẽ thấy cái `OIDC-Old` Realm vẫn nguyên vẹn, các User cũ vẫn sống sờ sờ.
Nhưng hệ thống lõi đã được Thay Máu thành công!

> **CẢNH BÁO BỎNG TAY:** Bạn CHỈ CÓ THỂ ĐI TỚI, KHÔNG THỂ LÙI.
> Nếu bạn cố tình sửa Image về lại `23.0.0` và bật lên, nó sẽ nổ tung báo lỗi Database đã bị chọc ngoáy bởi bản Mới! (Xem lý thuyết trong Bài 2 để biết thêm).
