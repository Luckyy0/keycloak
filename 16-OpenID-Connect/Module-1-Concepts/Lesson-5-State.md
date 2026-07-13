# Lesson 5: Cờ Định Vị (Tham số State)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Ở bài 4 chúng ta đã nhắc sơ qua về Cờ State như một khiên chắn chống CSRF. Trong bài này, ta sẽ mổ xẻ sâu hơn sức mạnh THỨ HAI của Cờ State. Nó không chỉ là Khiên Chắn, nó còn là "Tấm Bản Đồ Định Vị" giúp App của bạn nhớ được User đang làm dở thao tác gì trước khi bị đá sang Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Cờ State Là Tấm Vé Gửi Đồ Khớp Lệnh
Hãy tưởng tượng kịch bản (UX - Trải nghiệm người dùng):
1. Khách hàng đang đọc bài viết: `http://myapp.com/bai-viet-id-88`.
2. Khách hàng thấy hay quá, kéo xuống gõ Comment. Hệ thống báo: "Bạn phải Login mới được Comment".
3. Khách bấm nút Login. Khách bị Văng sang màn hình Keycloak.
4. Khách Login xong, Keycloak văng Khách về lại App: `http://myapp.com/callback?code=abc`.
5. **Nỗi đau ở đây:** Cái App lúc này làm sao biết khách đang đọc bài số 88 để chuyển hướng (Redirect) họ về đúng bài đó? Chả lẽ lại ném khách về Trang Chủ (Home) bắt khách tự lặn lội tìm lại bài viết 88? User sẽ chửi rủa và xóa App ngay!

### 1.2. Mạch Oanh Giao Dịch OIDC Bọt Cắt Cờ State Giải Cứu
Cờ `state` sinh ra chính là để giải quyết Lỗ Hổng Não Cá Vàng này của Stateless HTTP.
- **Trước khi nhảy sang Keycloak:** App tạo ra 1 chuỗi JSON: `{"csrf": "mã_chống_hack", "returnUrl": "/bai-viet-id-88"}`.
- App mã hóa Base64 cái mảng JSON đó thành cục chuỗi: `ey...xyz`. Xong App gắn nó vào URL: `&state=ey...xyz`.
- **Sau khi Keycloak nhả về:** Keycloak dội lại đúng cục `&state=ey...xyz`. 
- App nhận được Code, bóc Base64 cục State ra. App reo lên: "À há! Thằng này vừa đọc bài 88, tao redirect nó về đúng chỗ cũ để nó Comment tiếp! Kèm theo check Mã Chống Hack CSRF Khớp Lệnh Cắt Khung!". Trải Nghiệm Mượt Mà Oanh Chóp Bọt Lụa Đỉnh!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Điều Hướng Cáp Rút Lụa State Lưu Vết Mạch Dịch Cũ:

```mermaid
sequenceDiagram
    participant App as ReactJS Frontend (Client)
    participant KC as Keycloak (Auth Server)

    Note over App: Khách Đang Mua Cục Hàng Số 9. Bấm Login.
    App->>App: Tạo State Bọc Data: Base64UrlEncode('url=/cart/9; csrf=X7y') => ST_ABC
    
    App->>KC: Bắn URL Nhử Mồi OIDC: GET /auth?...&state=ST_ABC
    KC-->>KC: Bóc Tách Form. Mời Khách Nhập Pass Băng Tần Mạch Lụa!
    
    KC-->>App: Trả Code Rác Đội URL Kèm State Cũ: Redirect /callback?code=123&state=ST_ABC
    
    Note over App: App So Khớp Mạch CSRF. Lấy Được Mạch Data Cũ!
    App->>App: Bóc Base64 ST_ABC -> Thấy 'url=/cart/9'.
    App->>App: Sau Khi Đổi Xong Access Token Bọc Thép, App Nhảy Router Dẫn Khách Về Đúng Giỏ Hàng! Khách Rất Hài Lòng Mạch Lệnh!
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Oanh Cáp Trọng Lực (Bảo Mật Lệnh Data Nhạy Cảm Trên Mạch State)**
> **Tội Ác Thiết Kế:** Bạn thấy State có thể lưu được Dữ liệu điều hướng. Bạn nổi máu nhét luôn Thông tin nhạy cảm vào đó để trượt lụa cho lẹ (Ví dụ: Số tiền đang mua, Mã giảm giá, Thậm chí Mật khẩu cũ). 
> `&state={"return": "cart", "discount": "VIP99", "secret": "abc"}` (Encode Base64).
> **Hậu Quả:** Cờ State Bay Chơi Vơi Trên Bề Mặt Thanh Trình Duyệt Front-Channel. Thằng Cướp Mạng Bắt URL, Decode Base64 Thấy Nguyên Lỗ Hổng Khung Dịch Lụa Lộ Giao Dịch Bí Mật Mạch Rỗng. Nó Lập Tức Chỉnh Sửa Hoặc Ăn Trộm!
> **Biện Pháp Sống Còn Lớp Trọng Lực OIDC Đáy Lụa:** Tham số State Chỉ Nên Chứa ID Rác Định Danh Ngắn (VD: `state_id_99`). Còn toàn bộ Dữ liệu thực (ReturnUrl, Data Giỏ Hàng) BẠN PHẢI CHÔN SÂU XUỐNG DƯỚI BỤNG MÁY CHỦ BẰNG SESSION HOẶC LOCALSTORAGE (Ứng với cái Key `state_id_99` đó). Khi Cáp Mạch Dội State Trở Về, Bạn Bốc Khóa Tĩnh Đó Xuống Kho Lấy Data Lên Lắp Ghép Lụa Tránh Bị Thấy Trắng Lệnh Kẽ Oanh Khung!

---

## 4. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Giao Thức Oauth2/OIDC. Nếu Một Ứng Dụng Kém Cỏi (App Khách Lệnh) Bỏ Qua Tham Số 'State' Hoặc Dùng Nó Cố Định 'State=123' Cho Mọi Khách Hàng. Theo Chuẩn BCP Mới Nhất Oanh Khung Dịch Lụa Mạch Lệnh Đáy DB, Keycloak Có Chấp Nhận Xả Access Token Lệnh Đổi Cho Khách Không, Hay Bắn Lỗi Oanh Rỗng Chóp Cắt Bọt Đứt Băng?**
- **Senior:** Dạ thưa sếp, Chỗ này Lại Tùy Thuộc Vào Bản Chất Của Lãnh Chúa Keycloak!
  - Về Mặt Lý Thuyết Chuẩn OAuth 2.0 Nguyên Bản (RFC 6749), Cờ `state` Được Khuyến Khích Dùng (RECOMMENDED), Chứ Cấu Trúc Khung Rỗng Chữ Tĩnh KHÔNG BẮT BUỘC ĐÓNG CHẶT. Nếu App Khách Lệnh Cũ Bỏ Quên, Một Số Phiên Bản Keycloak Cổ Vẫn Nhắm Mắt Cho Qua (Pass) Trút Code Oanh Lụa.
  - Tuy nhiên, Về Mặt Bảo Mật OIDC Đỉnh Cao (OAuth 2.1 Mạch Trọng) Và Khi Bạn Bật Cờ FAPI Oanh Mạng, NẾU KHÔNG CÓ `state` Sinh Khớp Chữ Ký Chống CSRF Rỗng Bọt, Máy Chủ Sẽ Lập Tức Từ Chối Đập Băng Lỗi Văng `HTTP 400 Invalid Request Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa!`. Do Đó, Viết Code FrontEnd Bắt Buộc 100% Phải Chèn Tham Số Này Để Sinh Tồn Tương Lai!

---

## 5. Tài liệu tham khảo (References)
- **RFC 6749:** Section 4.1.1 Authorization Request (state).
- **OWASP:** Cross-Site Request Forgery (CSRF).
