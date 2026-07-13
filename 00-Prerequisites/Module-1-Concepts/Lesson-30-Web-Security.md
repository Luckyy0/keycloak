# Lesson 30: Nền tảng Bảo mật Web (Web Security Fundamentals)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Thiết lập nền móng vững chắc về các ranh giới bảo mật mà Trình duyệt (Browser) áp đặt lên thế giới Internet. Làm chủ Same-Origin Policy (SOP) và nghệ thuật phòng ngự bằng Security Headers.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Trình duyệt - Nhà giam lỏng (Sandbox) của Internet
Internet đầy rẫy các trang web lừa đảo và chứa mã độc. Nếu bạn mở trang `hacker.com`, trang đó sẽ chạy một đống mã Javascript trên máy bạn. Lý do mã độc đó không thể (hoặc rất khó) đọc được ổ cứng hay camera của bạn là nhờ môi trường **Sandbox** của Trình duyệt.
Tuy nhiên, có một rủi ro lớn hơn: Khi bạn mở `hacker.com` ở Tab 1, và mở `facebook.com` ở Tab 2. Làm sao ngăn Javascript của Tab 1 thò tay sang Tab 2 cướp tin nhắn?
Đó là nhờ Định luật Thép: **Same-Origin Policy (SOP)**.

### 1.2. Same-Origin Policy (Chính sách Cùng Nguồn Gốc)
SOP là bức tường lửa được xây cứng vào lõi của mọi Trình duyệt (Chrome, Safari, Firefox). Nó quy định: Cấm một File Javascript thuộc Nguồn Gốc A ĐỌC dữ liệu của Nguồn gốc B.
**Origin (Nguồn Gốc) được định nghĩa bởi 3 yếu tố phải Khớp 100%:**
1. **Scheme (Giao thức):** `http` vs `https` (Khác nhau).
2. **Host (Tên miền):** `auth.com` vs `api.auth.com` (Khác nhau).
3. **Port (Cổng):** `:80` vs `:8080` (Khác nhau).
Chỉ cần lệch 1 trong 3 yếu tố trên, trình duyệt lập tức vung gươm chém đứt cánh tay của Javascript đang cố thò sang ăn cắp dữ liệu.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hiểu rõ Nghịch lý của SOP: "Cấm Đọc nhưng Không Cấm Ghi"

```mermaid
graph TD
    subgraph "Trình duyệt của Nạn nhân"
        JS_Hack["Mã JS chạy trên hacker.com"]
        
        subgraph "Tường lửa SOP (Same-Origin Policy)"
            Block_Read{Cho phép ĐỌC?}
            Allow_Write{Cho phép GHI?}
        end
        
        Target["Máy chủ Ngân hàng (bank.com)"]
        
        JS_Hack -.->|1. Fetch/XHR để Lấy số dư| Block_Read
        Block_Read -.->|SOP Báo Đỏ: CẤM| JS_Hack
        
        JS_Hack -->|2. Submit Form (POST) chuyển tiền| Allow_Write
        Allow_Write -->|SOP Báo Xanh: CHO QUA| Target
    end
    
    Note over JS_Hack,Target: Trình duyệt CẤM Javascript của Hacker ĐỌC kết quả trả về từ Ngân hàng.<br/>Nhưng nó CHO PHÉP Hacker GỬI LỆNH (Viết/Ghi) sang Ngân hàng.<br/>Lỗ hổng này chính là nguồn gốc sinh ra thảm họa CSRF (Sẽ học ở Bài 33).
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tấm khiên HSTS (HTTP Strict Transport Security)**
> Bạn cài SSL (HTTPS) cho web. Nhưng người dùng có thói quen gõ `http://company.com` vào trình duyệt. Trong 100 mili-giây đầu tiên trước khi Nginx kịp Redirect sang HTTPS, Hacker ngồi chung quán Cafe xài đòn Man-in-the-Middle (MitM) cắt ngang kết nối HTTP đó và cướp sạch Pass.
> **Thực hành chuẩn:** Mọi Server đều phải trả về Header `Strict-Transport-Security: max-age=31536000; includeSubDomains`. 
> Nó là lời dặn dò Trình duyệt: "Từ nay về sau (Trong 1 năm), HỄ LÀ tên miền này, mày phải ÉP nó chạy HTTPS ngay từ trong lõi trình duyệt, không được gửi HTTP ra ngoài mạng dù chỉ 1 giây".

> [!CAUTION]
> **X-Frame-Options (Chống nhúng Web - Clickjacking)**
> Hacker làm một trang web đen, nhúng trang web Ngân hàng của bạn vào một cái `<iframe>` tàng hình. Người dùng tưởng bấm nút "Xem Video", nhưng thực chất là đang bấm vào nút "Chuyển tiền" của cái khung ẩn đó.
> **Thực hành chuẩn:** Trả về Header `X-Frame-Options: DENY` (Cấm nhúng hoàn toàn) hoặc `SAMEORIGIN` (Chỉ cho phép tên miền nội bộ được nhúng). Keycloak bật Header này mặc định cực kỳ gắt gao.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Cấu hình "Bọc Thép" Nginx bằng Security Headers tiêu chuẩn Doanh nghiệp:

```nginx
server {
    listen 443 ssl;
    server_name auth.company.com;

    # 1. Ép buộc HTTPS bằng HSTS (1 Năm)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # 2. Ngăn chặn Clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;

    # 3. Ngăn chặn MIME Sniffing (Ép Browser đọc đúng Content-Type, chống lén nhúng JS)
    add_header X-Content-Type-Options "nosniff" always;

    # 4. Ngăn chặn Reflected XSS (Tính năng cũ, đa số Browser hiện đại tự thay thế bằng CSP)
    add_header X-XSS-Protection "1; mode=block" always;
    
    # 5. Kiểm soát thông tin Referer lọt ra ngoài (Tránh rò rỉ JWT qua URL)
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **CORS (Cross-Origin Resource Sharing) đục thủng SOP:** Như đã học ở Bài 5 và Bài 57 (Nâng cao). Đôi khi ta BẮT BUỘC phải cho phép Frontend ở `app.com` gọi API sang `auth.com`. Định luật SOP sẽ chặn việc này.
  - CORS bản chất là một **Sự nới lỏng có kiểm soát** của SOP. Bằng việc cấu hình Máy chủ trả về Header `Access-Control-Allow-Origin: app.com`, Máy chủ thông báo cho Trình duyệt: "Tôi đồng ý đục một cái lỗ trên tường lửa SOP để cho phép riêng thằng app.com này được ĐỌC dữ liệu của tôi". Hiểu sai bản chất này dẫn đến việc Dev dùng `*` (Sao) tàn phá hoàn toàn tường lửa SOP.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "Trình duyệt kiểm soát bảo mật (Browser Enforcement)" là nguyên lý cốt lõi của Web Security. Điều này có nghĩa là các Security Headers (SOP, CORS, HSTS) hoàn toàn VÔ DỤNG nếu bị tấn công bởi Postman hoặc Script Python? Đúng hay Sai?**
- **Junior:** Đúng, Postman vượt qua được hết.
- **Senior:** ĐÚNG một nửa và SAI một nửa.
- Các Headers này sinh ra ĐỂ BẢO VỆ NGƯỜI DÙNG CUỐI (Bảo vệ cái Trình duyệt mà User đang dùng khỏi bị Hacker lợi dụng). Trình duyệt mới là kẻ thực thi việc "Khóa mõm", "Từ chối nhúng Iframe". Do đó Postman (Không phải trình duyệt) sẽ hoàn toàn phớt lờ các ranh giới này.
- NHƯNG, Postman KHÔNG THỂ hack được Server bằng cách lờ đi CORS. Vì CORS sinh ra không phải để cấm Postman gọi API, mà sinh ra để Cấm Hacker dụ dỗ Browser của Nạn Nhân tự động gọi API. Việc Postman gọi được API của bạn là chuyện hiển nhiên (Trừ khi nó không có Token thì bị Backend chặn ở luồng 401). Hiểu lầm CORS là Tường lửa của Server là sai lầm chết người của 90% Junior.

**2. Nếu SOP kiểm soát quyền Đọc chéo (Cross-origin Read). Vậy tại sao thẻ `<img src="https://hacker.com/anh.jpg">` lại được phép hiển thị hình ảnh từ một nguồn khác trên web của tôi?**
- **Junior:** Tại hình ảnh thì nó vô hại nên trình duyệt cho qua.
- **Senior:** SOP cố ý NỚI LỎNG (Exempt) việc nhúng các Tài nguyên Phụ (Sub-resources) thông qua các thẻ HTML như `<img>`, `<script src="">`, và `<link rel="stylesheet">`. 
Bởi vì nếu SOP chặn cả những thứ này, thế giới Web sẽ sụp đổ (Bạn không thể nhúng thư viện JQuery từ CDN, không thể nhúng video Youtube). 
Tuy nhiên, dù được nhúng, Javascript của trang web BẠN (Tên miền A) KHÔNG THỂ dùng lệnh Canvas để "Đọc" (Read) từng điểm ảnh của bức ảnh (Tên miền B) đó được. Trình duyệt vẽ nó ra màn hình, nhưng cấm Javascript chạm vào ruột của nó.

**3. Thuộc tính `nosniff` trong Header `X-Content-Type-Options` bảo vệ máy chủ khỏi đòn tấn công nào? (Polyglot files)**
- **Junior:** Nó cấm ăn cắp dữ liệu.
- **Senior:** Nó chống lại **Lỗ hổng Chuyển đổi Kiểu Dữ Liệu (MIME-Sniffing)**. 
Nếu bạn cho User upload Avatar (Bắt buộc đuôi `.jpg`). Hacker rất tinh ranh, tải lên file `anh.jpg`, nhưng trong ruột file đó hắn nhét một đoạn mã `<script>alert('hack')</script>`.
Khi trình duyệt kéo cái ảnh đó về, một số trình duyệt (Như IE cũ) có tính năng "Thông minh": Nó thấy file đuôi .jpg, nhưng nó "Ngửi" (Sniff) thấy nội dung bên trong giống Javascript, thế là nó Hô Biến biến cái ảnh đó thành một File Mã Độc và THỰC THI (Run) nó.
Header `nosniff` là lệnh Tối thượng: "Server bảo cái này là `image/jpeg` thì mày CHỈ ĐƯỢC VẼ nó ra như cái ảnh. Dù trong đó có chứa HTML hay JS, TUYỆT ĐỐI KHÔNG ĐƯỢC CHẠY".

**4. CSP (Content Security Policy) được coi là Vũ khí Nguyên tử diệt trừ XSS. Cơ chế hoạt động của nó là gì?**
- **Junior:** Nó lọc bỏ mã độc ra khỏi trang web.
- **Senior:** CSP (Policy) là một bản "Danh sách Trắng" (Whitelist) cực kỳ khắt khe được truyền qua HTTP Header. Nó chặt đứt hoàn toàn đôi tay của Hacker dù Hacker có nhúng được mã độc vào HTML.
Thay vì cố gắng tìm xem đâu là mã độc, CSP tuyên bố: 
`Content-Security-Policy: default-src 'self'; script-src https://cdn.trust.com`
Câu thần chú này ép Trình duyệt: (1) Chỉ cho phép tải Javascript từ đúng 1 nguồn `cdn.trust.com` (Ngay cả JS viết thẳng trong thẻ `<script>` trên web cũng bị cấm chạy - Inline script blocking). (2) Cấm JS tự tạo kết nối ra ngoài (No exfiltration). Bất chấp Hacker cấy được mã độc thành công, mã độc đó sẽ bị Trình duyệt khóa cứng không cho nhúc nhích.

**5. Lỗ hổng "Referer Leakage" ảnh hưởng như thế nào đến việc cấp Password Reset Link hoặc Token qua URL?**
- **Junior:** Nó làm lộ URL cho người khác thấy.
- **Senior:** Trình duyệt tự động gắn một Header mang tên `Referer` vào mọi Request để báo cáo "Tôi đi tới từ trang nào".
Kịch bản: Email gửi User một link Đổi Mật Khẩu chứa Token bí mật `https://auth.com/reset?token=123`. User bấm vào, web mở ra trang Đổi pass. Trong trang Đổi pass đó, Developer nhúng cái logo của công ty lấy từ `https://cdn.other.com/logo.png`.
Lúc này, trình duyệt gửi Request sang tên miền `cdn.other.com` để tải ảnh. TRONG REQUEST ĐÓ, trình duyệt gắn Header `Referer: https://auth.com/reset?token=123`. 
Bùm! Toàn bộ Token bí mật bay thẳng vào tay người quản lý cái CDN kia. 
**Khắc phục:** Mọi trang web chứa thông tin nhạy cảm trên URL (Dù là OIDC Redirect URIs) BẮT BUỘC phải dùng Header `Referrer-Policy: strict-origin-when-cross-origin` để trình duyệt XÓA phần tham số nhạy cảm khi gọi chéo tên miền.

---

## 7. Tài liệu tham khảo (References)
- **MDN Web Docs:** Same-origin policy.
- **OWASP:** HTTP Security Response Headers Cheat Sheet.
- **Mozilla Observatory:** Công cụ chấm điểm Security Headers (Giống SSL Labs).
