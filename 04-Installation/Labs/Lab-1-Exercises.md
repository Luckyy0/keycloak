# Lab 1: Dựng Cụm High Availability (HA) với Nginx và Keycloak

> [!NOTE]
> Bài Lab này sẽ biến lý thuyết thành hiện thực. Bạn sẽ tự tay đóng một Đấu Trường gồm 1 Khối Database (Postgres), 2 Động Cơ Keycloak (Chạy Khớp Cụm), và 1 Nhạc Trưởng (Nginx Load Balancer).
> Chúng ta sẽ kiểm chứng Tuyệt Kỹ: Chết 1 Máy, Session Không Rớt!

## Chuẩn bị
- Máy có Docker và Docker-Compose.

## Bước 1: Khởi động Đấu Trường Cụm

1. Mở Terminal, đi vào thư mục `04-Installation/code`.
2. Khám phá file `docker-compose.yml`. Ta có:
   - `postgres_db`: Nền tảng Đáy Database Trọng Tĩnh.
   - `kc_node_1` và `kc_node_2`: Cặp Song Sinh Đóng Role HA Khởi Start Nén Chạy Production (`KC_PROXY=edge`).
   - `nginx_proxy`: Đứng Giữa Mở Nút Keo Dính `ip_hash`.
3. Chạy lệnh Khởi động Cụm:
```bash
docker-compose up -d
```
4. Ngồi Đợi 1 Phút. Gõ `docker-compose logs -f kc_node_1` Và `kc_node_2`. Bạn Cần Phải Đợi Cho Tới Khi Cả 2 Thấy Dòng Chữ:
`ISPN000094: Received new cluster view... [kc_node_1, kc_node_2]`.
Đây là khoảnh khắc Linh Thiêng JGroups Đã Gắn Rễ 2 Khối Này Tìm Thấy Nhau Và Bắt Đầu Chia Sẻ Session RAM Rỗng! (Clustering Sóng Đỉnh Đã Thành Công Bọc Nét Sóng).

## Bước 2: Test Chức Năng Bù Trừ Nóng (Load Balancing)

1. Mở Trình Duyệt Bấm Cổng Nginx: `http://localhost:8080/admin`
2. Đăng Nhập `admin`/`admin`. (Bạn Chú Ý Nginx Tự Keo Dính Gọi Về Node Gắn Phẳng).
3. Tạo 1 Realm Mới `TestHA`. Tạo 1 User Tên `Hero`.
4. Mở 1 Tab Trình Duyệt Riêng Tư (Incognito). Đăng Nhập Bằng User `Hero` Ở Account Console `http://localhost:8080/realms/TestHA/account`. (Chú ý Bạn Vừa Sinh Ra 1 Cái Token/Session Sống Mỏng Lệnh Trọng Chóp RAM OIDC).

## Bước 3: Phép Thuật Gây Cháy Máy Chủ Vẫn Không Mất Khách Khung Session

1. Quay Về Tab Trình Duyệt Đầu Ở Trang Admin (Session Admin Đang Chạy Sóng Mạch). Xem Bảng Danh Sách Trống Session Hiện Của Thằng Khách Bọc `Hero`.
2. Kích Bật Máy Gõ Lệnh Bạo Cắt Phích Điện Đít Mạng Node 1 Kẽ Sống Khung Cắt:
```bash
docker stop kc_node_1
```
(Máy Chủ KC1 Bị Tắt 1 Cú Giết Nóng Rớt Oanh Liệt). Nginx Phát Hiện Và Ném Khách Qua Node Cứu Tinh KC2 Rớt Code Sóng Đỉnh Trí Nhanh Kẹp.
3. Qua Cái Tab Trình Duyệt Của Kẻ `Hero`. Bấm Reload (F5).
**KẾT QUẢ VĨ ĐẠI:** Thằng Khách `Hero` Bấm Sang Đáy Trang Profile Web Nhẹ Băng Trôi Khung OIDC KHÔNG HỀ BỊ VĂNG RA NGOÀI ĐÒI NHẬP PASSWORD LẠI ĐUÔI RỖNG CHỮ!
Mạch Mã OIDC Lệnh Cookie Ném Cú Nginx Tới Bụng KC2 Rút Mạch Máu. Bụng Thằng Não KC2 Lôi Ở Ram Nó Chữ Ký Infinispan Đã Ghi Đồng Bộ Ngầm Chéo Từ Hồi KC1 Còn Sống Giữ Trọng Giao OIDC Đánh Chặn Khách Hoàn Toàn Tươi Nóng.

## Bước 4: Dọn Dẹp Chiến Trường
Bạn Vừa Chạm Tay Khung Vận Hành Kiến Trúc Scale Doanh Nghiệp Không Rớt Nghẽn. Tắt Toàn Cụm Rỗng Lệnh:
```bash
docker-compose down -v
```

> [!TIP]
> Bất Kỳ Lúc Nào Thấy Mạng Keycloak Rớt 1 Đứa Mới Cập, Vào Xem Cấu Hình Lỗi UDP Hay Lệnh Báo DNS_PING. Khung Infinispan Chạy Sống Hay Chết Là Tội Rất Nặng Làm Phá Trận Kẹp HA Khách Thỉnh Văng Out Thẳng Đứt Cụm Chập Tải. Tụ OIDC Doanh Nghiệp Cốt Dựa Tại Phép Cluster Sống Chết Đuôi Mạch Kép Jgroups Infinispan Thép Giao Đáy Khung Rút Nhất Lõi!
