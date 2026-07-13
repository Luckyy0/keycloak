# Lesson 28: Kiến trúc Dữ liệu JSON (JavaScript Object Notation)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết & Kiến trúc)
> **Goal:** Phân phẫu định dạng dữ liệu thống trị Web hiện đại: JSON. Tại sao nó đánh bại XML trở thành ngôn ngữ chuẩn của mọi API, và đi sâu vào những lỗ hổng chết người khi Phân tích cú pháp (Deserialization Vulnerabilities).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. JSON là gì?
JSON (Đọc là "Giây-Sân") xuất thân từ một đặc tả kỹ thuật của ngôn ngữ JavaScript. Nó là định dạng **văn bản thuần túy (Plain text)** dùng để trao đổi Dữ liệu có cấu trúc.
Mặc dù mang tên JavaScript, JSON ngày nay ĐỘC LẬP HOÀN TOÀN với mọi ngôn ngữ. Java, Python, Go, C# đều có thư viện đọc hiểu JSON.

### 1.2. Tại sao JSON tiêu diệt XML trong thế giới Web API?
Trước năm 2010, XML (với SOAP) là vị Vua tuyệt đối. Nhưng JSON đã lật đổ ngai vàng vì:
1. **Trọng lượng siêu nhẹ (Lightweight):** XML dùng rất nhiều thẻ đóng/mở (Tags) tốn dung lượng băng thông. JSON chỉ dùng ngoặc nhọn `{}` và ngoặc vuông `[]`.
2. **Ánh xạ trực tiếp (Direct Mapping):** Cấu trúc JSON tương đương 1:1 với các cấu trúc dữ liệu lõi trong bộ nhớ của mọi ngôn ngữ lập trình (Hash Map, Array, List, Dictionary). Lập trình viên không cần viết mã Parse phức tạp.
3. **Thân thiện với Trình duyệt:** Trình duyệt (Chạy Javascript) có thể hô biến chuỗi JSON thành một Thực thể Object trong bộ nhớ (Memory Object) bằng hàm `JSON.parse()` trong nháy mắt. Cực kỳ tối ưu cho các Single Page Applications (React/Angular).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Toàn cảnh vòng đời của dữ liệu: Phân cực (Serialization) và Giải cực (Deserialization).

```mermaid
graph TD
    subgraph "Máy chủ Spring Boot (Memory)"
        JavaObj[Thực thể Java (Class User)<br/>String name = "Alice";<br/>int age = 30;]
    end
    
    subgraph "Mạng Internet (Text Stream)"
        JSON_String["Chuỗi Byte: <br/>{<br/>  &quot;name&quot;: &quot;Alice&quot;,<br/>  &quot;age&quot;: 30<br/>}"]
    end
    
    subgraph "Máy khách React (Memory)"
        JSObj[Thực thể Javascript (Object)<br/>user.name == 'Alice'<br/>user.age === 30]
    end
    
    JavaObj -->|Serialization (Jackson)<br/>Biến Object thành Chuỗi Text| JSON_String
    JSON_String -->|Deserialization (JSON.parse)<br/>Đắp Text thành Object trong RAM| JSObj
    
    Note over JavaObj,JSObj: Quá trình Serialization (Serialize) phá vỡ cấu trúc Class phức tạp của RAM,<br/>dàn phẳng nó thành một chuỗi Byte liên tục để truyền qua dây cáp mạng.<br/>Đến nơi, Deserialization dựng lại cấu trúc đó trên RAM của máy nhận.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Thảm họa Giải cực (Deserialization Vulnerability - OWASP Top 10)**
> Nếu Backend Java nhận một chuỗi JSON từ Hacker. Thư viện Parse (Jackson, Fastjson) sẽ đọc chuỗi đó và cố gắng TẠO RA CÁC OBJECT TRONG BỘ NHỚ (RAM) tương ứng.
> Nếu hệ thống cấu hình lỏng lẻo (Kích hoạt tính năng *Polymorphic Deserialization* / Cho phép khởi tạo Class tùy ý), Hacker có thể gửi một cục JSON ra lệnh: *"Hãy khởi tạo Class `java.lang.Runtime` và chạy lệnh bash `rm -rf /`"*. 
> Thư viện Parse vừa đọc cục JSON, vừa tự động chạy mã độc (RCE - Remote Code Execution) trước khi hệ thống bảo mật của bạn kịp làm gì.
> **Thực hành chuẩn:** Tuyệt đối không cho phép JSON tự định nghĩa Class (Ví dụ: Chặn các thẻ `@class` trong Jackson). Luôn ánh xạ JSON trực tiếp vào các Class DTO (Data Transfer Object) an toàn, tĩnh, và không có hàm logic nguy hiểm.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Mổ xẻ một cục JSON cấu hình Realm kinh điển của Keycloak (Realm Export):

```json
{
  "id": "my-realm",                   // String (Chuỗi)
  "realm": "my-realm",
  "enabled": true,                    // Boolean (Đúng/Sai - Không có ngoặc kép)
  "accessTokenLifespan": 300,         // Number (Số nguyên - Tính bằng Giây)
  "roles": {                          // Object lồng Object (Nested Object)
    "realm": [                        // Array (Mảng danh sách các Object Role)
      {
        "name": "admin",
        "description": "Administrator"
      },
      {
        "name": "user"
      }
    ]
  },
  "browserFlow": null                 // Giá trị Null (Trống rỗng)
}
```
*(Ghi chú: Khác với Javascript thuần, trong chuẩn JSON nghiêm ngặt, TOÀN BỘ CÁC TÊN BIẾN (Keys) BẮT BUỘC phải được đặt trong dấu Ngoặc Kép Kép `""`. Tuyệt đối không dùng Ngoặc Kép Đơn `''`).*

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Tràn RAM do Dữ liệu khổng lồ (OOM - Out of Memory):** Giả sử bạn dùng Admin API của Keycloak để xuất (`export`) 10 Triệu Users ra một file JSON. Kích thước file là 5 GB.
Nếu Backend đọc file này bằng phương pháp **DOM Parsing** (Tải toàn bộ file JSON lên RAM, dựng thành một cái Cây (Tree) khổng lồ rồi mới xử lý), Máy chủ sẽ nổ tung RAM (OOM Crash) vì cấu trúc Cây trong RAM tốn gấp 3-4 lần dung lượng File Text.
  - **Khắc phục:** Bắt buộc sử dụng phương pháp **Stream Parsing (Đọc theo dòng suối)**. Thư viện sẽ đọc tuần tự từng dòng chữ từ Ổ cứng, cứ gặp một cặp `{ ... }` của 1 User, nó nạp vào RAM, xử lý xong, vứt khỏi RAM, rồi đọc User tiếp theo. Mức tiêu thụ RAM luôn cố định ở vài Megabytes dù file JSON có lớn cỡ 100 GB.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Định dạng JSON có hỗ trợ kiểu dữ liệu Ngày tháng (Date/Time) hay Dữ liệu Nhị phân (Binary/Image) không? Làm thế nào để truyền chúng qua JSON?**
- **Junior:** Không có kiểu Date, nhưng gửi hình thì nhét luôn cái hình vô.
- **Senior:** Định dạng JSON nguyên thủy (RFC 8259) CHỈ HỖ TRỢ 6 kiểu dữ liệu: String, Number, Object, Array, Boolean, Null. Nó hoàn toàn **KHÔNG HỖ TRỢ** kiểu Date hay Binary nguyên bản.
- Để truyền Ngày Tháng: Bắt buộc phải ép kiểu về String theo chuẩn **ISO-8601** (VD: `"2024-05-20T10:30:00Z"`) hoặc Ép về Number theo dạng **Unix Epoch Time** (VD: `1715000000`). Token JWT (như exp, iat) sử dụng dạng Number Epoch Time cho gọn nhẹ.
- Để truyền Dữ liệu Nhị phân (File ảnh): Bắt buộc phải dùng thuật toán **Base64** mã hóa dải Byte nhị phân đó thành một chuỗi String ký tự văn bản, rồi mới gán vào JSON (VD: `"avatar": "iVBORw0KGgo..."`).

**2. Khái niệm JSON Schema là gì? Tại sao nó đóng vai trò cốt lõi trong việc phòng thủ API Gateway?**
- **Junior:** Nó giống như cái khung xương để biết JSON có những trường gì.
- **Senior:** JSON bản chất là Phi Cấu Trúc (Schema-less). Frontend có thể gửi lên một cục JSON chứa 10.000 trường (fields) rác làm tràn bộ nhớ Backend.
**JSON Schema** là một bản Hợp đồng Ràng buộc (Contract). Nó dùng chính định dạng JSON để định nghĩa Luật lệ cho một gói JSON khác (Ví dụ: Mảng `users` phải là kiểu Array, tối đa 100 phần tử, trường `email` phải đúng chuẩn Regex `@`, trường `age` phải là số nguyên > 18). 
Trong kiến trúc Enterprise, JSON Schema được nạp vào API Gateway (Tầng ngoài cùng). Mọi Request không thỏa mãn Schema sẽ bị chém rụng ngay lập tức, bảo vệ các Microservices bên trong khỏi các payload rác hoặc mã độc Injection.

**3. Số Number trong JSON có giới hạn lớn nhất là bao nhiêu? Nếu truyền số tiền Ngân hàng quá lớn qua JSON, lỗi thảm họa nào có thể xảy ra ở Frontend?**
- **Junior:** Không giới hạn, gõ số bao nhiêu cũng được.
- **Senior:** Trong lý thuyết Text của JSON thì không giới hạn. Nhưng khi Trình duyệt (Javascript) thực thi hàm `JSON.parse()`, nó sẽ ép kiểu Number của JSON về kiểu `Number (Double Precision 64-bit Float)` theo chuẩn IEEE 754 của Javascript.
Giới hạn An toàn tối đa của số nguyên (Max Safe Integer) trong JS là `9,007,199,254,740,991` (2^53 - 1). 
Nếu Backend Java (Dùng kiểu `Long` 64-bit cực lớn, VD: Số dư ví tiền mã hóa hoặc ID Snowflake của Twitter là `1234567890123456789`) truyền qua JSON cho React. React parse xong, số đó sẽ BỊ CẮT XÉN và làm tròn sai lệch hoàn toàn. User có thể mất tiền tỷ vì lỗi làm tròn này.
**Khắc phục:** Mọi số nguyên khổng lồ (Đặc biệt là ID hoặc Tiền tệ) khi truyền qua JSON BẮT BUỘC PHẢI BIẾN THÀNH STRING (Dùng dấu ngoặc kép `"1234567890123456789"`).

**4. JSON Injection là gì? Làm thế nào Hacker lợi dụng nó để vượt quyền hệ thống?**
- **Junior:** Hacker nhét mã SQL vào JSON để hack DB.
- **Senior:** JSON Injection xảy ra khi Máy chủ (Backend) NỐI CHUỖI (String Concatenation) thủ công để tạo ra một gói JSON thay vì dùng thư viện chuẩn hóa (Jackson/Gson).
Ví dụ Backend viết: `String payload = "{\"user\":\"" + username + "\", \"role\":\"user\"}";`
Nếu Hacker nhập ô Username trên web là: `alice", "role":"admin`.
Cục JSON được tạo ra sẽ là: `{"user":"alice", "role":"admin", "role":"user"}`. 
Nhiều thư viện Parse JSON khi gặp 2 cái key `role` trùng nhau, nó sẽ CHỌN LẤY CÁI KEY ĐẦU TIÊN (Là `admin`). Bùm! Hacker tự phong mình làm Admin thành công. Tuyệt đối dùng thư viện Serialize chuẩn, không nối chuỗi văn bản thủ công.

**5. Trong Token OIDC, tại sao JWT lại sử dụng định dạng JSON mà không phải XML hay Protobuf?**
- **Junior:** Vì JSON dễ đọc hơn và nhẹ hơn.
- **Senior:** Có 3 lý do chiến lược:
1. **Hệ sinh thái Trình duyệt:** Trình duyệt là Vua của OIDC/OAuth2. Trình duyệt giải mã JSON nhanh như điện bằng phần cứng (Native engine). XML thì rườm rà (DOM DOMParser), Protobuf thì trình duyệt mù tịt, phải nạp thư viện giải mã khổng lồ.
2. **Kích thước Token (Base64 Encoding):** JWT phải nhét qua HTTP Header (Cookie/Bearer). Nếu dùng XML, kích thước Token sẽ phình to gấp 3 lần (Do các thẻ đóng mở `<tag></tag>`), dễ dàng phá vỡ giới hạn 8KB của HTTP Header trên Nginx/Tomcat.
3. **Lưu trữ Cặp Khóa (JWKS):** Json Web Key Set (JWKS) dùng để phân phối Public Key của Keycloak. Cấu trúc JSON cho phép biểu diễn các tham số toán học của RSA (`n`, `e`) và ECC (`x`, `y`) một cách cực kỳ trực quan và chuẩn hóa, vượt xa khả năng của các định dạng chứng chỉ PEM/DER truyền thống.

---

## 7. Tài liệu tham khảo (References)
- **RFC 8259:** The JavaScript Object Notation (JSON) Data Interchange Format.
- **OWASP:** Deserialization of Untrusted Data (A8:2017).
- **OWASP:** JSON Injection.
