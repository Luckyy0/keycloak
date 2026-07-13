# Lesson 29: Kiến trúc Dữ liệu XML (eXtensible Markup Language)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Lặn xuống đáy biển công nghệ để tìm hiểu về XML. Tại sao một định dạng "Cũ kỹ, Nặng nề và Rườm rà" này lại là Xương sống Vĩnh cửu của các Giao thức Danh tính cấp Doanh nghiệp (Enterprise Identity) như SAML 2.0, và những cạm bẫy chết người (XXE) ẩn giấu bên trong nó.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Bản chất của XML
Khác với JSON (Sinh ra để lưu dữ liệu phần mềm), **XML (eXtensible Markup Language)** sinh ra với tham vọng to lớn hơn: **Mô tả cả Dữ liệu lẫn Cấu trúc của Văn bản (Markup)**. Nó là người anh em họ của HTML.
- XML sử dụng các Cặp thẻ đóng/mở `<Thẻ>Nội dung</Thẻ>`.
- Được thiết kế cực kỳ Kỷ luật, Khắt khe và Tự giải thích (Self-descriptive). Nếu bạn lỡ quên đóng một thẻ, toàn bộ văn bản XML sẽ bị Trình phân tích cú pháp (Parser) chém đầu ngay lập tức (Well-formed requirement).

### 1.2. Tại sao SAML 2.0 lại "Cuồng" XML?
Trong hệ sinh thái IAM, SAML 2.0 (Ra đời 2005) định hình toàn bộ chuẩn Mạng Doanh nghiệp Khổng lồ (Enterprise SSO). Tất cả các gói tin Ủy quyền của SAML đều viết bằng XML (XML Signatures, XML Encryption).
Lý do XML không thể bị thay thế trong SAML:
1. **Không gian tên (Namespaces):** XML cho phép gộp tài liệu của 10 công ty khác nhau vào chung 1 file mà không bị trùng lặp tên Thẻ nhờ tiền tố Namespace (Ví dụ `<saml:Assertion>` vs `<ds:Signature>`). JSON không hề có tính năng này.
2. **Khế ước sắt đá (XSD - XML Schema Definition):** XML sở hữu hệ thống xác thực kiểu dữ liệu Khủng khiếp nhất. Nó không chỉ kiểm tra chuỗi, mà kiểm tra được độ dài, biểu thức chính quy (Regex), thứ tự thẻ. Đảm bảo dữ liệu gửi giữa 2 Tập đoàn đa quốc gia là chính xác 100%.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Một ví dụ mô phỏng Khẳng định Danh tính (Assertion) của SAML 2.0 được nhào nặn bằng XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Không gian tên (Namespaces) định danh bộ từ vựng -->
<saml2:Assertion xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion" 
                 ID="_1234567890" IssueInstant="2024-05-20T10:00:00Z" Version="2.0">
    
    <!-- Tổ chức phát hành danh tính (VD: Keycloak) -->
    <saml2:Issuer>https://auth.company.com/</saml2:Issuer>
    
    <!-- Chữ ký số bọc bên trong XML (XML Signature) -->
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:SignedInfo>
            <!-- Tham chiếu thuật toán băm và ký -->
        </ds:SignedInfo>
        <ds:SignatureValue>Mã-Base64-Chữ-Ký-Nằm-Ở-Đây</ds:SignatureValue>
    </ds:Signature>

    <!-- Khẳng định đối tượng (User) -->
    <saml2:Subject>
        <saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
            alice@company.com
        </saml2:NameID>
    </saml2:Subject>
    
</saml2:Assertion>
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Thảm họa cấp S: Lỗ hổng XXE (XML External Entity)**
> Đây là lỗ hổng Kinh dị và Hủy diệt nhất của XML (Đứng top OWASP).
> XML có một tính năng di sản (Legacy) gọi là DTD (Document Type Definition), cho phép khai báo các "Thực thể bên ngoài" (External Entities).
> **Kịch bản Hack:** Hacker gửi lên Máy chủ SAML một file XML có chứa thẻ DTD mưu mô: `<!ENTITY xxe SYSTEM "file:///etc/passwd">`. Sau đó trong thân XML ghi `<username>&xxe;</username>`.
> Khi Máy chủ (XML Parser của Java) đọc cái file này, nó sẽ Ngoan Ngoãn tuân lệnh: Nó Tự Động mở ổ cứng Server, đọc sạch nội dung file chứa Mật khẩu Hệ điều hành Linux (`/etc/passwd`), thay thế vào chữ `&xxe;`, và trả về cho Hacker xem. Hacker cướp toàn bộ ổ cứng máy chủ từ xa.
> **Thực hành chuẩn:** KHI KHỞI TẠO BẤT KỲ XML PARSER NÀO, KIẾN TRÚC SƯ BẮT BUỘC PHẢI VIẾT LỆNH: **Disallow DOCTYPE declaration (Vô hiệu hóa toàn bộ DTD)**. Các hệ thống như Keycloak đã khóa kín tính năng này mặc định.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Kỹ thuật Phòng thủ XXE cốt lõi trong Code Java (Khi bạn tự viết Custom SPI cho Keycloak hoặc xử lý XML ngoài rìa):

```java
import javax.xml.parsers.DocumentBuilderFactory;

public class SecureXMLParser {
    public static DocumentBuilderFactory createSecureFactory() throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        
        // 1. NGĂN CHẶN XXE: CẤM TUYỆT ĐỐI MỌI KHAI BÁO DTD (DOCTYPE)
        factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        
        // 2. Chặn việc nhúng các File (External General Entities)
        factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
        
        // 3. Chặn việc tải các Schema ngoài (External Parameter Entities)
        factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
        
        // 4. Ép XML Parser không nạp các DTD bổ sung từ Internet
        factory.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
        
        // Bật tính năng nhận dạng Namespace (Rất quan trọng cho SAML)
        factory.setNamespaceAware(true);
        
        return factory;
    }
}
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Bom tỷ phú (Billion Laughs Attack / XML Bomb):** Đây là đòn tấn công Từ chối Dịch vụ (DoS) tinh vi của XML mà không cần tới External Entity.
Hacker khai báo 1 Entity tên là L1 chứa 10 chữ "Ha". Rồi khai báo L2 chứa 10 cái L1. L3 chứa 10 cái L2... Chỉ với một đoạn Code XML dài chưa tới 1 Kilobyte. Khi Máy chủ bung cái vòng lặp Đệ quy (Recursive) này ra, nó sẽ nổ tung thành 3 Gigabytes dữ liệu chữ "Ha" trên RAM, đánh sập Máy chủ Server tức tưởi.
  - **Khắc phục:** Các thư viện Java XML hiện đại (JAXP) đều có giới hạn `entityExpansionLimit` (Mặc định = 64000). Nó sẽ tự động cắt cầu dao nếu phát hiện vòng lặp đệ quy quá đà.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong giao thức SAML, người ta sử dụng khái niệm "XML Signature". Nó khác gì so với JSON Web Signature (JWS) ở Bài 24?**
- **Junior:** Nó dùng XML thay vì JSON để chứa chữ ký.
- **Senior:** Điểm khác biệt kinh điển nằm ở Khả năng Định vị (Placement) của Chữ ký.
JWS (JSON) là cấu trúc Cứng nhắc: Chữ ký luôn luôn nằm ở cái ĐUÔI của chuỗi (Phần thứ 3).
**XML Signature (XML-DSig)** sở hữu sự linh hoạt cực độ:
1. **Enveloped:** Chữ ký chui vào nằm TRONG RỘT của dữ liệu bị ký (Giống SAML Assertion).
2. **Enveloping:** Chữ ký BỌC LẤY toàn bộ dữ liệu ở bên trong nó.
3. **Detached:** Chữ ký nằm ở File A, Dữ liệu nằm ở File B (Tách rời hoàn toàn nhưng trỏ Link tới nhau).
Sự linh hoạt này giúp XML Signature có thể ký từng MẢNH NHỎ trong một file XML lớn (Ví dụ: Chỉ ký Thẻ Khẳng định, nhưng không ký Thẻ Phụ lục), điều mà JWS nguyên thủy không làm được.

**2. Lỗ hổng "XML Signature Wrapping (XSW)" đã từng đánh sập rất nhiều hệ thống SAML SSO. Cơ chế của đòn tấn công này là gì?**
- **Junior:** Hacker giả mạo chữ ký để qua mặt máy chủ.
- **Senior:** Đây là đòn Tráo đổi Cấu trúc đỉnh cao. Hacker KHÔNG HỀ giả mạo hay bẻ khóa Chữ ký (Vì không có Private Key).
**Cơ chế:** Hacker lấy 1 gói XML SAML hợp lệ (Có chữ ký chuẩn do Keycloak cấp cho user thường tên `Hacker`). Hắn GIỮ NGUYÊN cụm Dữ liệu thật + Chữ ký thật. NHƯNG, hắn sao chép một cụm Dữ liệu Fake (Với user tên `Admin`), nhét cái Cụm Fake đó vào Tầng trên cùng của File XML. Hắn đẩy cái Cụm Dữ Liệu thật (Cái đang được bảo vệ bởi Chữ ký) xuống làm một cái Thẻ Con (Sub-node) không ai quan tâm.
Khi API Backend chạy hàm Verify (Kiểm tra Chữ ký), Hàm Verify dò tìm cái cụm Dữ liệu thật ở dưới đáy -> Chữ ký khớp! (Bảo mật lọt lưới).
Nhưng khi Application Code lấy dữ liệu để Login, nó dùng hàm `getElementById` chọc vào Tầng trên cùng (Gặp ngay cụm Dữ liệu Fake) -> Hacker đăng nhập thành công với quyền Admin. Lỗ hổng này xảy ra do Logic Kiểm tra Chữ ký và Logic Lấy Dữ Liệu không nhìn vào cùng một chỗ trong cấu trúc Cây DOM.

**3. XML Namespaces (Không gian tên) là một lợi thế lớn của XML. Hãy cho một ví dụ thực tế trong hệ thống Identity nơi JSON sẽ bị "Vỡ trận" nếu không có Namespaces?**
- **Junior:** Khi hai người đặt tên biến giống nhau thì JSON bị lỗi.
- **Senior:** Giả sử một công ty Dược phẩm và một Ngân hàng muốn sát nhập dữ liệu. Ngân hàng có thẻ `<Account>` (Số dư tài khoản tiền). Dược phẩm có thẻ `<Account>` (Tài khoản người dùng Web).
Nếu dùng JSON: `{"Account": "100"}`. Máy chủ không thể biết đây là tiền hay là User ID, dữ liệu sẽ ghi đè nhau nát bét (Key Collision).
Nếu dùng XML: Bạn khai báo `<bank:Account>100</bank:Account>` và `<pharma:Account>100</pharma:Account>`. Hai bộ dữ liệu khổng lồ gộp chung vào 1 file XML vẫn giữ nguyên được ý nghĩa Nghiệp vụ riêng biệt mà không hề xung đột. Đó là lý do hệ thống B2B Enterprise Khổng lồ (Nhắn tin liên ngân hàng SWIFT) vẫn sống chết bám lấy XML.

**4. Khác biệt giữa DOM Parser và SAX Parser khi xử lý các File cấu hình XML khổng lồ (VD: 5GB) là gì?**
- **Junior:** DOM dễ dùng hơn, SAX chạy nhanh hơn.
- **Senior:** DOM (Document Object Model) là Mô hình Bộ nhớ. Khi dùng DOM, máy chủ đọc Toàn bộ file 5GB đó và nhét vào RAM để tạo ra một cái Cây (Tree) Object hoàn chỉnh. Máy chủ sẽ lập tức bị nổ RAM (OOM - Out of Memory). Đổi lại, bạn có thể nhảy tới nhảy lui mọi node trên Cây dễ dàng.
SAX (Simple API for XML) là Mô hình Sự kiện (Event-driven). Nó đọc File 5GB theo hình thức Dòng Chảy (Stream) từ Ổ cứng. Cứ gặp Thẻ Mở `<user>`, nó chớp sự kiện `startElement`. Gặp thẻ đóng `</user>`, nó chớp sự kiện `endElement` rồi giải phóng RAM phần đó. Dung lượng RAM tiêu thụ luỗn nằm ở mức gần 0 Bytes dù file có lớn cỡ nào. Đổi lại, bạn chỉ được đi tới 1 chiều, không thể quay lùi lại trên cấu trúc XML.

**5. Khái niệm XSLT (eXtensible Stylesheet Language Transformations) trong hệ sinh thái XML là gì? Nó có tác dụng gì khi các Microservices khác version giao tiếp với nhau?**
- **Junior:** Nó dùng để trang trí màu sắc cho file XML cho đẹp.
- **Senior:** Hoàn toàn sai. Chữ "Stylesheet" trong XSLT hay làm Dev nhầm lẫn với CSS của HTML. Thực chất, XSLT là một **Ngôn ngữ Lập trình Chuyển đổi Dữ liệu**.
Khi Service A (Phiên bản cũ) gửi gói XML phiên bản v1, Service B (Phiên bản mới) chỉ nhận XML phiên bản v2. Thay vì phải sửa mã nguồn Java của Service B để học cách parse XML v1. Hệ thống chỉ cần nhúng một File XSLT (Map Transformation) đứng ở giữa (API Gateway). XSLT sẽ tự động nhào nặn, bóc tách, đảo ngược cấu trúc Cây XML v1, biến hình nó thành XML v2 tiêu chuẩn và ném cho Service B. Tính năng "Biến hình dữ liệu trên đường bay" này giúp việc Tích hợp Hệ thống Di sản (Legacy System Integration) cực kỳ mượt mà.

---

## 7. Tài liệu tham khảo (References)
- **W3C:** Extensible Markup Language (XML) 1.0.
- **OWASP:** XML External Entity (XXE) Prevention Cheat Sheet.
- **SAML 2.0:** Assertions and Protocols for the OASIS Security Assertion Markup Language.
