# Lesson 11: Đặc quyền Tối thiểu (Least Privilege)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Học cách thu hẹp Vùng Ảnh Hưởng (Blast Radius) khi Hệ thống vỡ trận. Tôn chỉ của Kiến trúc sư: "Chỉ cấp đúng những quyền kiện tối thiểu, cho đúng công việc cần làm, trong một thời gian ngắn nhất".

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Nguyên lý Đặc Quyền Tối Thiểu (PoLP - Principle of Least Privilege)
Bản chất Con người là Lười Biếng. Các Developer rất ghét Lỗi báo Đỏ (HTTP 403 Forbidden). Thế nên để Code cho Lẹ, họ thường xuyên phang luôn quyền Tối Cáo (`Root`, `Administrator`, `AWS_Full_Access`) cho những Dịch vụ cỏn con.
- Cái Ứng dụng Gửi Email (Chỉ cần quyền Đọc Email). Nhưng Dev Lười bèn Cấp luôn Quyền Sửa Database cho cái App đó để "Khỏi lỗi lặt vặt".
- **Hậu Quả:** Hacker tìm ra lổ hổng trên cái Ứng dụng Gửi Email. Hacker trèo qua lỗ hổng đó, và nhờ cái Quyền `Full_Access` dư thừa, Hacker XÓA SẠCH DATABASE TOÀN BỘ CÔNG TY.
**Least Privilege** khắc nghiệt Yêu cầu: Nếu mi Cần Gửi Email, Tao Chỉ Cấp Cho Mi Quyền `SEND_EMAIL`. Nếu mi Ló mặt sang Cột Đọc Database, Tao chém Đứt Cổ (HTTP 403) ngay lập tức.

### 1.2. Thu hẹp Vùng Ảnh Hưởng (Blast Radius)
Giống như Tàu Ngầm có các Khoang cách thủy. Nếu Hacker đánh thủng 1 Khoang (1 Microservice). Nhờ Nguyên lý Đặc quyền tối thiểu, Hacker sẽ BỊ KẸT CHẾT tại Khoang đó. Không có Quyền (Token thiếu Scopes) nên không thể mò sang các Khoang bên cạnh để ăn cắp thêm. Vụ nổ được Khống chế.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Cơ chế Thắt Cổ Quyền Hạn thông qua OAuth 2.0 Scopes:

```mermaid
graph TD
    subgraph "Sự Tiến Hóa Của Quyền Lực (Token Bloat vs Scoped Token)"
        User(Khách Hàng)
        
        Note over User,Bad: Cách Cũ: Cấp 1 Token cầm Tất Cả Quyền
        User -->|Đăng nhập App Đọc Báo| Bad{Token:<br/>Role=User<br/>Role=Bank_Read<br/>Role=Bank_Write}
        Bad -->|Bị lộ Token| ThảmHọa[Hacker dùng Token Rút tiền Ngân hàng]
        
        Note over User,Good: Cách Mới (Least Privilege): Cấp Token Theo Nhu Cầu
        User -->|Mở App Đọc Báo| Good{Token:<br/>Scopes=read_news}
        Good -->|Bị lộ Token| AnToan[Hacker chỉ có thể cầm Token Đọc Báo.<br/>Mang sang Ngân hàng Bị 403 Đuổi Cổ]
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đối Không Sử Dụng Tài Khoản Admin Thường Trực (No Standing Privileges)**
> Nếu bạn Cấp vĩnh viễn Role `Super_Admin` cho tài khoản của Trưởng phòng IT. Đó là Quả bom nổ chậm.
> **Kiến trúc chuẩn:** Không một Sinh Vật Sống Nào trong hệ thống được Mặc định mang Role Admin. Các đặc quyền nguy hiểm chỉ được Bơm Vào (Inject) Dưới Dạng Tạm Thời (Temporary Elevation) trong đúng Khung Giờ Cấp Phép. (Đã phân tích ở Lesson 10 JIT Access). 

> [!CAUTION]
> **Quá tải Scopes (Scope Creep)**
> Lúc đầu App Zalo chỉ xin quyền "Đọc Danh Bạ". Năm sau update, Dev Lén Lút Nhét thêm Quyền "Đọc SMS", "Đọc Micro". Người dùng cứ Quen Tay Bấm YES. Cái App từ Quyền Ít trở thành Kẻ Soi Mói Quyền Lực Nhất điện thoại.
> Trong IAM, Việc Kiểm duyệt Scopes (Quyền Hạn) của các Third-party Apps (Ứng dụng bên thứ 3 xin quyền) phải được Ban Bảo Mật (CISO) ký duyệt thủ công trước khi Gắn lên Máy Chủ IdP. 

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Keycloak thi hành Đặc Quyền Tối Thiểu thông qua khái niệm **Client Scopes**:

- Mặc định Keycloak có một cái Scope Rác tên là `roles` (Nó nhét Toàn bộ Các Danh xưng Quyền Hạn của Bạn vào JWT Token, làm cái Token phình to, và Phơi bày quyền của bạn cho MỌI APP).
- **Tuyệt Chiêu Kiến Trúc Sư:** Vào `Client Scopes`, Gỡ cái `roles` ra khỏi Mặc định (Default). Chuyển nó thành Tùy Chọn (Optional).
- Tạo các Scopes Rất Nhỏ: `view_profile`, `read_payroll`.
- Khi Web Kế Toán (Client A) Xin Đăng Nhập. Nó Phải Truyền parameter OIDC: `scope=openid view_profile read_payroll`.
- Keycloak Đánh Giá: *"À, Client này chỉ xin 2 quyền đó. Tao Chỉ Gói 2 Quyền đó Nhét Vào JWT MÀ THÔI"*.
- Kết quả: JWT Sinh ra Siêu Mỏng, Siêu Nhỏ. Vừa Tăng Tốc Độ Xử lý Băng Thông Mạng, Vừa An Toàn Tuyệt Đối (Least Privilege).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Tài khoản Phá Kính Trọng Yếu (Break-Glass Accounts / Emergency Access):**
  - Giả sử Hệ thống Liên Hiệp (IdP và MFA) của công ty Sập Toàn Bộ. (Ví dụ Máy chủ Keycloak chết, Server Gửi OTP bị cháy).
  - Vì Least Privilege và Zero Trust quá khắt khe, KHÔNG MỘT AI ĐĂNG NHẬP VÀO ĐƯỢC CÁI SERVER ĐỂ MÀ SỬA LỖI. (Hệ thống khóa cổ cả Chủ Nhân).
  - **Bài giải:** Mọi Hệ thống Enterprise BẮT BUỘC Phải Thiết Kế một (và chỉ một) tài khoản "Break-Glass" (Đập Kính Cứu Hỏa). Tài khoản này Nằm Cục Bộ Tại Máy Chủ Ổ Cứng Lõi (Local Admin). Bỏ qua MFA, Bỏ qua SSO. Mật Khẩu Dài 100 Ký tự được Tách Làm 2 Nửa, Giao cho 2 Phó Giám Đốc Cất Két Sắt Khác Nhau (Shamir's Secret Sharing). Chỉ Lôi Ra Ghép Lại Dùng Lúc Thảm Họa Sập Mạng Cục Bộ. (Và Cứ Dùng Là Còi Báo Động Hụ Vang Trên Điện Thoại Sếp Tổng).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Microservices, Nguyên lý Phân Quyền Tối Thiểu (PoLP) áp dụng như thế nào Giữa Bản Thân Các Con Máy Chủ (Machine-to-Machine Auth)?**
- **Junior:** Tụi máy chủ thì tin nhau hết, khỏi cần quyền.
- **Senior:** Đòn Chí Mạng của Bảo mật. Tụi Máy Chủ là Kẻ Lì Lợm Nhất Cần Bị Khóa.
Dùng Giao thức OAuth2 luồng **Client Credentials Grant**.
- Microservice Giao_Hang (G) được Giao Nhiệm vụ Gọi Microservice Kho_Hang (K) để Đọc Tồn Kho.
- (G) Đi Xin Keycloak Cấp 1 Cái Token. Keycloak CHỈ CẤP Token mang Scope là `read:tồn_kho`.
- Nửa đêm, (G) Bị Hacker Lợi Dụng, Bắt Cóc (G) Bắn Lệnh Sang (K) Yêu Cầu `delete:toàn_bộ_kho`.
- Khi Lệnh Tới (K), (K) kiểm tra cái Token của (G). Thấy Token KHÔNG CHỨA Scope Delete. BÙM! (K) Quăng Mã 403 Forbidden vào Mặt (G). Kế hoạch Xóa Dữ Liệu Bị Đập Tan. PoLP cứu mạng công ty.

**2. Khái niệm "Time-Bound Access" (Quyền Có Thời Hạn) khắc phục nhược điểm gì của RBAC (Role-Based Access Control) truyền thống?**
- **Junior:** Giúp tự xóa quyền khi xong việc.
- **Senior:** Nó Diệt Trừ Hậu Họa Của Căn Bệnh **"Quên Thu Hồi Quyền Lực" (Orphaned Privileges)**.
Trong RBAC, một Lập trình viên mới vào công ty. Sếp Tích (Check) vào ô Role `DEVELOPER_AWS`. Xong. Quyền Lực Bị Gắn Cứng Cho Đến Khi Nào Sếp Nhớ Ra Và Gỡ Check (Mà Thường Là Không Bao Giờ Nhớ). Lập Trình Viên Đó Nghỉ Việc Qua Đối Thủ 3 Tháng Sau Vẫn Vào Tải Source Code Công Ty (Từng Khóc Thét Ở Nhiều Công Ty VN).
**Time-Bound (IAM Hiện đại):** Sếp Duyệt Cấp Quyền `DEVELOPER_AWS` KÈM ĐIỀU KIỆN `valid_until="17:00 31/12"`. 
Đúng Thời Khắc Giao Thừa Lệnh Đó Được Bật Ngầm. Quyền TỰ ĐỘNG Bốc Hơi Mãi Mãi. Không Trông Cậy Vào Trí Nhớ Con Người.

**3. Tại Sao Việc Kiểm Toán (Auditing/Logging) Lại Là Nửa Kia Bắt Buộc Của Least Privilege?**
- **Junior:** Log để sau này có cãi nhau lôi ra coi.
- **Senior:** Least Privilege Bản Chất LÀ MỘT QUÁ TRÌNH THỬ VÀ SAI (Trial and Error).
Sẽ Không Một Chuyên Gia Nào Biết ĐƯỢC Một Cái App Mới Lên Chạy CẦN CHÍNH XÁC Bao Nhiêu Chút Quyền Hạn. (Đa Số Sẽ Cấp Thiếu Quyền Làm App Chết, Hoặc Cấp Dư Quyền Làm Rủi Ro).
Kiểm toán (Auditing) Cung Cấp Đôi Mắt. Ta Chạy App Bằng Quyền Rộng Rãi Trong Môi Trường TEST 1 Tuần. Ta Bật Máy Quét Log Lên Coi: *"Trong 1 Tuần Qua, Cái App Này Thực Chất Dùng Quyền Gì?"*. Máy Báo Lại: "Nó Chỉ Dùng Quyền Đọc DB, Chứ Mấy Cái Quyền Ghi File/Xóa Thư Mục Anh Cấp Nó KHÔNG HỀ ĐỤNG TỚI". 
Dựa Vào Báo Cáo Đó, Kiến Trúc Sư Cầm Dao CHẶT BỎ CÁC QUYỀN DƯ THỪA. (Quy trình Privilege Discovery & Refining). Không có Log, Least Privilege Trở Thành Kẻ Chọc Mù.

---

## 7. Tài liệu tham khảo (References)
- **NIST SP 800-53:** Security and Privacy Controls (Access Control Family).
- **AWS IAM Best Practices:** Apply least-privilege permissions.
