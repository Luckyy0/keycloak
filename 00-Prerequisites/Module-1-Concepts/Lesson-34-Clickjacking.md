# Lesson 34: Lỗ hổng Clickjacking (UI Redressing)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Lật tẩy ma thuật "Click ẩn" (Clickjacking) - Kỹ thuật thao túng Giao diện (UI Redressing). Cách Hacker đánh lừa thị giác người dùng bằng các Iframe tàng hình và nghệ thuật tự vệ của Máy chủ thông qua X-Frame-Options.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Clickjacking là gì?
Clickjacking (Cướp Click) là một đòn tấn công Đánh lừa Thị giác (Optical Illusion) cực kỳ bỉ ổi. Thay vì hack hệ thống hay tiêm mã độc, Hacker nhắm thẳng vào Kẽ hở của BỘ NÃO CON NGƯỜI (Giao diện hiển thị).
- **Mục đích:** Khiến Nạn nhân bấm vào một Nút (Button) bí mật, trong khi họ cứ đinh ninh rằng mình đang bấm vào một Nút vô hại khác.

### 1.2. Ma thuật của Iframe Tàng hình
Hacker xây dựng một trang web lừa đảo (`hacker.com`) chứa một cái nút to đùng: "BẤM VÀO ĐÂY ĐỂ NHẬN IPHONE 16 MIỄN PHÍ".
Sau đó, Hacker dùng thẻ `<iframe>` để nhúng cái Màn Hình Đăng Nhập (Login Page) của Ngân hàng BẠN lên đè lên cái web đó.
- Điểm mấu chốt: Hacker dùng CSS chỉnh độ trong suốt của cái Iframe Ngân hàng thành Vô hình (`opacity: 0`).
- Kết quả: Cái Nút "Chuyển Tiền" của Ngân hàng (Nằm trong Iframe tàng hình) ĐÈ CHÍNH XÁC LÊN TRÊN cái Nút "Nhận iPhone 16". Nạn nhân bấm nhận điện thoại, thực chất là bấm nút Chuyển Tiền.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Phân tầng trục Z (z-index) trong trình duyệt khi bị Clickjacking:

```mermaid
graph TD
    subgraph "Màn hình Trình duyệt (Tầm nhìn của Nạn nhân)"
        V[Nhìn thấy: Nút 'Nhận iPhone Miễn Phí']
    end
    
    subgraph "Tầng 1 (Lớp đáy - Trang của Hacker)"
        H[Trang hacker.com <br/> z-index: 1]
        Btn[Nút 'Nhận iPhone']
        H --- Btn
    end
    
    subgraph "Tầng 2 (Lớp nổi - Iframe Tàng Hình)"
        Iframe[Iframe load trang bank.com <br/> CSS: opacity: 0; z-index: 999;]
        TransBtn[Nút 'Chuyển Tiền 1 Tỷ' của Bank]
        Iframe --- TransBtn
    end
    
    V -.->|Ảo giác thị giác| Btn
    V -.->|Cú click vật lý thật đập vào| TransBtn
    
    Note over V,TransBtn: Con trỏ chuột của người dùng bị chặn lại bởi Tầng 2.<br/>Do nó Tàng hình, người dùng không biết mình vừa kích hoạt<br/>một lệnh nghiêm trọng trên trang web Ngân hàng.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Phòng thủ Cổ điển: X-Frame-Options (XFO)**
> Để chống Clickjacking, Hệ thống Identity (Keycloak) BẮT BUỘC phải cấm không cho các Web lạ Nhúng mình vào Iframe của họ.
> Header `X-Frame-Options` ra đời làm nhiệm vụ đó. Nó có 2 cờ chính:
> - `DENY`: Cấm nhúng tuyệt đối. Không một ai, kể cả chính tên miền đó được phép nhúng nó vào iframe. (Tuyệt vời cho các trang Landing page).
> - `SAMEORIGIN`: Chỉ cho phép nhúng nếu cái Trang Mẹ (Nơi chứa Iframe) CÙNG TÊN MIỀN với cái Iframe đó. (Phù hợp cho các hệ thống ERP nội bộ có xài Iframe).

> [!CAUTION]
> **Phòng thủ Hiện đại: CSP `frame-ancestors`**
> XFO là tiêu chuẩn cũ (Dù vẫn hoạt động tốt). Yếu điểm của nó là không hỗ trợ Whitelist (Danh sách trắng). Giả sử bạn muốn Bán hàng, bạn muốn cho phép 3 Đối tác được nhúng trang Thanh toán của bạn vào Iframe của họ (Còn lại cấm hết). XFO không làm được.
> **Thực hành chuẩn:** Sử dụng Content Security Policy (CSP). Header: 
> `Content-Security-Policy: frame-ancestors 'self' https://doitac1.com https://doitac2.com;`
> Lệnh này chặt đứt mọi trang web lạ nhúng bạn, ngoại trừ 2 đối tác tin cậy trên. *(Lưu ý: Nếu bạn cấu hình cả XFO và CSP, các Trình duyệt hiện đại sẽ Ưu Tiên CSP và Lờ Đi XFO).*

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Trong Keycloak, tính năng chống Clickjacking được BẬT MẶC ĐỊNH MỨC CAO NHẤT. Bạn có thể kiểm chứng tại:
- Menu `Realm Settings` -> Tab `Security Defenses`.
- Mặc định, `X-Frame-Options` được set là `SAMEORIGIN`.
- `Content-Security-Policy` mặc định có chứa chỉ thị `frame-ancestors 'self'`.

Nếu một ngày, team Frontend (Chạy ở `app.company.com`) khiếu nại rằng: "Anh ơi, cái tính năng OIDC Silent Check SSO của thư viện `keycloak-js` bị lỗi rồi. Trình duyệt báo lỗi Console màu đỏ chót chặn Iframe".
- **Cách fix:** Đừng dại dột mà Tắt CSP (Lỗ hổng chết người). Hãy vào Keycloak, sửa lại trường `frame-ancestors` thành:
`frame-ancestors 'self' https://app.company.com` (Cho phép Frontend nhúng Keycloak để check session ẩn).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Bypass X-Frame-Options qua Proxy:** Đôi khi, Hacker không nhúng trực tiếp trang của bạn. Hắn xây dựng một Reverse Proxy trung gian (Tên miền C). Hắn lấy Iframe tải trang C. Trang C tải trang Ngân hàng của bạn, nhưng trang C CHỦ ĐỘNG XÓA BỎ cái Header `X-Frame-Options` (Do trang C nằm dưới quyền Hacker).
  - **Phân tích:** Đòn này thất bại. Nếu trang C thay mặt Hacker kéo trang Ngân hàng về (Gọi là Server-Side Proxying), Máy chủ Ngân hàng sẽ coi trang C là 1 Client mới cứng. Cookie Session (Đăng nhập) của Nạn nhân nằm ở trên Trình duyệt, không hề nằm trên Máy chủ C. Trang C kéo màn hình Ngân hàng về sẽ chỉ thu được Màn Hình Bắt Đăng Nhập trống rỗng, không chứa Token của Nạn nhân. Đòn Clickjacking vô hiệu tác dụng. (Trừ khi có kèm CSRF).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Kỹ thuật "Frame-busting script" là gì? Tại sao Javascript không còn được khuyên dùng để chống Clickjacking nữa?**
- **Junior:** Là viết JS kiểm tra iframe để cản hack.
- **Senior:** Frame-busting là kỹ thuật Cổ Đại (Trước khi có HTTP Headers). Dev viết một đoạn JS vào thẻ `<head>`: `if (top !== self) top.location.href = self.location.href;` (Nếu tao bị nhúng trong iframe, tao ép cửa sổ Tầng trên cùng phải load lại trang của tao).
Tuy nhiên, Kỹ thuật này bị phá giải dễ dàng (Busted the buster). Hacker chỉ cần thêm thuộc tính `sandbox="allow-scripts"` vào cái Iframe lừa đảo, hoặc dùng HTML5 `sandbox` để CẤM đoạn mã JS bên trong Iframe thực hiện hành vi Chuyển Hướng Tầng Cao Nhất (Top navigation). Đoạn JS phòng thủ bị vô hiệu hóa hoàn toàn. BẮT BUỘC phải dùng HTTP Headers (XFO/CSP) vì nó được thực thi sâu ở lõi C/C++ của Browser, không thể bị JS cản trở.

**2. Làm thế nào để vừa duy trì tính năng "Nhúng Youtube vào Web" (Video Player) mà không sợ bị dính lỗ hổng Clickjacking trên nút Play/Subscribe?**
- **Junior:** Youtube chắc có tool chặn riêng xịn lắm.
- **Senior:** Youtube không sợ Clickjacking (Vì họ MUỐN bạn nhúng Video khắp nơi). Cái họ sợ là Nút Đăng Ký (Subscribe) bị Clickjacking.
Giải pháp Kiến trúc: Youtube tách biệt hoàn toàn Giao diện Video Player và Giao diện Đăng ký Kênh.
Youtube trả về X-Frame-Options/CSP MỞ TOANG (Cho phép nhúng) đối với URL `/embed/video_123`. Tại đây, người dùng bấm nút Play, cao lắm là view ảo, rủi ro thấp.
Nhưng đối với URL trang quản lý Kênh (`/channel_abc`) hoặc nút Mua Hàng, Youtube đóng chặt `X-Frame-Options: SAMEORIGIN`. Bạn không bao giờ có thể nhúng một trang Mua Hàng/Cài Đặt của Google vào web của bạn được. Bảo mật Tùy biến theo Từng Endpoint.

**3. Khái niệm "Likejacking" hay "Cursor Spoofing" (Giả mạo con trỏ chuột) có phải là một dạng nâng cao của Clickjacking không?**
- **Junior:** Nó giống clickjacking nhưng dùng Facebook Like.
- **Senior:** Rất chính xác. Likejacking cực kỳ nổi tiếng. Hacker giấu cái Nút "Like" của Facebook dính chặt ngay bên dưới Con trỏ Chuột của bạn. Bạn rê chuột đi đâu, Nút Like (Tàng hình) chạy theo đó. Bạn bấm 1 cái vào khoảng không màn hình, Boom! Bạn đã vô tình Like trang Fanpage của Hacker.
Cursor Spoofing cao cấp hơn: Hacker ẩn con trỏ chuột thật (CSS `cursor: none`), rồi vẽ một cái Con trỏ chuột giả (Bằng ảnh) cách con trỏ thật 5 Centimet. Bạn cố gắng di chuyển chuột ảo để bấm vào Chữ A, thực chất con trỏ thật (Vô hình) đang nằm đè lên Chữ B (Nút nguy hiểm). Tất cả đều là nghệ thuật Lừa đảo thị giác (UI Redressing).

**4. Tại sao một hệ thống Single Sign-On (OIDC) như Keycloak lại vô tình trở thành mục tiêu "Béo bở" nhất cho đòn Clickjacking?**
- **Junior:** Vì nó nắm giữ pass của mọi người.
- **Senior:** OIDC có một đặc điểm: Trạng thái Session Tập trung (Centralized Session). Nếu bạn đã Đăng nhập vào App A (Qua Keycloak). Thì sáng hôm sau bạn mở App B, bạn bấm nút Login, Trình duyệt bay sang Keycloak, Keycloak thấy Cookie Session của bạn VẪN CÒN (Remember me). Keycloak tự động (Hoặc chớp màn hình 1 mili-giây) rồi văng bạn về App B (Đăng nhập thành công mà KHÔNG CẦN GÕ LẠI PASS).
Hacker biết điều này. Hắn không cần lấy Pass. Hắn làm một cái Iframe tàng hình chỏ thẳng vào cái Link Ủy Quyền Client OIDC (Nơi có nút: Xác nhận cấp quyền cho App Của Hacker). Vì bạn Đang Login sẵn Keycloak, bạn bị Clickjacking bấm trúng cái Nút Xác nhận đó. Hacker lập tức nhận được Access Token của bạn hợp pháp mà không tốn một giọt mồ hôi. Đó là lý do Cấm Nhúng Iframe Trang Login/Consent là mệnh lệnh Tử Hình.

**5. Nếu Hacker sử dụng kỹ thuật Drag-and-Drop (Kéo Thả) thay vì Click (Clickjacking), X-Frame-Options có cản được không? Đòn tấn công này gọi là gì?**
- **Junior:** Bấm không được thì kéo cũng không được đâu.
- **Senior:** Đòn này gọi là **Drag-and-Drop (D&D) Redressing (Lừa đảo Kéo thả)** hoặc **Strokejacking**.
Hacker không bảo bạn Bấm. Hacker làm một cái game "Kéo củ cà rốt vào mồm con Thỏ".
Con Thỏ tàng hình thực chất là một cái Ô Nhập Liệu (Input Field) của trang Ngân hàng. Củ cà rốt tàng hình thực chất là một cái Khối Văn Bản (Ví dụ: Một đoạn Text chứa Mã Token hoặc Chuỗi Mã Độc XSS do Hacker soạn sẵn).
Khi bạn kéo Cà Rốt nhét vào mồm Thỏ, bản chất là Trình duyệt thực thi Lệnh Kéo-Thả (Kéo Dữ liệu từ vùng A ném vào Input vùng B chéo Iframe). Tường lửa SOP lỏng lẻo với D&D, cho phép dữ liệu lọt vào Cửa sổ bên kia. 
X-Frame-Options VẪN CHẶN ĐƯỢC đòn này (Bởi vì nếu Iframe không được vẽ ra, thì không có chỗ để thả). Nhưng nếu hệ thống lỡ Mở XFO, D&D Redressing là kỹ thuật cực hiểm để vượt qua các Captcha không cần Click.

---

## 7. Tài liệu tham khảo (References)
- **OWASP:** Clickjacking Defense Cheat Sheet.
- **RFC 7034:** HTTP Header Field X-Frame-Options.
- **W3C:** Content Security Policy Level 3.
