# Lesson 14: Lõi Động cơ Siêu Âm (Quarkus Architecture)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết)
> **Goal:** Lịch sử đau thương và Cuộc cách mạng lớn nhất của Keycloak. Tại sao họ vứt bỏ Đứa Con Cưng WildFly để đập đi xây lại toàn bộ với Lõi Động Cơ Quarkus? Giải mã quyền năng Khởi động chớp nhoáng (Build-time optimization) khác biệt hoàn toàn với Spring Boot.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bóng Đen Của WildFly (Kỷ Nguyên Java EE Nặng Nề)
Keycloak Các Bản Cũ (Từ V17 Trở Lùi Khóa) Được Dựng Trên Server Khổng Lồ WildFly (JBoss).
- **Điểm yếu Chết Cứng:** Mỗi lần Bật Bấm Khởi Động Server. WildFly Lên Sàn Đọc Hàng Triệu Dòng File Cấu Hình XML (`standalone.xml`). Quét Tìm Khớp Phẳng Hàng Ngàn Thư Viện Lệnh Ốp.
- **Hậu Quả Sinh Tồn:** Bật Server Mất Gần 1 Phút Đến 2 Phút Chờ Khởi Động Tải Khối. Rút Cạn Gần 1 GigaByte RAM Chỉ Để Máy Chạy Nhích Hư Không Lên Dáng Đứng Trông Khách. Khi Gắn Bỏ Kubernetes (Khung Nền Sinh Lập Trọng Kéo Mạng Scale Mở Cháy Khởi Động Tắt Bật Pod). 2 Phút Chờ Start Server Của Nó Biến Việc Nở Băng Bớt Tải Quá Chậm Không Bắt Kịp Sóng Rớt Lệnh Login Do Đứng Cửa Treo Hoàn Toàn (Startup Lag Rác Hệ).

### 1.2. Hừng Đông Quarkus (Super-Sonic Sub-Atomic Java)
Cuộc Cách Mạng Khóa: RedHat Viết Ra Hệ Khung Quarkus Với Khẩu Quyết "Đem Mọi Tính Toán Khởi Động Vứt Sạch Vào Giai Đoạn Build".
1. **Lúc Nấu Cháo (Build Time):** Lệnh `kc.sh build` Đọc Sạch Cấu Hình Nén Lại, Quét Sạch Annotation Class Kéo Dấu Thừa Chặt Đi, Đóng Tất Cả Thành 1 Cục Vỏ Vô Cùng Cứng Và Trơn Tru (Immutable Bọc Mạch Nhỏ).
2. **Lúc Chạy Ra Sân (Run Time):** Lệnh `kc.sh start`. Máy Chủ KHÔNG HỀ TÍNH TOÁN HAY DỊCH GÌ NỮA. Nạp Thẳng Chút Mảng Cứng Vô Trí RAM Đáy Vụt Đèn Xanh Lên Chạy Luôn Trong 2-3 Giây Sóng Đỉnh Điểm. Ăn RAM Chút Xíu Bớt Cỡ Gấp Đôi So Khung WildFly Cháy Nặng Trục Đáy (Ném Rác Đằng Đáy).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Phá Thế Chia Phân Trọng 2 Lệnh Gắn Mảnh Đời Server (Tắt Wildfly Đứng Sửa Nóng Vĩnh Viễn Không Còn Nữa):

```mermaid
graph TD
    subgraph "Cách Quarkus Khóa Cửa Bê Tông Thời Gian Build"
        User[Admin DevOps Yêu Cầu: Đổi Database Sang MySQL / Nhét File SPI Lạ Vô Máy]
        
        Build[Chạy Lệnh Build Tạo Bê Tông Trống (kc.sh build --db=mysql)]
        Note over Build: Ở Khúc Này Trục Động Cơ Quarkus Ép Chết Cấu Hình Dòng Mã Hóa Bytecode Xóa Hết Thư Viện Lõi Postgres Ra Chừa Đúng Driver Lệnh Đệm MySQL Kéo Lõi Nén Vỏ Chạy Gọn.<br/>Chặt Sạch Sự Dynamic Động Mạch. Đúc Thành Cục Cứng Lọc Khung Tối Ưu (Closed World).
        
        Start[Bật Đèn Chạy Nhận Lệnh (kc.sh start --optimized)]
        Build -->|Chuyển Giao Trọng Vật Immutable Lên Kéo| Start
        
        Note over Start,User: Khác Biệt: Wildfly Cứ Chạy Ngay Xong Quăng File Database Đọc Mới Biết Mà Rẽ Nhanh Rất Trễ.<br/>Quarkus Giam Nhốt 3 Tiếng Phân Tính Ở Giai Đoạn Build Kín! Để Khúc Start Trở Nên Ánh Sáng Mượt Gấp Thập Phân Nền Thời Gian.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Đừng Trộn Lệnh Nếu Bạn Xài Docker Cấu Trúc Khối Scale Nhanh (Container Image Optimization)**
> **Tội Ác Viết Dockerfile:**
> `CMD ["/opt/keycloak/bin/kc.sh", "start", "--optimized=false"]` (Hoặc Lệnh Chạy start-dev Có Đội Lốt Build Tự Động Rác).
> Nghĩa Là Mỗi Lần Pod Của K8S Khởi Động Đánh Nền, Bạn Ép Nó Vừa Chạy Build Vừa Chạy Start Ngay Trong Runtime. Mất Toong Sứ Mệnh Tối Ưu Của Quarkus Đáy Mạch Máu Rớt Tốc Lên Tới 30 Giây.
> **Kiến Trúc Bọc Docker Điệu Trị (Multi-Stage Build):**
> Trong File Dockerfile, Phase 1 Chạy Lệnh `RUN /opt/keycloak/bin/kc.sh build --db=postgres` (Ép Build Kẹp Chặt Kẽ Ở Trạm Nấu Cháo Image Bất Diệt). Phase 2 Copy Chút Ruột Hút Trút Xuống Lõi Ra Start Bằng Đúng Lệnh `CMD ["kc.sh", "start", "--optimized"]`. K8S Bật Pod Lên Mất Đúng Vài Giây Nhanh Thấu Kính Băng Mạch Giao Khách Rơi Nhanh Vọt Nâng Ngược Trọng Kẽ Scale Ánh Sáng (Horizontal Scale Gấp Rút).

> [!CAUTION]
> **Bi Kịch Lệnh Sửa Cấu Hình Xong Không Lên Hình (Configuration Options Phân Phân Tầng Hóa Nhạy)**
> Trên Thế Hệ Quarkus Mới. Có Một Lệnh Cấu Hình Rơi Vào Phân Tầng Build (Ví Dụ Đổi Loại DB `--db`). Có Lệnh Cấu Hình Lại Rơi Vào Tầng Start (Ví Dụ Đổi Password Database Đỉnh Lệnh `--db-password`).
> Nếu Admin Không Trông Xem Lệnh Nằm Rìa Ở Tầng Nào. Bạn Gõ Lệnh Đổi Loại Database Vào Trục File Config Kéo Start Đè Tới Trọn. Keycloak Nằm Chết Văng Khóc Thét Báo Lỗi Hư `Build time option --db Cannot be specified at runtime`. (Nghĩa Là Mảng Sửa Khung Rễ Xe Ô Tô Bắt Buộc Đưa Về Giai Đoạn Nhà Máy Build Chứ Ra Đường Lăn Bánh Chạy Start Chỉ Sửa Được Kiểu Châm Thêm Đổ Xăng Mà Thôi).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Biến Thể Config Lựa Chỗ Đọc (File `keycloak.conf` vs `Environment Variables`):
Quarkus Thừa Hưởng Trọng Cầu Nguồn MicroProfile Config Dịch Tách Lệnh Chéo Khung Ngược Ưu Tiên:
1. Bạn Lưu Cấu Hình Vào File Cứng Nằm Nền Đĩa Đáy `/conf/keycloak.conf`.
2. Nhưng Trên Máy K8S, Lệnh Biến Môi Trường (Env Vars) Đè Phóng Đưa Vô Bằng Deployment Yaml Bắn Đè `KC_DB_URL=jdbc:mysql...` 
-> Quarkus Sẽ Lấy Cục Env Đè Tắt Dòng File Config Để Lấy Mệnh Lệnh Khớp Quyết Chót (Biến Môi Trường Ưu Tiên Cao Hơn). Điểm Tuyệt Đỉnh Giúp Bật Khung Hạ Tầng DevOps CI/CD Inject Giá Trị Bí Mật (Secret) Mà Không Rò Rỉ Để Lại Vết Trầy Vô File Nhạy Text Rác Cứng Giữ Mã Lệnh.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Trận Đấu Tắt Sóng Của JVM Khác Hệ (Trái Hệ Chạm GraalVM Native Image Ngược Phễu Lõi):**
  - Quarkus Sinh Ra Là Trọng Khí Ép Biên Dịch Lệnh Java Thành 1 Cục Binary Cứng Viết Bằng Lõi Hệ Điều Hành (.exe, Hoặc Cục .elf Không Chạy Qua Trình Cầm Tay Dịch Máy Ảo Java Nữa GraalVM Native). Nhờ Đó Nhanh Lên 100 Lần. Chết Khung RAM Dưới Trục.
  - Tuy Nhiên Keycloak Không Khuyên Hoặc Rất Gắn Bẫy Hỗ Trợ Đầy Đủ Chế Độ Native (Lệnh Rút Lõi Hỏng Toàn Khối Trái). Tại Sao?
  - Bởi Vì Nó Quá Rộng! Quá Nhiều Lỗ SPI Khớp Sợi Rút Cắm Động (Mà Nền Chạy Native Binary Bắt Đóng Mạch Phải Biết Sạch Sành Sanh Sợi Khai Báo Tại Lúc Compile). Nếu Đóng Cứng Lệnh Dịch Phân Mã Đi Binary Đáy, Bạn Chẳng Thể Thả Ném Đội Custom SPI Mới Cho Nó Nuốt Đè Chút Tươi (Hot Nhúng). Hiện Tại Vẫn Chọn Chạy Mảng JVM Quarkus Là Chủ Lực An Toàn Tuyệt Đại Điểm Vừa Nhanh Gấp 10 Vừa Mềm Khớp Chút Dẻo Rút Cắm SPI Ngoài Rìa (Một Khúc Giao Kết Bất Bại Hoàn Hảo Vấn Đề Lịch Sử Khắc).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Giai Đoạn Khởi Động Server Bằng Chế Độ `start-dev` Dùng Ném Cho Lập Trình Viên Ở Máy Bàn Cài Thử Nghiệm Nắm Nhanh Khác Biệt Trọng Yếu Thế Nào So Với Việc Phải Rã Bản `build` Và `start` Phức Tạp Lôi Hệ Của Production Kéo Băng Khó Khăn Đều Gọi Chết?**
- **Junior:** Nó giống nhau xài start-dev cho production đi cho nó khỏe chả sao. Mọi thứ tự build lo cho lẹ.
- **Senior:** Một Quả Phá Lỗi Hỏng Toàn Cục Tự Diệt Hệ Thống Chết Security Khác Khủng Khí!
`start-dev` Kích Bật Toàn Bộ Khung Mở Cửa Ác Liệt Của Developer Trái Trọng Điểm Doanh Nghiệp Bao Gồm:
- Bật Database Đáy RAM Mất Điện Sạch Bay (H2 Database Embedded Xóa Hết User). Mất Session Xóa Trắng Thông Tin.
- Tắt HTTPS Đóng SSL Cắm Nhận Mạng Http Phẳng Kéo Nối Trần Mọi Bức Mật Khẩu Bay Giữa Phố Cho Dân Đứng Bắn Súng Wireshark Hack Thấy Ngay.
- Cache Đóng Bắt Chặn Local Tránh Văng Mảng Máy Lưới Mạng Không Đụng Infinispan Clustering.
- Trả Khách Hàng Giai Đoạn Đợi Nạp Khung Nóng Chậm Để Ép Dịch Giao Cập Nhật (Gây Tốc Độ Xử Lý Trọng Ngậm CPU Lệnh Chết Xuống Tầng Cũ).
Do Đó, Để Tốc Độ Máy Chủ Rẽ Băng Khung Ánh Sáng Ở Môi Trường Thật, BẮT BUỘC Rẽ Trái Sang 2 Phase (Lọc Sạch Đáy Tắt Khung Ngậm Nháp).

**2. Làm Thế Nào Xử Lý Ca Phải Bật Tính Năng Tương Lai (Preview Features) Đã Bọc Mạch Mã Hóa Đáy Quarkus Nằm Trong Lõi Mã Nguồn Keycloak Đang Gắn Đè Dưới Đuôi Kẽ Phát Triển Ngầm Thăm Dò Nhanh? Nó Thuộc Về Khung Build Hay Start Trị Lệnh Nắm Giữ Ống Lõi?**
- **Junior:** Chắc Start lên bật lên coi thử nó chạy thế nào.
- **Senior:** Các Nút Gạt Features Mới Đóng Đinh Vào Code Móng Nền Cấu Trúc Khung Khóa Xử Lý Bề Mặt Tầng Bytecode (Ví Dụ Tính Năng Giao Token Loại Mới Hay MapStorage Bỏ Trục JPA Đáy Xưa).
Việc Này CẤM GỌI Lệnh Rẽ Khi Đã Đúc Khối Sáng Tại Dòng Runtime (Gõ Lệnh Đuôi `--features=...` Tại Start Báo Ngập Đỏ Đuôi Đứt). Bắt Buộc Đẩy Chạy Cờ Về Phase Khóa Đúc `build --features=...` Để Cỗ Máy Trục Động Cơ Nấu Cháo Quarkus Xoắn Bản Dịch Hàm Đút Ép Cắm Kết Nối Lệnh Phẳng Tầng Nhựa Nền Nằm Yên Chờ Phát (Chuẩn Nguyên Tắc Nấu Trước Nhai Bọc Giao Xong Start Lôi Đóng Chạy Rất Điển Hình Quarkus Thời Nay Dập Nền Giao Kiến Trúc Kép Mảnh Vỡ Giới Hạn Sống Trọng Khí).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Configuration:** Quarkus Build Time vs Run Time.
