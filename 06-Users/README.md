# Chương 06: Đội Quân Vô Danh (User Management & Lifecycle)

> [!NOTE]
> Chào mừng đến với Chương 6. Nếu Realm là Vùng Lãnh thổ Khép kín, thì User chính là Những Người Dân Đen sống trong Lãnh thổ đó. Quản trị hàng triệu Người dùng (Identities) không đơn giản chỉ là lưu trữ Username và Password xuống Bảng Cơ sở Dữ liệu. Bạn phải Cầm Cương Quản lý Toàn Bộ Vòng đời (Lifecycle) của họ: Từ lúc họ mới Sinh ra (Đăng ký), Móc nối Xác thực (Email Verification), Cho đến việc Trừng Phạt (Required Actions) hoặc Khóa Mõm (Disable) nếu phát hiện rủi ro bảo mật.

## Mục tiêu của chương
- Nắm bắt Cốt Lõi Vòng đời Người Dùng: Bật/Tắt Trạng thái (Enabled/Disabled) và Nghệ thuật Bơm Các Hình Phạt Ép Buộc (Required Actions).
- Chống Cuộc Tấn Công Bơm Rác Spam Đăng Ký: Nghệ thuật Mở Cổng Đăng Ký Tự Do (Self-Registration) mà không bị Hacker Bơm Hàng Triệu User Rác Làm Nổ Tung Database.
- Mở Rộng Dữ Liệu Bụng Người Dùng (User Profile & Attributes): Bức Tường Lửa Chống Dữ liệu Láo. Vượt ra khỏi Username/Email Để Lưu Trữ Mã Nhân Viên, Sở Thích.
- Rèn Luyện Kỷ Luật Mật Khẩu (Password Policies): Ép Khách hàng phải Đặt Mật Khẩu Có Ký Tự Đặc Biệt, hoặc Bất tử Chặn Tuyệt Đối Việc Đặt Lại Mật Khẩu Đã Dùng Trong Quá Khứ.

## Cấu trúc bài học
Chương này đi sâu vào Bảng Cấu Hình Admin của Nhóm Quản Trị User:

- **Nhóm 1: Vòng Đời & Biên Giới Đăng Ký**
  - `Lesson-1-User-Lifecycle.md`: Quyền Năng Tước Sinh Mệnh (Enabled/Disabled) và Sợi Dây Trói Ép Buộc Hành Động (Required Actions).
  - `Lesson-2-Registration.md`: Công Tắc Bật Mở Cửa Đăng Ký Tự Do. Tội Ác Ngập Lụt Băng Thông Nếu Quên Bật Chặn Cửa reCAPTCHA.
  - `Lesson-3-Email-Verification.md`: Bắt Buộc Xác Thực Email - Bức Tường Tĩnh Không Cho Phép Bot Rác Thâm Nhập Lấy Token.
- **Nhóm 2: Hồ Sơ Dữ Liệu & Hàng Rào Mật Khẩu**
  - `Lesson-4-User-Profile.md`: Kiến Trúc Mới Khai Tử Lỗi Dữ Liệu Xấu - User Profile (Rào Cản Type Tuyệt Đối Bằng Tĩnh).
  - `Lesson-5-Attributes.md`: Nghệ Thuật Lấp Đầy Chỗ Trống Key-Value Và Lỗ Hổng Hiển Thị Lộ Thông Tin Mật Ở Token.
  - `Lesson-6-Credentials.md`: Quản Trị Trung Tâm Mật Khẩu, Reset Đáy Cấu Hình Của Khách Hàng.
  - `Lesson-7-Password-Policies.md`: Luật Lệ Sinh Tử Chặn Dò Pass (Regex Hashing Algorithm, Băm Password Trọng Tĩnh).

## Hướng dẫn thực hành (Labs)
- Bài Lab cuối chương sẽ Dẫn Bạn Cầm Cờ Dựng Lên 1 Cụm Khung Đăng Ký OIDC Có Bật Tính Năng Tự Đăng Ký. Bạn Sẽ Cấu Hình Bắt Buộc Cắm Email Verification (Sử dụng Công Cụ Giả Lập MailHog Phẳng Để Bắt Bọc Thư Gửi Đáy) Bằng Đóng Kịch Bản Password Policy Cực Gắt Gắn Khách!
