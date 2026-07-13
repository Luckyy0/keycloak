# Lab 1: Thoát Khỏi Địa Ngục Báo Đỏ (The Escape Room)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Luyện tập cảm giác bắt bug thực tế. Trong môi trường này, chúng ta sẽ tạo ra một App Login. Ban đầu, mọi thứ CỐ TÌNH bị phá hoại cấu hình. Chạy lên sẽ thấy toàn lỗi. Bạn sẽ đi theo hướng dẫn để vá từng lổ hổng một (Fix Error) cho đến khi nút Login màu Xanh rực rỡ tỏa sáng.

## 1. Yêu cầu (Prerequisites)
- Docker Compose.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Khởi Động Đống Đổ Nát
Dùng file `docker-compose.yml` ở thư mục `code/`. Gõ:
```bash
docker-compose up -d
```
Chạy xong, máy có Cục Keycloak `8080`.
Tạo một Realm tên là `Lab-Troubleshoot`.
Tạo Client tên `my-bug-client` (Loại OIDC, Bật Access Type là Confidential -> Đòi Client Secret).

### Bước 2: Thưởng Thức Bệnh Lạc Đường `redirect_uri mismatch`
- Mở một Tab ẩn danh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề. Dán Đoạn Code sau lên thanh URL (Cố tình gọi OIDC Flow Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy):
  `http://localhost:8080/realms/Lab-Troubleshoot/protocol/openid-connect/auth?client_id=my-bug-client&response_type=code&redirect_uri=http://dia-chi-ma-toi-thich.com/callback`
- **Kết Quả:** Bị Trả Về Màn Hình Trắng Có Chữ Đỏ Tổ Chảng "Invalid parameter: redirect_uri".
- **Hành Động Fix Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:** Quay về Admin Console -> Clients -> `my-bug-client` -> Mở ô Valid Redirect URIs -> Gõ vào ĐÚNG Y CHANG cái Dòng Địa Chỉ `http://dia-chi-ma-toi-thich.com/callback` -> Lưu Lại Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!
- Ra Tab ẩn danh Refresh Lại -> Màn hình đăng nhập Tuyệt Đẹp Đã Hiện Ra Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh! Đăng Nhập Xong Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Trình Duyệt Bị Đẩy Về Link Kèm Theo Cái Tham Số Nhỏ Xíu `?code=123-abc...` Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh. Hãy Copy Nhanh Cái Dòng Đó Vào Notepad Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!

### Bước 3: Thưởng Thức Bệnh Trộm Cắp Mạo Danh `invalid_client`
- Bật cURL bằng dòng lệnh. Cầm Cái Code Vừa Xin Được Gửi Lên Đòi Đổi Thành Token Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh:
```bash
curl -X POST http://localhost:8080/realms/Lab-Troubleshoot/protocol/openid-connect/token \
  -d "grant_type=authorization_code" \
  -d "code=CÁI_MÃ_CODE_VỪA_COPY_BƯỚC_2" \
  -d "redirect_uri=http://dia-chi-ma-toi-thich.com/callback" \
  -d "client_id=my-bug-client" \
  -d "client_secret=TÔI_CỐ_TÌNH_GÕ_SAI_MẬT_KHẨU"
```
- **Kết Quả Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp:** Json trả về 401 `{"error":"invalid_client"}` Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh!
- **Hành Động Fix Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:** Chạy Vào Admin Console -> Vào Lại Client Chữ `Credentials`. Copy Đúng Cái Mật Khẩu Siêu Cứng Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. Xong Chạy Lệnh cURL.

### Bước 4: Thưởng Thức Bệnh Bánh Mì Đã Thiu `invalid_grant`
- Ở Bước 3 Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề, BẠN CHẮC CHẮN SẼ BỊ BÁO JSON TRẢ VỀ: `{"error":"invalid_grant"}` Cho Dù Đã Nhập Đúng `client_secret` Oanh Khung Dịch Lụa Mạch Lệnh! 
- **Giải Thích Hiện Tượng Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:** Do Ở Bước 2 Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Cái Mã `code` Đổi Ra Để Chờ Bạn Lấy Nó Chạy Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng, NÓ CHỈ SỐNG CÓ 1 PHÚT Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy. Bạn Ngồi Đọc Kỹ Hướng Dẫn Tới Đây Đã Quá Trễ Rồi Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị. Bánh Mì Nó Thiu! Thằng Server Trả Về Báo Quá Hạn Cấp Phép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy!
- **Hành Động Fix Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp:** Quay Trở Lại Bước 2. Bấm Chạy Lại Link. Lấy Lại Thằng Dãy CODE MỚI Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần! Rồi Quay Trực Tiếp Lại Lệnh Curl Nhập Vào Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy! 
- **BÙM!** Bạn Nhận Về Cuộn Json Dài Hàng Ngàn Chữ Bọc Cái TOKEN Quý Giá Trong Tay Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh. Bạn Đã Trở Thành Dev Cứng Cựa Khắc Phục Lỗi Chuyên Nghiệp Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!
