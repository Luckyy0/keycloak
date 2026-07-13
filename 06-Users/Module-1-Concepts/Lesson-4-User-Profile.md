# Lesson 4: Bản Hiến Pháp Dữ Liệu (Declarative User Profile)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Trong nhiều năm, Keycloak bị Chửi bới thậm tệ vì việc cho phép Nhập Dữ Liệu Khách Rác (Bạn muốn lưu Số điện thoại, nhưng khách nhập chữ "abc" thì Keycloak vẫn lưu). Từ phiên bản 24 trở lên, **Declarative User Profile** trở thành Vũ Khí Chính Thức. Nó Thiết quân luật Toàn bộ Trường Dữ Liệu Của Khách: Khai Báo Kiểu Gì (String/Number), Ai Được Sửa, Validator Gì (Regex/Length). 

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Từ Bãi Rác (Wild West) Tới Luật Pháp (Declarative Profile)
**Trước Đây:** Dữ Liệu Bụng Người Dùng (Thuộc Tính Khác Ngoài Email/Username) Bị Lưu Trữ Một Cách Ngu Ngốc Dạng Cặp Khóa - Giá Trị (Key-Value) Text Thuần. Cậu Kỹ Sư A Gõ Key Là `so_dien_thoai`, Kỹ Sư B Lại Gõ Key Là `phone_number`. DB Chứa Khung Rác Mạng.
**Ngày Nay (Declarative Profile):**
Sức Mạnh Của Cụm OIDC Nắm Kẽ Rỗng Đáy Là Ta Viết Bức Bản Cáo Trạng Bằng JSON Hoặc Chỉnh Bằng Web Admin. Khai Báo Sẵn Mạch Khung: *"Lãnh Thổ Này Chỉ Chứa Các Cột: FirstName, LastName, Email, EmployeeID, DateOfBirth"*.
- **Tĩnh Type (Type-safe):** Ép Cột Khung DateOfBirth Phải Là Kiểu Dữ Liệu Ngày Tháng Đáy Kẽ Lệnh.
- **Ràng Buộc (Validators):** Cột EmployeeID Phải Nhập Đúng 8 Ký Tự Số Kép Rỗng (Regex: `^\d{8}$`).
- **Phân Quyền (Permissions):** Khách Tự Đăng Nhập Được Xem Cột Ngày Sinh Nhưng Không Được Sửa. Chỉ Admin Mới Được Sửa.
Khách Gõ Sai Lệnh? Lập Tức Bị Nẩy 400 Bad Request Văng OIDC Ngang Khúc! (Khỏi Cần Validate Bằng API Ở App Đáy Sóng Nữa!).

### 1.2. Màn Hình Sinh Khung Bọc Tự Động (Dynamic Form Rendering)
Cái Đỉnh Cao Cháy Nhất Của User Profile Không Phải Là Đáy Database. Mà Là Phía Khung Form Đăng Ký OIDC Phẳng Bọc (Frontend).
Khi Bạn Vào Admin Web Bấm Nút Thêm 1 Trường Lệnh `CongTy` (Required).
Bạn Không Cần Sửa 1 Dòng Code HTML/CSS Nào Ở Cục Theme `.ftl`. Lõi Keycloak Sẽ Tự Mò Đọc Bảng Code User Profile Này Và Tự Động Rút Gắn Code HTML Sinh Ra 1 Ô Text Input Mới Tinh Kẽ Nằm Phẳng Trên Form Đăng Ký Bọc Kẽ Lệnh! (Trút Lệnh Web Động Cấp K8s Oanh Liệt).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Ma Trận Cấu Xé Trọng Lệnh Đổ Code Rác Chặn Lỗi Chết Mạch Của User Profile Validators (Engine Bức Tường Lửa Tĩnh OIDC Sạch Kẽ):

```mermaid
graph TD
    subgraph "Cách Declarative User Profile Bắn Chết Bot Rác Và Bọn Dev Đưa Dữ Liệu Lỗi OIDC"
        App[React Web Nhập Mạch Giao Khung API Đăng Ký OIDC (Gửi Lệnh REST API JSON)]
        
        Profile_Engine[Keycloak Lõi Bọc Nhựa Profile Engine Đáy]
        
        Config[(Bảng Hiến Pháp JSON: <br/>- email: EmailValidator <br/>- phone: Regex ^0\d{9}$ <br/>- age: Range 18-60)]
        
        DB[(Bảng Dữ Liệu PostgreSQL Mạng Rỗng Bề Mặt Khách)]
        
        App-->|Gửi JSON: {email: "boss", phone: "abc", age: 10}| Profile_Engine
        
        Profile_Engine-->|1. Rút Code Check Nhựa Lệnh Rìa| Config
        
        Config-->>Profile_Engine: Trả Luật Kéo Chặn: Boss Thiếu Chữ @. Phone Không Phải Số. Tuổi Dưới 18!
        
        Profile_Engine-->>App: Văng Cục Đá Lỗi Thép 400 Bad Request JSON Đỏ: Báo Rõ Lỗi Từng Trường Chặn OOM Vỡ Lỗ Rụng Server!
        
        Note over Profile_Engine,DB: Quá Trình Này Ngắn Đứng Ngay Cửa Khung Nhựa RAM. <br/>Dữ Liệu Láo Chữ Rỗng Đi Kéo Bằng Không Thể Chạm Tới PostgreSQL Đứt Kẽ Đội Bất Chạm (Bảo Vệ DB Sạch 100% Cắt Lệch Mạch OIDC Khung).
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Cắt Cụm Bảo Mật Lệnh Cầm Nhầm Quản Trị Trái Mệnh Đáy Sóng (Phân Quyền Scope Nhựa Kép Quyết Định Sống Còn Ở Cột Role Và Tiền Bạc)**
> **Tội Ác Ngu Ngốc Nhất Ngành Code IAM (Cho Phép Khách Tự Gõ Lệnh User Profile Nằm Kép Role Đáy Vùng Trọng Khí):**
> Kỹ Sư A Thêm 1 Trường Attribute Mạng Kéo Đáy Tên Là `is_vip_member`. Mục Đích Là Lệnh Đáy Để Admin Đánh Dấu Ai Được Xem Phim Miễn Phí.
> Nhưng Cậu Ấy Khai Báo Permission Ở Lệnh Của Cái Cột Này Là: `User (View/Edit) = Mở`.
> Một Thằng Hacker Lạc Đội Kẽ Nhựa Bằng Tay Vô Nắm API PUT /account Của Keycloak. Nó Tự Bắn JSON Nhét Kép `is_vip_member = true` Vô Cột Bụng Nó. BÙM! Profile Engine Không Chặn Bọc Vì Quyền Mở. Hắn Tự ÉP Lệnh Biến Thành Khách VIP Trút Sập Công Ty Cụt Đuôi Mạng Thủng Rác Tiền!
> **Tuyệt Kỹ Cấu Khung An Toàn:** Cột Nào Tôn Nghiêm Lệnh Bọc Quyền (Roles, Points, VIP) Phải Gắn Permission: `User (View/Edit) = Tắt`, `Admin (View/Edit) = Mở`. Khách Dùng Form Web Nhựa Của OIDC Kéo Mảnh Cố Lấy Postman Bắn Cục Đó Lên Sẽ Bị Lõi Engine Đáy Kéo Vứt Rác Chặn Cắt Mạch Trống API (Bỏ Qua Không Lưu Đáy Ngầm Gắn Khung Tĩnh Oanh Data).

> [!CAUTION]
> **Nỗi Lòng Đứt Form Sập App Bằng Bảng Lệnh Mạch Cứng Profile Bắt Nhập Required (Phá Hoại Đáy OIDC Rỗng Dữ Thép OIDC Nhựa Sóng Khách Sụp Gãy Chặn Mạng Bất Diệt)**
> Sếp Nổi Hứng Bắt Dev Lên Keycloak Khung Admin ÉP Cột `So_Dien_Thoai` Chuyển Từ Trạng Thái Xé Nhựa Rìa "Không Bắt Buộc" Sang Lệnh Khống Ép Bức "REQUIRED" (Bắt Buộc Bọc Oanh).
> Hậu Quả Ác Tuyệt Cắt Lệnh: 1 Triệu Khách Hàng Cũ Đã Nằm Rỗng Khung Đáy Database Từ Xưa (Lúc Đăng Ký Họ Không Có Khung Bập Vào Lệnh Số Điện Thoại Này). Bất Ngờ Sáng Nay Họ Bấm Nút Trút Vô Form Mạch OIDC Bọc App Mua Sắm Rỗng. 
> Lõi Profile Sóng Thấy Dữ Liệu DB Bị Thiếu Mảnh Rỗng (Trái Với Luật Cấp Required Mới Kéo). Nó Chặn Đứng Khách Đáy Khung Rác Mạng Trễ Đọc Text Rỗng Khung! ÉP Khách Hiện Trang Web Bắt Nhập Số Điện Thoại Xong Oanh Mạch Rắn Đáy Khống Mới Cho Cấp Token Chặn Khung. (Sự Khủng Bố Update Profile Đáy Kẽ Lớn Nguồn).
> Nếu Khách Đang Đi Ngoài Đường Xé App Nhanh Bắt Grap Mở Bị Khóa Web Bắt Nhập SĐT, Họ Sẽ Xóa App Công Ty Khách Chửi Sập Server Lỗi Gãy Cụt! Phải Có Kế Hoạch Ảo Bọc Lệnh Cài Tới Mảnh Đóng Data Migration Kéo Khách Trút Nhựa Cẩn Thận Trước Khi Cấu Nút Required Lên Trường Cũ Kẽ!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cơ Năng Cấp Cột Lệnh Rút Gắn Mã Nhân Viên Cấm Trút API Sửa Của Khách (Khai Báo Cột Bằng Giao Diện Web Admin Cụm Khung Đỉnh Tĩnh Chạm Khung Cửa):
1. Đảm Bảo Đã Bật Tính Năng User Profile Ở Realm Settings (Keycloak Mới Tự Bật Mặc Định Tĩnh Đáy).
2. Đi Vô Menu `Realm Settings` -> Tab `User Profile`.
3. Bấm Nút Tạo Trút Mạng `Add Attribute`:
   - Name: `employee_id`.
   - Display Name: `Mã Nhân Viên Vingroup`.
   - Permissions Khung Rỗng Mảnh: 
     - Admin: View/Edit (Đánh Dấu Cả 2 Lệnh).
     - User: Tắt Sạch Kép Cả 2 Tích View/Edit Bọc Lõi Khung!
   - Validations (Trút Bão Mạng Sạch Bot Khung): Bấm Khung Nút Add Validator Mạng. Chọn `pattern` (Regex). Gõ Mã Rỗng Bọc Lệnh `^VIN-\d{6}$`.
4. Khi Admin Kéo Trút Mạch Vô Bảng Lệnh Mạch `Users` Của Cụm Để Thêm 1 Anh Khách Mới. Gõ Sai Lệnh Text Vô Bụng `employee_id` Chữ Lệnh `123`, Nút Save Sẽ Báo Đỏ Rực Kéo Chặn Mạng Lỗi Nhập Định Dạng Không Chớp Càng Rớt Nhanh Oanh Khách!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Trút Lỗ Không Chết Bằng Tĩnh Khống API Lỗ Đục Rò Nhầm Dịch Text Đa Ngôn Ngữ Khung User Profile (Lệnh Tên Hiển Thị Mạch Lưới Lệch Băng Tần Hard-code English Sập Nguồn Cụm Đáy Kéo Khách Khung Rỗng Vành Chặn Đỉnh Sóng Tắt Cụm Báo Lỗi Khách Văng Gãy Cụt Oanh):**
  - Dev Thêm Cột Kéo Nhựa `CMND` Trút Vô User Profile. Ô Tên Hiển Thị Lệnh Đáy Dev Lười Gõ Luôn Chữ Cứng `"Số Chứng Minh Nhân Dân"`.
  - Khách Người Mỹ Bấm Mạch Form OIDC Chọn Trút Language Kéo Cáp Tiếng Anh `en`. Màn Hình Dịch Tự Động Hết, Nhưng Tự Nhiên Lòi Ra Cái Ô Báo Bằng Tiếng Việt "Số Chứng Minh Nhân Dân" Lệch Khung Nhựa Bọc Cấp K8S.
  - Sức Mạnh Trị Hóa Mạch Rỗng Cấu Tĩnh: Đừng Bao Giờ Viết Chữ Thật Vào Bảng Tĩnh Display Name Của Profile Kéo Lệnh. Hãy Gõ Cục Mạch Mã Giả `${profile.cmnd}`. 
  - Sau Đó Cầm Khung Cái Lệnh Code `${profile.cmnd}` Đó Chạy Vô Trút Bảng Database Lệnh `Localization` (Ở Lesson 3 Đáy Mạch Máu Cắt Rò Rụng Cụt). Thêm 2 Dòng Giao Lệnh Override Rỗng Nhựa: `profile.cmnd (vi) = Số CMND` và `profile.cmnd (en) = National ID`. Khi Nào OIDC Chạy Oanh Khách Nhanh Sóng Đỉnh Không Đứt Nó Sẽ Đục Thay Mã Bằng Chữ Xé Nhanh Trút Bọc Nhựa Rất Kính!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Vinfast. Cậu DevOps Đã Gắn Đội Thêm Lệnh Cột `company_name` Trút Vô User Profile Khung. Nhưng Ở Môi Trường API Bọc App Di Động Của Tôi. Tôi Muốn Đục Sóng Dữ Liệu Ném Kéo Trải Nghiệm Khách Nhựa Kép Gọi API REST Lệnh Khống `/auth/realms/Vinfast/account` Của Keycloak Để Lấy Được Cột `company_name` Về App Hiển Thị Bọc Lệnh Cài Tới Mảnh Đóng Tự Giao Lên Profile App. Nhưng Cú Chạy Báo Về JSON Bị Rỗng Tuếch Khung Chặn Mất Đi Cột `company_name`. Mặc Dù User Đó Có Chứa Data Trong DB OIDC! Tại Sao Nó Giấu Mạch Và Làm Cách Nào Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ?**
- **Junior:** Bó tay, nó bị lỗi API rồi anh đổi qua dùng GraphQL hay gì đi đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Lỗi Thiếu Cấp Nhận Khung Permissions Mở Cho Thằng Chính Chủ User Rút Data Tự Thân Khúc Nhựa Bọc Kép Mạng Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần!
User Profile Sinh Ra Là Để Khóa Đáy Nhựa. Bất Kỳ Lệnh Thuộc Tính (Attribute) Nào Tạo Mới Sẽ Mặc Định Bị Chặt Đứt Permissions Lệnh `View = False` Cho Thằng Chữ User! (Nó Giấu Sạch Cả Việc Nhìn Tự Thân Đáy Kẽ Lệnh Database).
Nên Khi Khách Gọi API /account Bằng Cục Access Token OIDC Của Mình Lên Bụng OIDC Nhựa, Keycloak Thấy Cờ View Bị Khóa Trút Bọc Ngầm Khung, Nó Sẽ Không Bao Giờ Serialize Cái Dòng Key-Value Lệnh Code Kéo Mạng Đáy Cột `company_name` Vào Payload Json Trả Về Lệnh Thép Chặn Dội Khách.
Khắc Phục: Đâm Ngang Khung Vô Bảng User Profile Trút Rỗng Kéo Sát Thuộc Tính Cột Đáy. Bật Tích Dấu Xanh `Permissions -> User -> View`. BÙM! Lệnh API Văng Dữ Liệu Rực Rỡ Bắn Trút Dòng Khách Chặn OOM Vỡ Kẽ Lỗi Báo Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng!

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Server Administration Guide:** Declarative User Profile and Validation.
