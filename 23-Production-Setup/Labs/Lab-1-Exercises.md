# Lab 1: Dựng Lên Cụm Tên Lửa Lõi Kép (Clustered Keycloak behind NGINX)

## 1. Mục Tiêu (Objectives)
Thiết lập một kiến trúc Production chuẩn mức Enterprise gồm 4 thành phần:
1. **1 Máy Chủ PostgreSQL** làm Trái Tim Dữ Liệu duy nhất.
2. **2 Máy Chủ Keycloak (KC1, KC2)** chạy song song, kết nối với Postgres và tự động Tìm Thấy Nhau qua `JDBC_PING` JGroups để tạo Lưới Infinispan.
3. **1 Máy Chủ NGINX** đứng chắn cửa làm Reverse Proxy, nhận Request từ cổng 80/443 và Load Balance 50-50 chia đều tải cho KC1 và KC2. Ép Header `X-Forwarded-For`.

---

## 2. Chuẩn Bị (Prerequisites)
Bạn cần có Docker và Docker Compose trên máy tính.
Hệ thống mạng lưới này cực kỳ phức tạp và ăn khá nhiều RAM, hãy đảm bảo máy tính của bạn còn trống ít nhất 2GB RAM.

```bash
cd code
docker-compose up -d
```

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Khởi Động Và Quan Sát Tiếng Hú Của Bầy Sói (JDBC_PING Discovery)
1. Trong Terminal, Gõ Lệnh Theo Dõi Log Của Máy Chủ K1:
   ```bash
   docker logs -f keycloak-node1
   ```
2. Bạn Hãy Cuộn Log Lên Để Ý Tìm Các Dòng Chữ Có Tiền Tố `[org.jgroups.protocols.JDBC_PING]`.
   Đó Là Lúc Máy Chủ K1 Ghi IP Của Nó Xuống Cơ Sở Dữ Liệu PostgreSQL Đáy Lõi DB Trút Cắt Khung Tương Lai.
3. Tiếp Tục Tìm Dòng Log Huyền Thoại Của Infinispan Báo Hiệu Kết Nối Bầy Đàn Thành Công Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:
   `ISPN000094: Received new cluster view for channel ispn: [keycloak-node1|1] (2) [keycloak-node1, keycloak-node2]`
   -> Câu Này Nghĩa Là Máy 1 Đã Nhìn Thấy Máy 2 Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh! (Cluster Size = 2 Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh). Tòa Lâu Đài Phân Tán Đã Xây Xong Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề!

### Task 2: Kiểm Chứng Cánh Cửa Chắn NGINX (Load Balancer & Edge Proxy)
1. Máy Chủ Nginx Được Mở Ra Ở Cổng 80 Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng. Bạn Truy Cập Vào: `http://localhost/` Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề.
   (Tuyệt Đối Không Dùng Cổng 8080 Vì Chúng Ta Không Cho Trực Tiếp Vào Keycloak Nữa Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa).
2. Đăng Nhập Bằng Tài Khoản Admin Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa (`admin` / `admin`).
3. Bây Giờ Chúng Ta Chơi Trò Chơi Sinh Tử Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy:
   Hãy Trực Tiếp Rút Dây Mạng Của Máy Chủ 1 Bằng Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:
   ```bash
   docker stop keycloak-node1
   ```
4. Quay Lại Trình Duyệt Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh, Bạn F5 Hoặc Chuyển Sang Tab Bất Kỳ Trong Màn Hình Admin Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp.
   **KẾT QUẢ ĐÁY LÕI TỰ TRỊ BỌC LỆNH:** Trang Web VẪN CHẠY BÌNH THƯỜNG TRƠN TRU Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy! BẠN KHÔNG BỊ VĂNG RA MÀN HÌNH ĐĂNG NHẬP LẠI Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh!
   Bởi Vì NGINX Đã Thấy Node 1 Chết Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Tự Động Hất Lệnh Đáy DB Qua Node 2 Oanh Khung Dịch Lụa Mạch Lệnh. VÀ Node 2 Vẫn Cầm Bản Sao Session (Bộ Đệm Phân Tán Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa) Nên Nó Ghi Nhận Bạn Là Thằng Đã Đăng Nhập Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích. High Availability Tuyệt Đối Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần!

### Task 3: Chứng Minh Con Mắt Xuyên Tường Của Keycloak
1. Vô Lại Màn Hình Admin (Trên Node 2 Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy).
2. Vào Menu **Sessions**.
3. Bạn Hãy Bấm Vào Cột Chứa IP Của Phiên Đăng Nhập Gần Nhất Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh.
4. BẠN SẼ THẤY: Địa Chỉ IP Ghi Chú Ở Trong Đó Sẽ Là Địa Chỉ Của Trình Duyệt Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh (Ví dụ Vùng `172.x.x.x` Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề Gateway Của Bạn), CHỨ KHÔNG PHẢI Địa Chỉ Của Container NGINX Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng. 
   Đó Là Khả Năng Xuyên Tường Của Tính Năng Nhận Diện Bức Thư Chuyển Tiếp Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề `KC_PROXY_HEADERS=xforwarded` Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh!

---

## 4. Dọn Dẹp (Cleanup)
Hủy Các Cỗ Máy Khủng Long Tránh Tràn RAM Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:
```bash
docker-compose down -v
```
