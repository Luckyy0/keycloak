# Lesson 2: Triển khai bằng Khối Siêu Nhẹ (Docker Container)

> [!NOTE]
> **Category:** Theory & Practice
> **Goal:** Lột bỏ nỗi lo Cài cắm Phiên bản Java, Xung đột cổng mạng hay Cấu trúc thư mục rối rắm. Trải nghiệm Bí kíp đóng gói Keycloak thành một khối Container siêu nhẹ, nhét vào ổ đĩa và Bắn lên mọi môi trường từ máy Dev Cùi Bắp đến Server Đỉnh Cao không sai một lỗi nhỏ.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Chối Bỏ Môi Trường Máy Chủ Thực Tế Cũ
Sự Đau Đớn Của Cài Đặt ZIP (Bài 1): Bạn Cài Keycloak Chạy Mượt Trên Mac. Đem Mã Khung Ném Sang Kỹ Sư Window. Bùm! Window Nó Đọc Thiếu Path, Mất Lệnh Shell Vỡ Chữ, Java Thiếu Cập Nhật, Sập Từ Nút Khởi Động Start Không Hiện Khung Lên.
**Docker Image Giết Chết Đau Khổ:** Image Của Keycloak Là Một Cái Thùng Đóng Đinh Kín Kẽ Rỗng Rút Cắt Chặt (Được Team RedHat Cắt Gọn Từng Mega RAM Linux). Trong Bụng Nó ÔM SẴN Lõi CentOS Mỏng Dính + Java 17 Tuyệt Phẩm + Gói Code Đỉnh. Dù Máy Ngoài Có Là Mac/Win/CentOS/Ubuntu Chẳng Ảnh Hưởng Khỉ Gì Cả. Run Là Trực Tiếp Rực Cháy Lên Máy Server Hạt Gạo Hoàn Hảo (It works on my machine problem SOLVED).

### 1.2. Quyền Năng Tiêm Thuốc Kích Thích Bằng Biến Môi Trường (Env Variables)
Thay Vì Phải Vào Sửa Chỉnh File Text XML/Conf Loạn Cào Cào Ở File Cứng Cổ Đại. Docker Biến Cấu Hình Trở Lên Căng Đét Linh Động Bằng Giao Thức Ép Env Trúc Động Xuyên Thành Nhựa Thùng Container.
Mọi Thông Số Cấu Trúc Khung DB, Tài Khoản Bật Đều Lòi Xuyên Mạch Gọi Lệnh Đè Lên Dạng `KC_XXX` Bơm Ở Phía Ngoài Gọi Cấu Lệnh Ngược Rớt Vô Bụng Hoàn Hảo.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Khi Docker Daemon (Động Cơ Kéo Khối) Móc Kéo Chạy Ngược Khối Image Keycloak Xuất Thưởng Thành:

```mermaid
graph TD
    subgraph "Sự Ràng Buộc Thẳng Chọt Nút Độc Lập Port Và Lõi Ngầm"
        Host[Máy Chủ Laptop Vật Lý Của Bạn]
        
        DockEngine[Docker Engine Bẻ Khóa Luồng Gọi Cổng]
        
        Container((Vỏ Bọc Container Keycloak Nhựa Kính))
        
        Port_Nhựa[Cổng Ngầm Rỗng Bên Trong: 8080]
        Port_Sắt[Cổng Lõi Thép Bạn Nắm Ở Ngoài: 9090]
        
        Host -->|Bạn Gõ: http://localhost:9090| Port_Sắt
        Port_Sắt -->|Docker NAT Đảo Cầu Nhanh Ép Kẽ Rơi Chuyển Áp| Port_Nhựa
        Port_Nhựa --> Cỗ Máy Keycloak Kéo Rút HTTP Token Văng Rớt Trả Token
        
        Note over Host,Container: Bạn Hoàn Toàn Phủ Bóng Máy Mình Khỏi Đụng Cổng Gắt Của Thằng Nào Khác (Nếu Dùng Spring 8080 Cứ Để Yên Đó).<br/>Chỉ Cần Bẻ Rẽ Bảng Chặn Sắt Ngoài Bằng Lệnh Mapping ( -p 9090:8080 ) Nhẹ Như Lông Hồng.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Thảm Họa Rò Rỉ Đóng Quên Rác (Ephemeral Storage Mất Mát)**
> **Ác Mộng Đêm Container Cúp Điện:** Bạn Chạy Docker Bằng `start-dev`. Lấy H2 Ghi Database Cứng Vô Thư Mục Ảo Nằm Bên Trong Thành Vách Bụng Container (Chỗ Chứa Dữ Liệu Tạm Của Bọc Kính). Bạn Mày Mò 1 Tuần Add Trăm Kẻ Địch Và Rút Role Gắn Mệt Nghỉ Lòi Phân Khu Giao Quyền OIDC Ngon Đứt Tốc.
> Bạn Tắt Bật Máy Đóng Mạch Container Để Cập Nhật OS (Dùng Lệnh `docker rm` Gỡ Xác Thùng Rác). Sáng Mai Bạn Kéo Thùng Container Lại Khởi Chạy Lần Nữa. **TRẮNG TINH MÀN HÌNH CHẾT! MỌI DATA ĐÃ BỊ XÓA BÓNG BỌT BIỂN.**
> **Best Practice Sinh Tồn Nhất Ngành Docker:** Container Bản Chất Là (Ephemeral) Đồ Dùng 1 Lần Trôi Đồ Rỗng Đáy Chứa Nhựa Tạm. Bất Cứ Dữ Liệu Cứng Nào Phải Trọng Sinh Cứu Lâu Cần Khung Data Sống Dài Trăm Năm THÌ PHẢI ÁP LỆNH DÙNG **VOLUME MAPPING (`-v`)** Xuyên Đục Khét Tường Nhựa Gắn Móng Thép Lưu Lên SSD Mạch Máy Của Bạn! (Cụ Thể Khuyên Xài Database Rời Ra Tránh Đụng Ác Mộng Cắm Kẽ Này).

> [!CAUTION]
> **Trộm Bản Build (Fat Image Trái Mạch)**
> Đừng Đem Cái Image Gốc Chuẩn Mặc Định Lõi Nặng Red Hat (`quay.io/keycloak/keycloak:latest`) Lên Bắn Chạy Ngầm Ở Môi Trường Đỉnh Cao Nhất (Production). 
> Kéo Bản Lệnh Gốc Này Ép Nó Vừa Chạy Đợi Build Khoảng Thêm Chục Giây Tốn Cả Khối Băng Vị Tích Trái Ngầm Lắp Code Nóng Lẽ Ra Không Nên Có Ở Rìa Ngài Internet. Phải Tự Tạo Lệnh Dockerfile Nướng Lại Lọc Ép Nằm (Custom Multi-stage Build) Cắt Cụm Nén Trống Đi Lệnh Mảnh Siêu Trọng Cứng Bê Tông Xong Chỉ Ném Lệnh Rút Lõi Giữ Hình Đi Vận Hành (Thực Hiện Này Tại Tầng Triển Khai K8s Sẽ Học Rất Chặt Dưới Kia Mảng Bài Bọc).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lệnh Gọi Khóa Ảo Diệu Docker Môt Cú Nổ Cháy (Setup Cụm Container Nhanh Thấu Đáy):
```bash
# Một dòng lệnh Chọc Trái Lên Luôn Giao IDP Khủng Bằng Biến Khống Nút:
docker run -d \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  --name my_keycloak_container \
  quay.io/keycloak/keycloak:24.0.1 start-dev
```
Giải nghĩa Bảng Thép Nhanh:
- `-d`: Bọc Ngầm Chạy Sóng Nền Giấu Cắt Terminal (Không Bị Treo Lệnh).
- `-p 8080:8080`: Trổ Khe Đục Sắt (Máy Tính Mình: Khối Nhựa Bụng Của Thùng).
- `-e KEYCLOAK...`: Bơm Biến Môi Trường (Tiêm Env Tự Khởi Mã Nhận Admin Lõi).
- `--name`: Đóng Kẽ Đuôi Tên Thùng Tránh Tìm Lục Lại ID Dài Rác Không Thuộc Phân Vùng Lệnh Rỗng.
- `quay.io/...`: Mỏ Image Tuyệt Phẩm Của Cục RedHat Dọn Khống Code Kép Chặn Thay Vì Dùng Lõi DockerHub (Vị Trí Rác Thường Dễ Rò Khúc Mã Chứa Mã Độc Nhái Hàng Official).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Cái Lẫy Mất Biến Bí Mật Bọc Ngoài Hầm Đuôi Cụt Rỗng Mạch Máu Bash Không Bắt Nhận Data Thép Khác Thường (Tấn Công Chạy Quá Dài Lệnh):**
  - Nếu Công Ty Chống Viết Pass Ở Câu Lệnh CLI Vì Sợ Bị Xem Trộm Lịch Sử (Lệnh `history` Kéo Thấy Chữ `admin` Cười Văng Răng Đục Rò Tận Rễ Database). 
  - Đóng Ngăn Chặn Bằng Tệp Cứng Tuyệt Trị Tối Kỵ Chạm Mã (Env File Lạnh). Bạn Ép Sợi Chữ Pass Vào Tệp Kín Nơi Bóng Đêm Gắn Quyền Thép Chỉ Cho Đọc File `secrets.env`.
  - Lúc Nhận Nguồn Start Kéo Container, Gọi Bảng Tín Ngầm Cụt Nút: `docker run --env-file ./secrets.env ...`. Mọi Dữ Bí Mật Chạy Thẳng Trút Xuất Ngắn Không Có Thằng Khốn Nào Trượt Check Log Terminal Bắt Bớ Vệt Cháy Giao Quyền Được. Tịch Thu Trắng Hoàn Toàn Tầm Nhìn Kẻ Ngồi Cạnh Ngó Chữ (Blind Shell Bọc Khéo Tàng Hình Secret Rớt Container).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Câu Lệnh Mồi Cấu Chạy Start Của Image Cốt, Nếu Ta Không Ép Quyền Nhét Hai Dòng Biến Định Khởi Đầu `KEYCLOAK_ADMIN` Vào, Khi Keycloak Văng Kéo Giao Diện Lên Màn Hình Xong, Bạn Trắng Tay Bị Vứt Ở Lại Phía Cửa Kín Khách Vô Danh Không Lối Nào Vào Setup Bằng Quyền Admin. Trong Tình Cảnh Đó, Chui Cách Nào Xuyên Khe Thùng Hở Tường Kẻ Cắp Quyền Lấy Lại Quyền Quản Trị Hệ Thống Tối Cao Tạo Bảng Code Tạo User Mới Chốt Đóng DB Admin?**
- **Junior:** Giết cài lại chạy gõ thêm lệnh đó đè vào chắc là nhanh nhất.
- **Senior:** Chữa Cháy Mạng Sống Container Cứng Đáy Gãy Mà Không Cần Restart!
Bên Trong Vùng Mạch Ruột Của Máy Thùng Docker (Tầng Bin) Luôn Được Nhà Phát Triển Để Hờ 1 Cái Chìa Vạn Năng (Khóa Ngầm) Là Mã Tool Công Cụ Lệnh Quản Trị Trọng Điểm: Bảng Bash Thêm User Quản Trị Gốc Phản Chủ Lõi. 
Bạn Đóng Terminal Vào Thẳng Bụng Thùng Chạy Mở Shell Ngầm (Lệnh Chạy Xuyên Bọc Kính: `docker exec -it my_keycloak_container /bin/bash`). Vừa Vô Vùng Nhựa Kính Trong. Chạy Nén Chui Mã Gấp Ở Thư Mục Bin: `/opt/keycloak/bin/add-user-keycloak.sh -u super_boss -p password_manh`. Thao Tác Chết Khớp Gãy Chui Khung Gắn Nạp Tạo Mũi User Mới Ngầm Phá DB Rễ Tại Chỗ. Sau Đó Cút Khỏi Thùng Nhựa Bật F5 Màn Hình Trình Duyệt Quét Rút Nhập Tên Siêu Sếp Super_boss Nhẹ Xoay Nhấn Login Qua Bức Tường Tàn Chống Cửa Phế Sáng Ngời Lệnh Giao Môi Trường Rớt Bọc!

**2. Làm Thế Nào Trút Bỏ Được Data Bơm Ở Tầng Khác (Ví Dụ Thư Mục Gốc Ổ Đĩa D:\certs Máy Window Của Tôi Cất File Chứng Chỉ Gắn SSL) Sang Kéo Ghép Ngậm Xoắn Nhét Mạng Khớp Vào Cột Ống Ổ Rỗng Ở /opt/keycloak/conf Bên Mảng Nhựa Kính Linux Container Bụng Rút Cửa?**
- **Junior:** Copy thủ công xài docker cp thôi cho rồi nhét nó vô. 
- **Senior:** Copy Thủ Công Là Đồ Bỏ Khi Build Hạ Tầng Tự Động Hóa Xóa Máy Liên Tục.
Phép Nhồi Đảo Ngược Bọc Kết Ghép: Phép Bắn **Volume Bind Mount (`-v`)**.
Khái Niệm Khung Chạm Vách Tường Xuyên Luồng:
Lập Trình Gọi Sát Mạch: `-v /d/certs:/opt/keycloak/conf/certs`. Trục Kép Này Nhúng Móc Đánh Khung Tạo 1 Ống Bơm Nối Liền Thông Nhau Chảy Xuyên Đục Không Đứt Quãng Lệnh RAM Data Nền Tại Window Xé Thấu Lỗ Đục Linux Container. Bạn Quăng Cái Ảnh Tại Window D Nào Thằng Keycloak Thấy Có Liền Ở /opt Chỗ Đó Ảo Cục Bộ Hoàn Hảo Hóa Trực Kết Gắn Mọi Nẻo Đường Setup Cấu Hình Bọc Đuôi File Cứng Cực Kì Linh Động (Nền Tảng Tiêm File Đỉnh Để Phân Khúc Bơm Giấy Chứng Chỉ Rỗng Thép SSL HTTPS Mở Bảo Mật Lên Mây Mà Không Chạm Cháy Hình Gốc Image Build Lõi Cũ).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Docker Docs:** Running the Official Quay.io Container.
