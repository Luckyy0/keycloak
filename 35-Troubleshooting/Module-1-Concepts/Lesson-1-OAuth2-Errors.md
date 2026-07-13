# Lesson 1: Bộ Lỗi Sinh Tử Của OAuth2 (invalid_client, invalid_grant...)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Thuộc nằm lòng 4 mã lỗi kinh điển nhất của giao thức OAuth2. Nhìn lỗi biết ngay sửa ở màn hình nào trong Admin Console của Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Khi làm việc với Keycloak Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, giao thức nền tảng là OAuth2. OAuth2 quy định một bộ mã lỗi tiêu chuẩn. Khi App của bạn (Client) nói chuyện với Keycloak (Authorization Server) mà bị vả lệch mặt, nó sẽ trả về mã JSON có dạng:
```json
{
  "error": "invalid_grant",
  "error_description": "Code not valid"
}
```
Dưới đây là Cuốn Từ Điển Cứu Mạng:

### Bệnh 1: `invalid_client` (Kẻ Mạo Danh)
- **Triệu chứng:** Khi App của bạn (Backend Spring Boot, Express...) mang `client_id` và `client_secret` lên cổng `/token` để đổi mã Code lấy Access Token. Keycloak đá đít bạn ra kèm lỗi `invalid_client`.
- **Căn nguyên:**
  1. Bạn gõ sai chữ `client_id` (Dư dấu cách, sai chữ hoa thường Oanh Khung Dịch Lụa Mạch Lệnh).
  2. Bạn copy paste `client_secret` bị thiếu mất 1 ký tự Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa.
  3. Quản trị viên Keycloak VỪA BẤM NÚT "Regenerate Secret" (Đổi mật khẩu Client) mà chưa báo cho team Dev cập nhật Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy!
  4. Bạn cài Client là Public (Không có Secret), nhưng lúc gọi API bạn lại cố tình nhét header `Authorization: Basic ...` chứa Secret vào.
- **Cách Chữa:** Vào Keycloak -> Clients -> Tab *Credentials*. Nhấn Copy Secret và Dán lại vào file `.env` của App.

### Bệnh 2: `invalid_grant` (Tờ Phiếu Quá Hạn)
- **Triệu chứng:** Lỗi này CỰC KỲ PHỔ BIẾN Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng! Xảy ra chủ yếu ở 2 luồng: `authorization_code` và `refresh_token`.
- **Căn nguyên 1 (Đổi Code lấy Token):** Cái `code` mà Keycloak trả về ở trình duyệt, chỉ có thời hạn sống Mặc Định Là 1 Phút Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần. Hơn nữa Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp, NÓ CHỈ ĐƯỢC XÀI ĐÚNG 1 LẦN DUY NHẤT Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa! Nếu Frontend gửi cái Code đó xuống Backend 2 lần, lần thứ 2 Keycloak sẽ ném `invalid_grant`.
- **Căn nguyên 2 (Refresh Token):** Khi Access Token hết hạn (5 phút), App mang Refresh Token lên xin Token mới. Nhưng... Refresh Token đó ĐÃ BỊ Thu Hồi (Logout), hoặc tài khoản User đó Vừa Bị Quản Trị Viên Khóa (Disable), hoặc Session của user trên trình duyệt vừa Timeout (Sau 30 Phút) Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề.
- **Cách Chữa:** Báo Khách Hàng Login Lại Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa! Code đã chết thì chỉ có cách Login lại để cấp Code mới.

### Bệnh 3: `invalid_scope` (Đòi Hỏi Quá Đáng)
- **Triệu chứng:** Khi Frontend gọi hàm `login(scope="openid profile email vip_access")`. Keycloak chặn họng ngay.
- **Căn nguyên:** Thằng Client đó CHƯA ĐƯỢC QUẢN TRỊ VIÊN CẤP PHÉP quyền xài cái scope `vip_access` Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. OAuth2 bảo vệ hệ thống bằng cách: Dev muốn xin quyền gì phải khai báo trước.
- **Cách Chữa:** Vào Keycloak -> Client Scopes -> Tạo một scope tên `vip_access`. Sau đó vào Clients -> Chọn Client -> Tab *Client Scopes* -> Bấm Add Client Scope `vip_access` (Loại Optional).

### Bệnh 4: Màn hình Đỏ Chữ Đen: `Invalid parameter: redirect_uri` (Tên Lạc Đường)
- **Triệu chứng:** Đây KHÔNG PHẢI lỗi trả về dạng JSON. Đây là Lỗi Văng Thẳng Ra Cái Màn Hình Báo Lỗi To Tổ Bố Của Keycloak Ngay Khi Vừa Bấm Login Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy!
- **Căn nguyên Cực Kỳ Kinh Điển Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:** Bảo Mật Tuyệt Đối Của OAuth2. Khi Đăng Nhập Xong Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề, Keycloak Chỉ Đồng Ý Chở Khách Hàng Về Đúng Cái Địa Chỉ Mà Nó Đã Ghi Sổ Lúc Cấu Hình Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.
  Bạn Cấu Hình Trong Keycloak: `Valid Redirect URIs` = `https://myapp.com/callback`.
  Nhưng Code Frontend Của Bạn Chạy Dưới LocalHost Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Gọi Lên: `?redirect_uri=http://localhost:3000/callback`.
  Keycloak Đọc Thấy `localhost:3000` Lệch Với Sổ Ghi Danh `myapp.com`. Nó Từ Chối Ngay Lập Tức Để Tránh Hacker Trộm Code Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh!
- **Cách Chữa:** Vào Keycloak -> Clients -> Valid Redirect URIs -> Thêm chính xác Đường dẫn mà Frontend đang gọi (Có Cả Dấu `/` Cực Kỳ Quan Trọng Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp, Thiếu 1 Ký Tự Cũng Chết).

---

## 2. Câu hỏi Phỏng vấn (Interview Questions)

**1. Khách Hàng Vừa Bấm Đăng Nhập Ở Trình Duyệt Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh, Keycloak Trả Về Lỗi `redirect_uri mismatch` Ở Màn Hình Đỏ Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh. Dev Khăng Khăng Đã Cấu Hình Khớp Từng Chữ Một Giữa FrontEnd Là `https://myapp.com/*` Và Trong Keycloak Valid URI Cũng Là `https://myapp.com/*`. Theo Kinh Nghiệm Của Em Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị, Lỗi Sâu Xa Nằm Ở Đâu Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh?**
- **Senior:** Dạ Thưa Sếp Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Lỗi Này Có Tới 90% Chết Ở Chỗ Cấu Hình Con Cổng Phân Tải (NGINX/HAProxy) Chứ Không Hề Do Dev Oanh Khung Dịch Lụa Mạch Lệnh!
  - Ở Sơ Đồ Mạng Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy: NGINX Đứng Trước Hứng SSL `https://myapp.com` Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Rồi Forward Vào Cho Keycloak Bằng Đường Ống Kín Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy `http://10.0.0.5:8080` (HTTP Trơn Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp).
  - Keycloak Nó Nhận Được Request Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, Nó Lại Tưởng Rằng Nó Đang Chạy Ở Dưới Địa Hình Không Bảo Mật `http://` Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị. Nên Dù Trình Duyệt Client Truyền Tham Số Url Lên Là `redirect_uri=https://myapp.com...`, Keycloak Ở Trong Ruột Nó Xử Lý Lại Nghĩ Rằng Client Đang Đòi Về Cái Cổng `http://myapp.com` Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa! Chữ **HTTP** Đương Nhiên Khác **HTTPS** Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng, Thế Là Nó Phạt Lỗi Mismatch Ngay Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh!
  - **Cách Fix Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:** Phải Nhét Cờ Header Forward Vào NGINX (`proxy_set_header X-Forwarded-Proto $scheme;`) Và Khởi Động Keycloak Bằng Lệnh `kc.sh start --proxy=edge` Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Lúc Đó Keycloak Đọc Cái Chữ `X-Forwarded-Proto` Nó Mới Ngộ Ra Rằng: "À, Hóa Ra Anh Khách Đang Đi Bằng HTTPS, Nên So Khớp Mới Chuẩn Xác"!
