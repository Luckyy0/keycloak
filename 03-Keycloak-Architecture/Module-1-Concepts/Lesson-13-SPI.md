# Lesson 13: Cắm Rút Tiện Ích Mở Rộng (SPI)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khám phá Thứ Vũ Khí Sát Thủ Của Keycloak Giúp Nó Đè Bẹp Mọi Đối Thủ Đóng Móng (Auth0, Okta). Đó Là: Service Provider Interfaces (SPI). Muốn Sửa Tính Năng Gì, Không Cần Chọc Bụng Sửa Lõi Mẹ Mã Nguồn Mở, Cứ Cắm Mũi Tiêm File `.jar` Gắn Phẳng Vào Ngoài Rìa Là Xong.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. SPI Là Cổng Cắm Bo Mạch (Motherboard Slots)
Khi một Khung Phần Mềm Kín Trọng (Platform) Ra Đời. Bạn Bị Khóa Chặt Vào Tư Duy Của Thằng Sinh Ra Nó. (Nhà Cung Cấp Xây Cho Khẩu Súng Đạn Nhựa, Đem Bắn Chó Sói Bị Cắn Cổ Mất Xác Chạy Bất Khóc Đáy Rỗng).
Keycloak Được Sinh Theo Thiết Kế Open Architecture: Toàn Bộ Thân Hình Nó Đục Sẵn Cả Trăm Cái Lỗ (Interface Cổng Cắm Rút SPI).
- Lỗ Gắn Tùy Biến Lệnh Login (Authenticator SPI).
- Lỗ Gắn Tùy Biến Gửi Nhắn (EmailSender SPI).
- Lỗ Trút Sóng Tráo Data (UserFederation SPI).
Bạn Không Cần Đợi Đội Kỹ Sư Của RedHat Nâng Cấp Bản Mới Chờ Suốt Năm Nữa Mới Có. Bạn Tự Viết Cái Chui Cắm Mã Trái Dữ Liệu Nhét Vô, Nó Khớp Sóng Đè Vọt Đạp Đáy Chạy Tức Thời. Mọi Doanh Nghiệp Có Logic Xác Thực Kỳ Quặc Đều Khung Được Tại Bức Rào Này Đập Bóng Ngược Hợp Logic Gắn Tại Chỗ.

### 1.2. Tuân Thủ Trục Dòng Factory-Provider
Như Đã Học Ở Bài 2 (Components). Mọi Cái SPI Bạn Viết Bắt Buộc Phải Chứa Khung Bọc 2 Miếng Kẹp Song Sinh:
1. `Mã_Tên_Factory.java`: Khai Báo Trưng Bảng Hiệu Tên Chức Năng Của Tôi (Tạo Động Lực Sinh Mầm).
2. `Mã_Tên_Provider.java`: Chứa Cục Code Thực Thi Chạy Đè Tắt Dọn Cắt Cụm Nóng RAM Bất Khước Cháy Nhiệm Vụ Xong Là Tắt Mạch.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Gắn Giao SPI Lên Lõi Vận Hành Server:

```mermaid
graph TD
    subgraph "Cách Keycloak Nuốt Mảnh SPI Của Khách Khung Ngoài"
        Code[Dev Viết Code Java Đóng Thành Custom.jar]
        ThuMuc[Quẳng Vào Thư Mục Khởi Động: /opt/keycloak/providers]
        
        Boot[Gõ Lệnh: kc.sh build]
        Boot -->|Quarkus Thuật Phân Giải Bytecode Mạng| Chay{Quét Thấy Khai Báo Dịch Vụ java.util.ServiceLoader?}
        
        Chay -->|Đăng Ký Factory Vào RAM Khung Lõi Trái Đứng Sóng Nội Bộ| BuildXong[Gói Cứng Build Thành Lõi Đục Nguyên Khối Phẳng Immutable]
        
        BuildXong --> Start[Gõ Lệnh: kc.sh start]
        Start -->|User Tương Tác Chọn Cấu Hình Đập Đúng Logic Gắn| Chạy_Code_Custom
    end
    
    Note over ThuMuc,Start: Điểm Sát Khí Đổi Mới Của Quarkus:<br/>Thời WildFly Chạy Xưa, Có Thể Quẳng File JAR Vô Lúc Server Đang Sống Chạy Bình Thường (Hot-deploy).<br/>Thời Quarkus Vĩnh Viễn Cấm Chạy Phá Ngầm Nóng!<br/>Cắm File Vào LÀ PHẢI BUILD LẠI ENGINE 1 PHÁT ĐỂ ĐÚC VỎ LÕI (Closed-world assumption). Chạy Nhanh Gấp Tỉ Nhưng Hết Cửa Sửa Sống Nóng Rớt File.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tránh Trở Thành Nút Thắt Kẹt Luồng Chắn Sập Hệ Thống (No Blocking in SPI)**
> **Án Mạng Toàn Cục:** Bạn Viết 1 Cái `Authenticator SPI` Để Lôi Tọa Độ GPS Của User Qua Map API Ngoài 3rd-Party Chờ Định Vị (Chống Đăng Nhập Xuyên Bang Mỹ Bị Phạt). Mạng API Bản Đồ Bên Kia Bị Hỏng Trả Code Trễ 10 Giây Tắt Rớt Lệnh Socket Mòn Rỗng.
> **Hậu Quả Mạch Kéo:** Thread HTTP Của Keycloak (Xử Lý Yêu Cầu Cửa) Bị Ngậm Đứng Ôm Lệnh Chờ API Bản Đồ. 200 User Bấm Nút Login Tức Là 200 Thread Cháy Sập Đứng Đợi Lệnh Bản Đồ. MÁY CHỦ KEYCLOAK CHẾT CỨNG Không Quẫy Được Không Giao Lệnh Chặn Lọc 401 Được Dù Đáy PostgreSQL Nằm Rảnh Mát CPU Rớt Xuống Khung.
> **Luật Thép Sống Còn Viết SPI:** Mọi Phép Gọi Network Call Qua Máy Khác TRONG Lõi SPI **Bắt Buộc Bọc Lệnh Timeout Nhanh Nhất (Chậm Quá 1000ms Là Chém Hủy Trả Exception Đuổi Khách Báo Lỗi)**. Tuyệt Đối Không Cho Bất Kỳ Luồng Lệnh 3rd-Party Trói Giam Lực HTTP Worker Của Trục Server Chính Đứng Tại Đáy Mò Ngóng Bọn Lệnh Thối Hư Trực.

> [!CAUTION]
> **Cái Bẫy Đứt Gãy Khi Sang Bản Mới (API Breaking Changes)**
> Keycloak Nâng Cấp Phiên Bản Mới Nhanh Như Gió (Mỗi Quý Đẻ Ra 1 Bản). Các Giao Diện Interface SPI Dưới Đáy Móng Java Của Họ Thỉnh Thoảng Bị Xóa/Đổi Tên Hàm Rễ (Phá Hợp Đồng Contract). 
> Nếu Công Ty Bạn Đội SPI Viết Cả Đống Mảng. Lên Keycloak 24 Bấm Build Rớt Mạch Compiler Báo Lỗi Đỏ Chót Cụt Cả Cụm 400 Trục Hàm Lỗi MethodNotFound. Bạn Mắc Kẹt Không Dám Lên Bản Mới Khớp Gắn Lệnh Vá Lỗ Hổng Nạn Nhân Hacker Xuyên Rò Sóng (Nợ Kỹ Thuật Tech Debt Gắn Đáy Kẹt Khớp).
> **Khuyên Chặn Mạng:** Hạn Chế Trút Mọi Logic Dữ Liệu Đời Sống Vào Viết SPI. Nên Bóc Nó Ra Bằng Chút Mảng Viết Mỏng Gọi Nhánh API Ra Backend Viết Gọi Code Bên Kia Đóng Lệnh Chặn Data, Hư Lệnh Code Ta Tự Chịu. Giữ SPI Mảnh Dẻ Khẳng Đuôi Dễ Bóp Đổi Gắn Rễ Build Chéo Nhanh Hồi Chấn (Tối Giản Dependency Trục Áo).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh SPI: Bắt Buộc Xác Nhận Bằng Dấu Vân Tay Qua Form (Custom Authenticator):
Luồng Cấu Hình Lôi SPI Ta Tự Code Lên Chạy Bằng Flow (Giao Diện Bề Mặt Hóa Rắn Mạch Lõi):
1. Code Xong 1 Thằng Java Gọi Là `VanTayAuthenticator`. Đóng Chút Tên `id="van-tay"`. Build Rớt Nhét Vào File Lõi Tắt.
2. Vô Admin Console, Chuyển Đến **Authentication -> Flows**.
3. Copy Dây Đăng Nhập Cũ (Browser Flow) Ra 1 Bảng Mới Đặt Tên `Browser Với Vân Tay`.
4. Add Step (Thêm Bức Tường). Bấm Vào Ô Tìm, Tự Nhiên Thấy Cục `VanTayAuthenticator` Của Mái Chèo Ta Vừa Đúc Nằm Dáng Chiễm Chệ Kế Bên Mấy Đàn Anh Đỉnh Chóp RedHat Lệnh.
5. Cắm Nó Vào, Set Dấu `REQUIRED`. Chỉnh Bind Flow Để Ép Luồng Chạy Cho Ánh Sáng Nhận Biết Sợi Khớp Dây Chữ Mở.
XONG! Bạn Vừa Phá Thể Chế Authentication Auth Khống Khung RedHat Ép Thành Bản Chấp Lõi Hư Vô Code Trái Thắng Lệnh Form Sinh Trắc Phẳng Riêng Biệt Hóa Trọng Lệnh Doanh Nghiệp Đỉnh.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Vỡ Mạch RAM Tại Biến Tĩnh Văng Đụng Độ Chéo (Static Variables Leak Của Provider):**
  - Dev Thích Đóng Mảng Khai Báo Biến Tĩnh `static List<User> cacheNhanh = new ArrayList<>()` Ở Bảng Lệnh `EmailProvider.java`.
  - Hàng Ngàn Yêu Cầu Chạy Vô, Thằng Tí Bỏ Data Rác Vô Biến `cacheNhanh` Bị Nhầm Bóng Của Thằng Tèo Chạy Chéo Ngang Đa Luồng. Trình Báo Thư Trút Bơm Gắn Nhầm Mật Khẩu Chéo Người Này Qua Thằng Kia. (Lỗi Chạm Tuyến Concurrent Đỉnh Mệnh Vô Đối Lệnh Rơi Sai Tên).
  - Không Có Spring Đỡ Lệnh Đạn Rác Cho Đâu Nữa Đâu Khung Khống Lệnh Java Trúc Trắc Tự Cắt Dọc Luồng Từng Khách Rất Chú Tâm (Luôn Gắn Biến Dữ Vào Ruột Nhọn Giao Thức Gọi Thuần Local Method Tránh Data Global Share Xoáy Rác).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Giai Đoạn Viết SPI Đặt Nối (Ví dụ Thêm Người Dùng Từ Form Nhập), Nếu Tôi Muốn Dùng Trọng Lệnh Gọi Đọc Từ PostgreSQL Kéo Ra. SPI Có Cho Phép Tôi Tiêm Cả Khung Database (EntityManager) Bằng Khớp Hibernate Của Ta Vào Cắt Giao Được Không? Hay Bắt Buộc Dùng Hàm Giới Hạn Của Keycloak Bọc Vứt Hẹp?**
- **Junior:** Mở port trỏ thẳng jdbc chọt vô database luôn cho lẹ, có password mà lo gì.
- **Senior:** Một Quả Phá Hoại Architecture Khủng Khiếp Nếu Cho Phép Connection Hỗn Loạn! 
Keycloak Tuyên Thệ Kiến Trúc **One Session Rule (Luật Đơn Phiên)**. 
Trong Ruột Lệnh SPI (Hàm Trích Method). Khung Cắt Mẹ Sẽ Truyền Cho Bạn Duy Nhất 1 Vật Thể Tôn Giáo Đỉnh Cao: `KeycloakSession session`. 
Cục Này Đã Cầm Bao Gồm Trọn Gói Entity Manager Lẫn DB Connection Tròng Vòng Giao Dịch (Transaction) Kép Ngầm Xong Rồi Nhá! Bạn Không Cần Mở Giao Ống Connection Nào Cả. 
Chỉ Việc Gõ Lệnh `session.users().getUserByUsername(...)`. Nó Tự Chạy Xuống Đáy JPA Rút Mạch Trùng Ống Luồng Với Thằng Khởi Gọi Đảm Bảo Toàn Vẹn Atom Chống Lệnh Trái. Hễ SPI Viết Code Bấm Mở Connection JDBC Trái Tuyến Dọc DB Là Vi Phạm Lệ Đứt Đi Rò Ống Chết Sập Connection Pool!

**2. Nếu 1 Thằng API Khác Của Công Ty Nhờ Keycloak Tính Toán Bằng Mã Code Viết Thêm (Tính Băm Mã Doanh Thu Trả Nhẹ Chẳng Hạn). SPI Nào Cho Phép Đục Cửa Đóng Lỗ Lệnh Bơm Ngắn Mở Rộng API Mới Endpoint `/doanhthu` Để Ra Hứng Trọng Token Cổng HTTP Luôn Tại Server Keycloak Mở Mạch?**
- **Junior:** Không đục được, Keycloak chỉ chặn cửa đăng nhập thôi chứ đâu phải Spring boot.
- **Senior:** Bạn Chưa Đọc Quy Tắc RealmResourceProvider!
Keycloak Cho Phép Cắm Nóng Chút Lỗ Hổng Bơm Khung Đỉnh Tên Gọi: **Realm Resource SPI**. 
Nó Cho Phép Bạn Đăng Ký Viết Các Đường Dẫn Endpoint JAX-RS Riêng Lạ Hoắc Ngay Đằng Đáy Mạng Rễ Nhập Ngắn Khung Cấu Trúc (VD: `/realms/vingroup/thongke/doanhthu`). Cổng Lỗ Phẳng Này ĐƯỢC THỪA HƯỞNG Hết Quyền Kẹp Cốt Thép Xác Thực Chắn Mạng Của Lõi Keycloak (Đảm Bảo Sóng Bảo Vệ Xịn Xò Xé Chữ Ký Giao Bắt Rễ OIDC Đóng Bằng Tự Sinh). Rất Tuyệt Để Khung Ra Những Nút Nới Lỏng Giao Mạng Nhỏ Tắt Từng Mảng Nghiệp Vụ IAM Tại Điểm Ảo Giữa Cổng Phẳng Không Cần Dựng Hẳn 1 Cụm Microservice Riêng Rác Mạng.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Developer Guide:** Service Provider Interfaces (SPI) and Custom Extensions.
