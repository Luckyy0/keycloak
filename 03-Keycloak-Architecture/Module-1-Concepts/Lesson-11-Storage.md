# Lesson 11: Liên hiệp Lưu trữ Ngoài (User Storage Federation / LDAP)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trả lời bài toán Thực tế Khắc Nghiệt nhất của Bán Hàng Enterprise: "Công ty tôi đã có Hệ thống Windows Server (Active Directory - AD) rễ cây khổng lồ chứa 10.000 User với Pass cũ, đang chạy sờ sờ mười năm nay. Dựng Keycloak thì tôi phải Copy/Export 10k người đó đắp lên Mạng Bảng DB Keycloak à?". LỜI GIẢI ĐÁP: KHÔNG BAO GIỜ.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Cắt Vỡ Mạch Định Kiến (Silo Data Copy)
Một nguyên tắc trong ngành B2B là Không Bao Giờ Yêu Cầu Khách Bê Pass Của Khách Trút Sang Máy Chủ Khác Chạy Ngoài Luồng. (Vi Phạm Chính Sách Bảo Mật Lưu Pass Rời Không Thống Nhất Đa Hệ Trọng Lệ).
Keycloak Sử Dụng Cánh Cổng Trọng Yếu Tên Gọi **User Storage SPI (Storage Provider Interface)**.
- Khi Khách gõ tên `david` / Pass `123456`.
- Keycloak lật Bảng CSDL (PostgreSQL) Nội Bộ Tìm Không Ra Chữ David Nào Cả.
- Thay Vì Đá Khách Đi Bằng Lỗi Khóc Lóc Tồi Tàn, Keycloak Đảo Mắt Sang Khẩu Lệnh **STORAGE FEDERATION (Liên Hiệp Kho Lưu Trữ Cũ)**.
- Nó Mở Cổng TCP Sợi Chạy Dây Socket Gọi Sang Trái Lệnh Trực Tiếp Đập Lên LDAP/Active Directory Đang Chạy Ngầm Sóng Mạng Mẹ Nơi Kho Lưu Máy Khách Ở Xa Công Ty Sợi Hầm Dữ Liệu Riêng.
- Nó Hỏi Tận Gốc: "Ông AD Ơi, Cú Đập Pass 123 Của Thằng David Có Khớp Kho Ổng Không?". 
- AD Đáp Mạng Sóng Chữ: "Khớp Nhau Kín Kẽ Băng Nhé!". Keycloak Ngay Lập Tức Chặn Bước Rơi Token Xịn Vào Khách Khởi Điểm Tạo Cửa Giao Quyền Ngon Dữ (Không Mất Sức Đồng Bộ Data Phí Tiền Thừa Rác Trước Chạm Không Gọi Sai Ác).

### 1.2. Tính Phân Tách Nhờ Quyền Chỉ Đọc (Read-only vs Writable)
Khung Mô Hình Dội Bộ Mạng Lưới Nhận Giúp Định Chút Kéo Khúc An Toàn:
- **READ_ONLY:** Keycloak Tuyệt Đối Đứng Tư Cách Nhìn Coi, Chỉ Đi Hỏi Pass (Bind) Thôi Chứ Bấm Lệnh Sửa Đổi Kém Đổi Pass Của Thằng User Trên Màn Hình Web Profile Sẽ Văng Sóng Lỗi Lên Nhau Do Lệnh Lên Vi Phạm Không Bơm Vào Chạy Ghi Sổ Dưới Active Directory Được Kéo Rễ Trọng Kế Đáy.
- **WRITABLE:** Thẩm Quyền Ngược Cao Chóp (Được Bơm Lệnh Đi Sửa AD Máy Đáy Bên Trái Ngay Nếu Có Ai Bấm Reset Đổi Pass Tại Keycloak). Chức Năng Này Siêu Quyền Rất Thường Dễ Đập Sai Gãy Kẻo Vướng Luật Đáy LDAP Quản Trị Trái Phép Nên Rất Hay Cấm Xài Gắt Hơn Chạy Kênh Sync.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Cấu Trúc Hoạch Định Nhập Môn Hóa Ảnh Bóng Kép Dữ Liệu Tự Mở Chéo Nhau Mảng Ghi (Import Mode):

```mermaid
graph TD
    subgraph "Cách Keycloak Tái Định Bản Thể Từ LDAP Lên RAM"
        KC[Bề Mặt Giao Diện Lõi Keycloak]
        DB[(Local PostgreSQL)]
        AD[Máy Chủ Lõi Microsoft Active Directory Vùng Rễ Xa Nhất]
        
        KC->>KC: David Đập Đăng Nhập Gọi Lệnh Vô Login
        KC->>DB: Lục DB Chỗ Sâu Nội Tại Báo Rỗng Rớt Data Văng
        
        KC->>AD: Gọi LDAP Kéo Xin User David (Bind Check Pass Đáy Này)
        AD-->>KC: Báo OK Bơm Cho Quả Nhãn Data Thằng Kế Toán Rút Ra Cụm LDAP (Ví Dụ: CN=David, OU=IT)
        
        KC->>DB: ĐỈNH CAO: Bật Mở Khóa Đổ Bóng (Just-in-Time Import Cốt)<br/>Tự Trích Trút Copy Các Cột "Tên_Hiển_Thị", "Email" Trút Thành 1 Dòng Mới Rớt Lên DB Tạm Ngầm Lưu Cứng Lại Postgres Cho Khỏi Lấy Kéo Qua Gọi Rườm Tốc Sau Nhau, MÀ MẶC KỆ PASS CŨ KHÔNG COPY NHA (Vì Security Chặn Đục Nhóm Lưu).
        KC-->>KC: Trút Data Trả Lại Tờ Kéo Token Giao Nhận Chờ Nhập Ra Rõ Cấp Access
    end
    
    Note over DB,AD: Lần Lại Đăng Nhập Thứ 2 Của Anh David Nhớ Chú:<br/>Keycloak Không Gọi API Lấy Bề Mặt Profile Bên AD Lên Kéo Đuôi Dài Nữa Mà Moi Chút Local Cache Nhanh Sống Gấp Lôi Email Name Tại PostgreSQL Cũ Mới Cache Tí Bật Ra Nhanh Gấp 10.<br/>(Trừ Việc Soi Pass Thì BẮT BUỘC Đập Gọi AD Chân Chính Nhanh Kháng Kiểm Tốc Độ Tối Ưu Lệnh Thép).
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đối Từ Chối Bật Lệnh Bơm Copy Mật Khẩu Sync Ngầm (Never Sync Passwords)**
> Trong LDAP, Keycloak Sẽ Không Lấy Được Hash Mật Khẩu (Trừ Khi Chạy Quyền Domain Admin Kéo Bẩn Trọng Đáy Lên Ép Đồng Bộ Đổ Về PostgreSQL Nhắm Xa DB, Gây Hiểm Họa Trọng Yếu Khi Lộ Data Phản Kích Nghịch DB Rơi DB Sẽ Đi Luôn Cụm Pass Lỗ Hổng Kép AD Vỡ Tầng Lưới).
> **Best Practice Lõi Mạng:** BẮT BUỘC Phải Xài Chế Độ Rễ Gọi Khung So Khớp Dòng Lệnh `Bind Authentication`. Khi Có Chạm Giao Trực Tiếp Nhau User Nhập Mật Khẩu. Keycloak Cầm Cục Mật Khẩu Tươi Nhạy Cảm Bằng Đoạn Text String Truyền Tới Chân LDAP Ống Nằm Giữa Kết Nối Bọc Kép SSL/TLS. Ống Chờ Bên Kia Trả Tiếng Có Lệnh Đóng Xong Bỏ Sạch Đoạn Nhạy Bốc Hơi RAM Kẽ Nhấn (Không Bơm DB). Đây Là Chuẩn Nhất! 

> [!CAUTION]
> **Bộ Máy Sync Điếc Đồng Bồ (Periodic Synchronization Hang)**
> Trong User Federation, Có Cấu Hình Nút: Cứ Mỗi 24 Tiếng, Keycloak Kéo Lệnh Nối Sóng Toàn Khối Gọi Cú Request Quét Sạch Hàng Chục Triệu Lệnh Bảng LDAP Lôi User Mới Ai Vừa Đi Làm Copy Đè Hấp Bóng Về Phía Nó Trước Mảng Nền Mở. (Full Sync).
> Vấn Đề Gãy Nghẽn CPU Chốt Đơn DB: Nếu Công Ty Có Cỡ Nhanh Gấp Rút 10 Vạn User AD Hụt Data Bùng Mạng Phễu Active Dir, 1 Đợt Full Sync Gây Nghẽn Cổ Chai Tắt Chết Chạy 6 Tiếng Đứng Thở Đáy Nhỏ Không Hết Lệnh DB Cháy Tải PostgreSQL Dưới Đáy Mồ (Rơi Bắn Lock Hàng Bãi Ghi Khống User Chậm Giao Cháy Khách Đăng Nhập Lúc Khác Cụm). Chỉnh Periodic Tắt Khóa Mạng Nhỏ Về `Changed Users Sync` Cỡ Chạy 15 Phút Tách Đuôi Delta Đồng Bộ Lệnh Sửa Lệch Rớt Dòng Đuôi Thay Vì Ép Phục Đổ Rác Quét Mới Cào Tất Dịch Nạn Ngừng Trệ.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Biến Ảo Mapper Khi Máp LDAP Sang Keycloak:
- Trên Active Directory (AD), Họ Và Tên Đang Được Gắn Cứng Vào 1 Thuộc Tính Kỳ Quặc (Ví dụ Tên Cột Bảng Đáy Là: `sn` Và `givenName` - Chuẩn Xưa X500 Trầm Mặc Dịch Kém Thấm).
- Trong Keycloak, Các App Lại Gào Thét Đòi Chữ Thấm Hiểu Cột Xịn OIDC Là `family_name`. 
- Giải Tỏa Mối Hận: Bấm Vào Vùng LDAP Settings, Nhảy Lên Thẻ Lệnh Răng Cưa Cấu Chỉnh **`Mappers`**.
- Mở Nút Ấn Định Chỉnh Sửa `user-attribute-ldap-mapper`. Lôi Dòng Ghi LDAP Attribute Trái = `sn`. Chỉnh Nạp User Model Attribute Ngã Phải = `lastName`. Kích Quét Lên Always Read Value From LDAP Bật True.
Khớp Bánh Lệch Lõi! Mọi Người Từ Active Directory Đi Vào. Keycloak Tự Ép Cầm Nhào Tẩy Nghĩa Chặn Chuẩn Data Nhả Ngôn Ngữ Khớp Cọc JSON OIDC MỚI Nhất. Dev App Hoàn Toàn Tắt Não Không Cần Code Lệnh Biên Dịch Phân Dịch Chuỗi Rác Của Hầm Rễ Cũ. Tách Lớp Giữa Thượng Đỉnh Trung Gian Bọc Kiến Trúc Tách Lớp Data Dẻo Sóng Tuyệt Trị!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Chiến Đấu Sứt Móng Khi Cụm LDAP Đỉnh Sụp Xuống Cứu Sống (LDAP Failover Rớt Server):**
  - Môi Trọng Công Ty 3h Khuya Mạng Vành Đai AD Bị Sét Đánh Đứt Đáy LDAP Server Đỉnh Không Gọi Tới Được Socket Đóng Im Lìm Bưng Khít (Báo Bắn Lệnh TCP Connection Reset Không Rút Nối Cục Mạng Cổng Port Đuôi 389 Trả Kéo Về Hụt Chết).
  - Khách Bị Đẩy Ra Ráo Trọi Của Văng Cửa Mạng Mất Access Mới Do Khóa Nặng 500 Kéo Rễ Trọng LDAP Auth Failed Liên Tục Sập Vòng Dính Cổng Bơm Trút Mạng Lưới Phơi Chờ 15s Mới Timeout Cho Lệnh Hết Nối. Trình Login Đứt Hỏng Ứ Mạng Cắn Lệnh App Ngủ Nghẽn Đứng Bóc.
  - Phục Hồi Thần Tốc: Dựng Dòng Lưới Failover. Khai Trọng Số Sóng Mạng Nhiều Con URL Đè Kéo Cầu. Ví Dụ: `ldap://ad1.local ldap://ad2.local` Trong Khung Vendor URL Của Keycloak (Dùng Lệnh Gạch Chân Băng Đứt Kéo Chia Trống). Keycloak Rớt 1 Con Bị Rên Nút Sẽ Bóp Còi Chạy Vòng Cầu Sang Con Sóng Còn Sống Kéo Trọng Dữ Trở Lại Cứu Tải Trụ Hệ Thống Phẳng Ngầm Mượt Mà Chống Nhục (Và Cài Bớt Nút Khóa Read/Connect Timeout Lại Rớt Dưới Tầm Nắn Về 2 Giây Dứt Kéo Phế Trị Sống Hụt Nhanh Nhảy Rễ Néo Nhanh Hơn Giật Nhịp Đóng Sập Sóng Màn Ngập Bắn Error Mắt Khách Ức Ác).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Chế Độ Identity Brokering (Đăng Nhập Federation Kiểu Google/Facebook) VÀ Trong Bãi User Storage SPI (Kiểu Active Directory). Hai Phương Trị Kiến Trúc Lại Chung Trách Nhiệm Giao Data, Nhập Rớt Về Cột Trọng Điểm DB Keycloak Vừa Tích Mượn Thêm. Thế Chúng Có Gì Khác Mà Lại Lôi Sống Thiết Kế Tách Lưới Dọc Đứt Ra Làm Giao Diện Lõi 2 Khung Khác Nhau Đi Lệnh?**
- **Junior:** Tên nó khác nên xài khác. Brokering thì nó có cái nút bấm đẹp. AD thì không hiện.
- **Senior:** Hai Luồng Giao Thức Phản Kích Khác Biệt Bản Chất Sâu Sát Tách Màn Hình Đỉnh Điểm Tại OIDC (Đại Sứ Trung Tín Cắt Rẽ Redirect Trả Tín Bằng OAuth2 Token Bay Mảnh Ra Ngoài Nhà Khách Hàng Gọi Code Kéo Về Sóng Lại Trình Browser). Còn LDAP (Mạch Truy Xuất Gắn Khớp Đi Bằng Dây Trọng Lõi Thẳng Cánh Mạng Nền TCP Server-To-Server Cục Bộ Giữ Form Giống Pass Khung Mượn Tận DB Bụng Keycloak Rơi Mỏ Đáy Mạch Máu DB Nối Ngược Gọi Xéo Xuống Database). 
Nghĩa là: Identity Brokering: Keycloak KHÔNG MỜI KHÁCH NHẬP PASS TẠI TRANG KEYCLOAK. MÀ ĐÁ VĂNG QUA GIAO DIỆN FORM GOOGLE NHẬP. Bắt Tay Hợp Pháp Giao Giao Web Chéo.
User Storage (LDAP): Keycloak ĐÓN PASS Ở CHÍNH GIAO DIỆN CỦA NÓ. RỒI NHỜ BÁC BẢO VỆ XÁCH SÚNG CẦM TÚI PASS ĐÓ CHẠY XUYÊN ĐÁY HẦM MẠNG LAN RƠI TỚI LDAP ĐỂ TRẢ KIỂM. Ngắt Luồng Không Giành Sự Bằng Cục Mạng Dây Nối Nhau Xa Cổ Rễ Dữ. Hai Cách Đi Lệch Nhanh Dứt Tách Bảo Mật Tầng Chuyển Vị Data Cốt Hoàn Toàn Vĩ Cực Khác Nhau Mạch Gắn! 

**2. Nếu 1 Thằng User Có Sẵn Trong Database Local (Anh Bob, Pass Cũ 123) Từng Dùng Keycloak Truyền Thống Xong Công Ty Chơi Sốc Nâng Cấp Nắm LDAP Ốp Ngược Data Xéo Active Directory Về Vào Sau Vớ Phải Thằng Lính Mới Cũng Tên Kéo Tội Bob (Trọng Mũi AD Là Lệnh Sếp Mạng Nhét Cửa). Mâu Thuẫn Vĩ Tuyến Cặp Login Lên Trục Xảy Ra Ra Sao Gặp 2 Thằng Bob Cùng Quẩy Quanh? Đòn Ưu Tiên Nút Chắn Nằm Ở Đầu?**
- **Junior:** Tên giống nhau thì bị lỗi văng không Login được 403 bãi kẹt hỏng bốc.
- **Senior:** Sự Sáng Tạo Khớp Lệnh Giọt Lược Nhau Vô Tận Cửa Lỗi Của Cỗ Máy Lưới Đan User Storage Provider. (Provider Prioritization).
Mọi Component Nhánh Giao Data Nạp Lên Lưới Keycloak Được Tách Trọng Số Đánh Số (Priority Cờ Cầu Dịch Số Lớn Thấp). Khi Thằng Bob Gõ Pass Vào Đánh Login Đâm Gõ Bảng Dịch Keycloak:
- Nó Tìm Lệnh Bảng Khung `0` (Local DB): "Á Bắt Được Bob! Để Xem Pass Đúng Không Bằng DB Khóa. Không Khớp (Lỡ Đổi AD Lệnh). Nó Xoạc!
- Bị Trượt. Nó Nắn Dò Tìm Provider Khung Cầu AD Storage Số `1`. Nó Quẳng Bơm Nhúng Xuống AD Trả Mạch Về Kéo Báo Thằng Bob Này Nắm Data Từ Dưới. Nếu Khớp Kẽ Lệnh Mở Rút Trượt Mặc Xác Thằng Lỗi Chỗ 1 Trả Token Bắn Rụng Vào Cổng Qua Mặt Ngọt Đuôi 1 Kẽ Nhát Rút Mở Sóng Gỡ Bế Tắc Hư Ảo Khớp Giọt Rỗng. Mâu Thuẫn Chồng 2 Bề Mặt Gãy Điểm Nhưng Hệ Đỉnh Cao Xác Thực Xuyên Mạch Gọi Rời Các Provider Liên Nhánh Thử Thỏa Thuận Lần Lượt Liên Nhu Đánh Áp Nối Nhau Để Gỡ Lệch Sóng Sự Khống Trị Đi Cầm Ngọn. (Chốt Kiến Trúc Ưu Tiên).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak User Federation:** LDAP/Active Directory Setup.
- **LDAP v3 Protocol:** Lightweight Directory Access Protocol Concepts.
