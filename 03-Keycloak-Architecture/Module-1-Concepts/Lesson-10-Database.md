# Lesson 10: Đáy Hầm Ngầm Dữ Liệu (Relational Database)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Tìm hiểu Cấu trúc Bê tông Cốt thép của Keycloak. Hệ CSDL Quan hệ KHÔNG PHẢI là chỗ để đọc liên tục. Nó là cái Kho Khóa Trái Cửa để đảm bảo tính Toàn Vẹn ACID và sự sống còn của Mật mã khi Server sập nguồn.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Lập Trình JPA/Hibernate Đè Lên SQL Nguyên Thủy
Trong Kiến Trúc Của Keycloak, Không một Dòng Code Nào Trực Tiếp Viết Câu Lệnh Lồng `SELECT * FROM USER`.
Toàn bộ Lõi Tầng 3 Được Bọc Kính Kín Bằng Khung **JPA (Java Persistence API) Và Thư Viện Hibernate ORM**.
- **Ưu Điểm Cốt Tử:** Lớp Áo Này Giúp Keycloak Trở Thành Một Kẻ Đa Hệ Miễn Nhiễm Với Ổ Cứng (Database Agnostic). Khách Muốn Xài MySQL Trả Kém Băng? Được. Khách Giàu Có Muốn Cắm Oracle Enterpise? Tự Chạy Luôn. Khách Xài Trái PostgreSQL Chuẩn Cứng Mở? Nạp Dư Sức. Bản Thân Lõi Keycloak Chỉ Giao Tiếp Bằng Class Object (Ví dụ `UserEntity`), Trình Dịch Hibernate Sẽ Tự Bẻ Cổ Thành Mọi Thứ Lệnh Phù Hợp Riêng (Dialect) Dịch Nghĩa Trọn Trị Theo Từng Dòng CSDL Máy Chạy.

### 1.2. Quyền Chống Thay Đổi ACID & Transaction
Tại sao Keycloak Đã Xài RAM Infinispan (Cực Nhanh) Cần Gì Còn Gắn PostgreSQL Nữa Cho Mệt Nặng?
Bởi Vì Infinispan Bộ Nhớ Nhện Đan Rất Mong Manh Bốc Hơi Lỡ Mất Nguồn Khí Tắt Điện Đột Ngột Sụp Trung Tâm Data (Văng Mất Luôn Khách).
PostgreSQL Được Xem Như 1 Sổ Đăng Ký Quần Ngựa Gắn Móng Thép (Khóa Toàn Vẹn ACID). 
Khi Thay Đổi Cái Tên Password, Nó Đòi Hỏi Việc Chỉnh Lửa Mật Khẩu (Phần 1) VÀ Việc Phủ Bóng Nhổ Sổ Lệnh Cache Gây Quét Phiên Nóng Tức Thời (Phần 2) Phải VẬN HÀNH TRONG ĐÚNG 1 ĐƯỜNG ỐNG DẪN KÍN KẼ KHÔNG SÓT (Transaction Commit Cứng). Nếu Chỉnh Nửa Đường Bị Lỗi Dây Chuyền Bắn, Lệnh Giật Rollback Quay Ngược Trả Lại Cũ Nguyên Trạng Tức Khắc Không Sứt 1 Dấu Trầy Kẻ Lấy.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Quá Trình Móng Vuốt Xây Dựng Bản Vẽ Khi Nâng Cấp Hệ (Liquibase Migrations):

```mermaid
graph TD
    subgraph "Mỗi Lần Khởi Động Upgrade Bản Keycloak (Từ V23 Lên V24)"
        Boot[Server Quarkus Bật Nguồn Lên Điện]
        
        Liq[Engine Liquibase Chèn Vào Giao Cắt Database]
        
        Check{Bảng DATABASECHANGELOG Xem Version Số Đuôi Nằm Đâu?}
        
        Check -->|Đang Báo Đứng Ở V23| Run[Chạy Kịch Bản (Changelogs.xml) Thêm 1 Số Cột Client Bị Thiếu Và Đổi Type Cột Mã Lõi]
        
        Run -->|Execute UPDATE SQL| DB[(PostgreSQL Chịu Đựng Lệnh Xuyên Khoan)]
        
        DB --> Đóng[Cập Nhật Chốt Đóng Lệnh Chuyển Lên Bản V24 Vào Ghi Chép Đuôi Cốt Thép]
        
        Đóng --> KC_Start[Keycloak Mới Tự Tin Bung Vành Start Máy Chủ Chính Thức Kéo HTTP Cổng]
        
        Note over Liq,Đóng: Đừng Tự Dùng Lệnh Tay Gõ Sửa Cột Lõi (ALTER TABLE) Trong Database Ngầm.<br/>Liquibase Phát Hiện Mã Checksum Tự Sinh Sai Lệch Nó Báo Lỗi 500 Sập Khóa Database Vĩnh Viễn Đình Bãi Công Trình Không Cho Lên Nguồn Bị Bẩn.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tử Huyệt Nghẽn Cổ Chai Áp Suất Chặn Tại Kết Nối (Agroal Connection Pool)**
> Server Keycloak Có 200 Luồng Công Nhân Rước Khách Lên Cổng (Threads HTTP Quarkus). 
> TRONG KHI ĐÓ, Đường Ống Máng Hút Nước Kéo Khóa Cắm Xuống Postgres (Connection Pool) Mặc Định Mới Khởi Động Đỉnh Chóp Chỉ Mở Đúng Có **20 KẾT NỐI (Connections)**. 
> Tưởng Tượng Cảnh 200 Công Nhân Đi Đổi Rút Hồ Sơ Khóa Chạm Vào Bảng Dữ Liệu Khẩn, Chạy Tới Đụng Kho Kho Tàng, Chỉ Có 20 Cây Bút (Connection) Để Điền Khung Thấm Viết Vét Dữ DB. 180 Kẻ Bị Hất Đứng Đợi Lỳ Chặn Tại Dòng Xếp Hàng Khô Trí. Quá Trình Hết Time-out Gọi Cửa Gây Chết Lịm Luồng 200 Người Hàng Khác Gõ Ngoài Chết Văng Trả Lỗ Đứt Mạch.
> **Cách Cứu Trở Tối Thượng:** Khi Có Số Cụm Cao Trào Request Tải Login Khổng Lồ. Lên File Config Quăng Dữ Liệu Biến Môi Trường Cầu `KC_DB_POOL_MAX_SIZE=200` Và `KC_DB_POOL_INITIAL_SIZE=50` Bằng Trọn Vẹn Đôi Mạch Ống Rễ Thở Khí Giống Nhau Khoang Bụng (Tất Nhiên Chú Ý PostgreSQL Bên Kia Đã Tăng `max_connections=500` Tránh Nó Đạp 100 Connection Chạm Chết Khước Từ Máy Chủ DB Bụng Nhỏ DB-Tràn Lệnh Error Không Còn Bút Đón).

> [!CAUTION]
> **Tội Ác Lưu Log Ở Lòng Đáy (Event Logging Vỡ Bảng Cứng SQL)**
> Keycloak Hỗ Trợ Ghi Event (Sự Kiện - Đăng Nhập Sai Bị Lỗi) Nhét Trực Tiếp Văng Vào Table Database Của Nó.
> **Bi Kịch Khởi Điểm:** Bạn Ép Chạy Cấu Hình Nhét Bật Sáng Chức Năng `Save To Database` Mà Không Cắt Kén Lọc Các Sự Kiện. Đám Khách Bị 1 Trận DDoS Gọi Lệnh Code Bị Fail Auth Error Quăng Trả Liên Thanh. Bảng `EVENT_ENTITY` Điên Tiết Nuốt Khoảng Trăm Triệu Dòng Ghi Chép Chắn Hết Ổ SSD Xót Sạch Trọng Lượng Storage Giới Hạn Của Postgres (500GB Của Server Bị Lấp Chết Rỗng Bảng Cứng).
> Cứ Thế Các Bảng Khác Lòi Rễ Nghẽn Vì Thao Tác Chặn Lấy Table Rác Làm Kẹt Lệnh Query Chạy Timeout Banh 400. 
> **Kế Sách Lõi Giới:** Cắt Bỏ Save DB Các Lệnh Dư Thừa Đời Log Vặt Lẻ. Chuyển Đổi Văng Thẳng Output Mảng Kéo Bằng Kênh Đuôi Chuẩn JBoss Logging Ra Hệ Bắn JSON Kép ELK (Elasticsearch/Logstash) Ngồi Ngoài Tiết Kiệm Chỗ Rỗng Của Ổ Cốt Lõi (Phần Nghệ Thuật Này Nói Ở Lesson 12 Tách Event Phẳng Kênh).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ép Thông Tín Bẻ Chuỗi Tại PostgreSQL Không Có Lệnh 403 Sóng Lệnh Timeout Quá Gắt (Tuning JDBC Parameters Khẩu Quyết Lệnh URL Database Bọc Lõi Keycloak):

Khi Chạy Keycloak. Nếu Không Gắn Đuôi An Toàn Trên URL Khai Báo (Ví dụ: Chết Kẹt Do Rớt Mạng). Cỗ Máy Trì Chạy Connection Chết Chịu Đứng Chết Luôn Lôi Server Bó Chân Vào Đá Rối Hỏng Quạt:
Biến Cầu Giao Môi Trường Dây Buộc JDBC DB_URL PostgreSQL Thép Chuẩn B2B Bắn Trúng Hướng Cấu Hình Chịu Lửa Tải Lặn:
`KC_DB_URL="jdbc:postgresql://pg-cluster:5432/keycloak?ssl=true&sslmode=verify-full&connectTimeout=5000&socketTimeout=10000"`

- `connectTimeout=5000`: Đi Xin Nối Đường Với DB Mà 5 Giây Nó Trơ Mặt Không Thèm Rep (Ví Dụ Cụm DB Bị Đứt Phích Cắm Đang Mở Máy Lại). Cắt Phăng Chạy Đi Tìm Connection Lỗ Khác Khỏi Kẹt Chờ Treo Không Vĩnh Viễn Chạy Hồi Kẽ Bỏ Thread (Ngăn Mạch Cứng Dây Dây TCP).
- `socketTimeout=10000`: Kết Nối Xong Ngon Rồi. Lệnh SELECT Kéo Xuống Nhờ Mạng Lỗi Đứt Chặn Bất Khốc Giữa Đằng Đẵng Rớt Đường Gửi Chậm Gặp Sóng Khó Hút Sụt. Nằm Chờ Dài Dây Qua 10 Giây Tắt Ruột Query Giết Khúc Bắn Ra Exception. Ngắt Cơn Khó Hút Mạch CPU Chạy Cứu Sinh Cho Kẻ Sau Bước Đường Văng Tới Xin Quyền Rút Thở Bào Khối Mới An Nhàn.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Trận Huyết Chiến Giữa Bảng Song Song (Deadlocks - Bóp Cổ Tranh Giành Lệnh):**
  - Môi Trọng Căn Bản Hàng Loạt Giao Dịch Song Song Tự Phá Cầm Thú. User Đăng Nhập Khắp Nơi Cập Nhật Token (Mỗi Giao Dịch Chạm 5 Bảng Liên Kết Mạng Lưới Nhau Xóa Nhóm Bơm User Update Dữ Liệu Chéo Rải Rác). 
  - Khung Cứng RDBMS Có Tính Cố Chấp. Thread Khóa Dòng Row Mở Giới Hạn Hướng UPDATE Bảng A Xong Xin Giết Vòng Update Bảng B. Kẻ Cùng Thời Kéo Vô Từ Lối Lệnh Trái Chiều Ngược UPDATE B Xin Bọc Khóa UPDATE Bảng A Lấp Đè Nút Nhau. (Deadlock Cắn Nhau 2 Vòng Sinh Dây Khóa Chết Nghẽn Cực Ổ Cứng Database Không Chịu Thoát Mắc Kẹt Timeout Gãy Bắn Ngược Cụm Giao Khối Văng 400 500 Rollback Thảm Hại Khó Biết).
  - Vị Cứu Tinh Duy Nhất Là Bắn Sụp Sửa Chữa (Chạy Lệnh Kép Nút Lại Logic Cấu Dữ Liệu Hoặc Xoáy Lại DB Transaction Isolation Bớt Khắc Khe Khát Cầu Nhất Quán Vừa Read-Committed Tại Backend Khớp Sợi).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Thế Giới Keycloak Sợi Quarkus Gấp Đôi Cáp, Công Cụ Kết Nối Agroal Làm Sao Biết Rằng Đường Ống Connection Nằm Chờ 3 Tiếng Đã Bị FireWall Trấn Đóng Cổng Lên Nghẽn Giết Đứng? Việc Trả Lại Cái Ống Chết Đó Cho Khách Xài Sẽ Gây Mất Tín Gọi Bắn Exception Phình Rò Thế Nào?**
- **Junior:** Nó xài thôi chừng nào lỗi thì báo lỗi 500 cho khách đăng nhập lại.
- **Senior:** Cái Tư Duy Đội Chịu Đựng Lỗi Tưởng Bỏ Mặc Khách Tệ Hại! 
Connection Pool (Agroal) Là Môi Giới Cứu Chữa Động. Các Khung Ống Kết Nối Nằm Rảnh Đáy Ngủ 3 Tiếng Bị Mạng Vành Đai FireWall Bắn Lệnh FIN/RST Lén Cắt Ngầm Mà Thằng Giữ Ống Keycloak Đâu Trí Nhớ Giọng Lên Hỏi Kéo Gọi Nên Không Hay Biết Tắt Cụt.
Lúc Khách Chờ Cửa Đập Vô Kéo Dây Rút Trúng Dây Ống Mục Lỗi Nước Rơi Xuống Mới Kéo Báo Đứt Connection Closed! Sập 1 Request.
**Mẹo Cứu Rỗi Xương Tuỷ Backend Lập Trình (Connection Validation):** Bật Khung Kiểm Soát Nền (Background Validation) Tại Pool Cài Biến Kiểm Đảo: `quarkus.datasource.jdbc.background-validation-interval=30s`. Agroal Cứ Nằm Chơi Đi Lại Đi Gõ Cửa Tới Thằng DB Kêu Chú Lệnh Siêu Nhẹ Tí Teo Cỡ (SELECT 1). Hễ Chú Kêu Bị Văng Liệt Ngầm Thì Trục Xuất Cắt Hủy Sinh Tạo Kéo Thay Ống Khác Ngay Thay Dây Xịn Vào Sẵn Đó Chờ Đợi Không Chờ Đập Sập Mới Báo (Health Check Đuôi Chốt Đóng Lệnh Chủ Động Phát Ngược Bơm Kênh Phản Lực Rụng Data Trống Hơi Nhỏ Chặt Cắt Sai Ổ Cứng).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Guide:** Database Configuration (Agroal Connection Pool & Liquibase).
- **PostgreSQL Tuning:** Max connections and Deadlocks Handling.
