# Chapter 12: Authentication - Thực Hành Chống Phá Khóa & Triển Khai Không Mật Khẩu

> [!NOTE]
> Bài thực hành này sẽ đưa bạn vào vị trí của một Kỹ sư Bảo mật OIDC. Chúng ta sẽ cùng nhau khóa chặt hệ thống trước những đợt tấn công dò mật khẩu tự động (Brute-force) bằng cách bật các lớp bảo vệ của Lãnh chúa. Cuối cùng, chúng ta sẽ mở ra Kỷ nguyên Không mật khẩu (Passkeys) trải nghiệm thực tế ngay trên máy tính của bạn.

## Bài tập 1: Dựng Tường Lửa Chặn Đứng Cỗ Máy Quét Mật Khẩu (Brute-Force Protection)

Kịch bản: Hacker có một danh sách 1 triệu mật khẩu rò rỉ. Hắn dùng Bot bắn liên tục API POST `/token` để đoán mật khẩu của Admin. Bạn sẽ dùng Tấm khiên Brute-Force để trừng phạt bằng cách khóa tài khoản 5 phút nếu hắn gõ sai 3 lần liên tiếp.

1. **Chuẩn bị môi trường:**
   - Đảm bảo `docker-compose` đã chạy (Không cần bật cờ gì thêm vì đây là tính năng cốt lõi).
   - Truy cập `http://localhost:8080`, đăng nhập bằng `admin` / `admin`.
   - Vào `Realm settings` (Cột menu trái).

2. **Kích hoạt Kỷ luật thép (Brute-Force Detection):**
   - Chuyển sang Tab `Security Defenses` -> Tab con `Brute Force Detection`.
   - Gạt nút `Enabled` sang `ON`.
   - Cấu hình các thông số trừng phạt như sau:
     - **Max Login Failures:** `3` (Nhập sai 3 lần là bị máy chém gõ).
     - **Wait Increment:** `5` Minutes (Mỗi lần tiếp tục sai sẽ bị cộng dồn thời gian phạt).
     - **Max Wait:** `15` Minutes (Phạt tối đa 15 phút, sau đó đếm lại).
     - **Failure Reset Time:** `1` Hours (Nếu ngừng phá trong 1 tiếng, máy chém sẽ tha thứ đếm lại từ đầu).
   - Nhấn **Save**. Tấm khiên đã được dựng!

3. **Kiểm thử đóng giả Hacker:**
   - Mở 1 tab ẩn danh mới, vào `http://localhost:8080/realms/master/account`.
   - Cố tình đăng nhập bằng User: `admin` nhưng Pass sai: `123456`.
   - Làm lại đúng 3 lần!
   - Ở lần thứ 4, kể cả bạn nhập **Pass Đúng** là `admin`, Keycloak vẫn văng màn hình lỗi: `Invalid username or password` (Thực chất tài khoản đã bị khóa cứng tạm thời). Bật F12 Network bạn sẽ thấy lỗi `Account is disabled`. Hacker đã bị chặn đứng hoàn toàn!

4. **Giải cứu con tin (Mở khóa thủ công):**
   - Về lại màn hình Admin Console (ở Tab cũ đang còn Session).
   - Vào mục `Users` -> Tìm User `admin` -> Sang Tab `Details`.
   - Nhấn nút `Unlock users` ở ngay góc trên bên phải màn hình để ân xá cho tài khoản.

## Bài tập 2: Tự Động Hóa Ép Buộc Toàn Bộ Khách Hàng Quét Mã Google Authenticator (Required Actions OTP)

Kịch bản: Giám đốc bảo mật của Công ty Fintech yêu cầu: "Bắt đầu từ hôm nay, bất kỳ ai login vào hệ thống cũng phải tự cài app Authenticator trên điện thoại và quét mã QR. Nếu không thì văng ra ngoài".

1. **Kích hoạt vũ khí ép buộc OTP:**
   - Đứng ở Realm `master` (Hoặc tạo Realm mới).
   - Vào Menu `Authentication` -> Tab `Required actions`.
   - Tại dòng `Configure OTP`, bật công tắc Cột `Default` thành `ON` (Tức là mặc định áp dụng ép buộc này cho toàn bộ user mới và cũ).

2. **Kiểm thử luồng Login bị chặn đứng đòi mã:**
   - Mở tab ẩn danh mới, vào `http://localhost:8080/realms/master/account`.
   - Tạo 1 User mới bất kỳ bằng tính năng `Register` (hoặc tạo dưới Admin rồi login vào bằng tài khoản đó).
   - Ngay sau khi nhập mật khẩu thành công. ĐÙNG! Keycloak không cho vào trang đích. Nó chặn màn hình lại và hiện ra 1 cái Mã QR Code chà bá. 
   - Nó yêu cầu: Cài đặt Google Authenticator/FreeOTP trên điện thoại -> Quét mã QR -> Nhập 6 số hiện trên điện thoại lên màn hình để xác nhận sở hữu. Cứ làm theo là được thả qua trạm gác.

3. **Gỡ bỏ thiết quân luật:**
   - Nhớ tắt Cột `Default` ở mục `Configure OTP` đi để tránh phiền phức cho các bài Lab sau nhé.

## Bài tập 3: Cảnh Giới Tối Thượng - Mở Khóa Keycloak Bằng Vân Tay/FaceID (Passkeys)

Kịch bản: Trải nghiệm cảm giác công nghệ tương lai. Đăng nhập hệ thống bằng cảm biến vân tay của chiếc Macbook hoặc Windows Hello mà không cần nhập một ký tự mật khẩu nào.

1. **Bật công tắc Kỷ nguyên Không mật khẩu:**
   - Vào Menu `Authentication` -> Tab `Required actions`.
   - Cuộn xuống dưới cùng tìm dòng `Webauthn Register Passwordless`. Nhấn icon ba chấm ở cuối dòng chọn `Enable`.

2. **Sửa luồng Browser Flow (Dành cho bản Keycloak 24 trở xuống, bản mới hơn có sẵn):**
   - Tab `Flows` -> Mở `Browser`.
   - Đảm bảo trong cây luồng có cục `WebAuthn Passwordless Authenticator` nằm ngang hàng với `Username Password Form`.
   - Chuyển cục WebAuthn Passwordless thành `Alternative` (Để ưu tiên quét Vân tay trước, nếu không có mới cho điền form Pass cũ).

3. **Khách hàng tự đăng ký Vân tay (Khai báo Passkeys):**
   - Đăng nhập vào trang `http://localhost:8080/realms/master/account` bằng User `admin`.
   - Menu bên trái -> `Account security` -> `Signing in`.
   - Tìm hộp `Passkeys` -> Bấm `Set up passkey`.
   - Trình duyệt (Chrome/Safari) sẽ nhảy lên Pop-up do Hệ điều hành bắn ra: Đòi chạm ngón tay vào cảm biến vân tay (Macbook) hoặc Windows Hello. Quét cái rẹt là xong!
   - Passkey đã được lưu. Giờ đăng xuất (`Sign out`) khỏi Keycloak.

4. **Trải nghiệm đăng nhập Ma thuật:**
   - Trở lại form Login `http://localhost:8080/realms/master/account`.
   - Bạn sẽ thấy có 1 nút mới toanh xuất hiện dưới ô Mật khẩu: **`Sign in with Passkey`**.
   - Bấm vào đó. Hoặc chỉ cần điền đúng chữ `admin` vào ô Username và bấm Enter.
   - Pop-up vân tay bật lên -> Chạm nhẹ ngón tay -> BÙM! Keycloak đá văng bạn vào trang chủ mà không thèm hỏi Password.

Chúc mừng bạn! Bạn đã hoàn thành Khóa đào tạo Level Core. Mọi lỗ hổng bảo mật chết người đã được vá, và trải nghiệm Authentication đã được đẩy lên tầm cực hạn!
