# Lesson 25: JSON Web Encryption (JWE)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Lắp ghép mảnh ghép tối thượng của Mật mã học (Cryptography) trong OIDC: JWE. Hiểu kỹ thuật Envelope Encryption (Mã hóa 2 lớp) để biến Token thành một viên gạch tàng hình mà ngay cả Firewall cũng không đọc được nội dung.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Tại sao JWS (Chữ ký số) là chưa đủ?
Như đã học ở Bài 24, JWS (`xxxxx.yyyyy.zzzzz`) ai cũng đọc được phần Payload (Dùng Base64 decode). 
Giả sử hệ thống Bệnh viện cấp cho bệnh nhân một Token. Trong Payload chứa thuộc tính: `"benh_nen": "HIV_Positive"`.
Dù JWS không cho phép ai sửa thông tin này, nhưng BẤT KỲ AI ngó qua màn hình của bệnh nhân, hoặc Router mạng trung gian ở Quán Cafe đều ĐỌC ĐƯỢC căn bệnh nhạy cảm đó. Đây là thảm họa vi phạm dữ liệu cá nhân (GDPR/HIPAA).
**JSON Web Encryption (JWE - RFC 7516)** ra đời để bịt kín lỗ hổng này. Nó mã hóa toàn thân Token.

### 1.2. JWE và Kỹ thuật Envelope Encryption (Mã hóa Bọc thư)
Để vừa có Tốc độ cao (AES) vừa Gửi được Chìa khóa qua mạng (RSA), JWE sử dụng kỹ thuật Mã hóa 2 Lớp (Envelope Encryption):
1. **Lớp Lõi (Nội dung):** Dùng thuật toán Đối xứng cực nhanh (AES-GCM) để mã hóa cái Payload JSON nhạy cảm. Chìa khóa AES này gọi là **CEK (Content Encryption Key)**.
2. **Lớp Vỏ (Bọc chìa khóa):** Dùng thuật toán Bất đối xứng (RSA) để MÃ HÓA CÁI CEK (Chìa AES) kia lại. Khóa RSA này gọi là **KEK (Key Encryption Key)**.
Kết quả: Payload bị AES bọc, còn chìa AES bị RSA bọc. Tất cả gói thành một chuỗi JWE gửi đi.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Chuỗi JWE khác hẳn JWS. Nó dài thòng lọng và có tới **5 PHẦN** (Ngăn cách bằng 4 dấu chấm):

```mermaid
graph TD
    subgraph "Cấu trúc JWE (5 khối Base64Url)"
        H[1. JOSE Header]
        E[2. Encrypted Key]
        I[3. Initialization Vector - IV]
        C[4. Ciphertext]
        A[5. Authentication Tag]
        
        H -->|.| E -->|.| I -->|.| C -->|.| A
    end
    
    Note over H,A: Header báo thuật toán RSA bọc thuật toán AES.<br/>Encrypted Key là cái Chìa AES bị RSA mã hóa.<br/>IV dùng cho chế độ CBC/GCM của AES.<br/>Ciphertext là ruột JSON đã biến thành rác.<br/>Tag là nhãn xác thực AEAD của thuật toán GCM.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Hiệu năng và Nút thắt cổ chai**
> JWE mã hóa cực kỳ an toàn nhưng Tốn Tốc Độ (Performance Overhead). Máy chủ phải giải mã 2 lần (RSA bóc CEK, rồi AES bóc Ciphertext) cho MỖI LẦN Request đập vào API.
> **Thực hành chuẩn:** Tuyệt đối không dùng JWE mặc định cho mọi API thông thường. Chỉ kích hoạt JWE ở các chuẩn Bảo mật Ngân hàng (FAPI), hoặc khi Token chứa PII (Căn cước công dân, Lịch sử tín dụng). Đối với Token bình thường chứa các Claims vô thưởng vô phạt (Role ID, Department ID), hãy dùng JWS (Chữ ký số) là quá đủ và tiết kiệm CPU.

> [!IMPORTANT]
> **Nested JWT (Ký Xong Mới Mã Hóa hay Mã Hóa Xong Mới Ký?)**
> Nếu bạn muốn token Vừa Kín (JWE) Vừa Không Thể Chối Bỏ (JWS). Chuyên gia (IETF) khuyên dùng mẫu **Nested JWT**: 
> **THỰC HÀNH CHUẨN:** Ký dữ liệu trước (Tạo ra JWS), sau đó nhét nguyên cái JWS đó vào ruột, rồi Mã hóa toàn thân (Bọc JWE ra ngoài). 
> Tại sao? Vì Máy chủ API (Tầng ngoài cùng) sẽ giải mã JWE (Bóc lớp giáp ra), thu được lớp JWS bên trong. Sau đó nó tự tin kiểm tra Chữ ký của JWS. Nếu hợp lệ nó cho qua. Cách này tránh rò rỉ bất kỳ thông tin nào kể cả Chữ Ký.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

JOSE Header của một JWE phức tạp hơn JWS vì nó phải khai báo 2 bộ Thuật toán:

```json
{
  "alg": "RSA-OAEP-256", 
  "enc": "A256GCM",
  "kid": "keycloak-pub-123"
}
```
- `alg` (Algorithm): Là thuật toán dùng để Mã Hóa Cái Chìa Khóa (KEK). Ở đây dùng RSA.
- `enc` (Encryption): Là thuật toán dùng để Mã Hóa Nội Dung thật (CEK). Ở đây dùng AES-256 chế độ GCM.

*Trong Keycloak, để ép một Client xuất ra JWE, bạn vào phần cấu hình của Client đó -> Tab `Advanced` -> Phân hệ `Fine Grain OpenID Connect Configuration` -> Cài đặt `ID Token Encryption Key Management Algorithm` thành RSA-OAEP.*

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Lệch Phễu Giải Mã ở API Gateway:** Khi bạn thiết kế kiến trúc Microservices, API Gateway (Kong, Spring Cloud Gateway) đóng vai trò Gác cổng. Nó thường kiểm tra JWS rất mượt. Nhưng nếu bạn ném JWE cho nó, Gateway sẽ bị "Mù".
  - **Khắc phục:** Máy chủ Gateway BẮT BUỘC PHẢI CÓ **Private Key** để bóc cái JWE đó ra. Điều này vô tình biến Gateway thành một "Kho chứa Private Key" rủi ro. Phương án xịn xò hơn (Kiến trúc Opaque Token / Token Exchange): Backend Gateway cầm JWE chọc ngược về Keycloak (qua Endpoint Introspection) để Keycloak giải mã giúp, Gateway chỉ nhận lại kết quả True/False, giảm tải rủi ro quản lý khóa trên các Edge Router.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "CEK" và "KEK" trong cơ chế JWE là gì? Vai trò của chúng giải quyết mâu thuẫn bảo mật nào?**
- **Junior:** CEK là key nội dung, KEK là key ổ khóa.
- **Senior:** CEK (Content Encryption Key) là khóa Đối Xứng (như AES) sinh ngẫu nhiên dùng để mã hóa Payload (Tốc độ ánh sáng). KEK (Key Encryption Key) là khóa Bất Đối Xứng (như RSA Public Key) dùng để bọc cái CEK lại (Chậm chạp, nhưng an toàn mạng).
Cơ chế này giải quyết mâu thuẫn khốc liệt của Mật mã học: Bạn muốn mã hóa dữ liệu hàng Megabytes cực nhanh (Bắt buộc dùng AES), NHƯNG bạn lại muốn chia sẻ cái khóa AES đó an toàn cho đối tác qua Internet (Bắt buộc dùng RSA). KEK bọc CEK, CEK bọc Payload. Perfect Match!

**2. Nếu một gói tin vừa cần JWS (Ký) vừa cần JWE (Mã hóa). Tại sao quy trình chuẩn là `Sign-then-Encrypt` (Ký xong mới Mã hóa) chứ không phải `Encrypt-then-Sign` (Mã hóa xong mới Ký)?**
- **Junior:** Làm cái nào trước cũng giống nhau.
- **Senior:** Nếu bạn `Encrypt-then-Sign` (Mã hóa thành Ciphertext, rồi Ký lên cục Ciphertext đó): Kẻ thù có thể bắt được gói tin, gỡ bỏ cái Chữ ký của bạn ra, sau đó tự lấy Private Key của hắn Đóng Chữ Ký Của Hắn Lên, rồi gửi cho Server. Server kiểm tra thấy chữ ký Hacker hợp pháp, và bóc ra nội dung mà Bạn Đã Viết. Máy chủ sẽ hiểu lầm Hacker là Chủ nhân của dữ liệu đó. (Lỗ hổng Ký đè - Surreptitious Forwarding).
Quy trình `Sign-then-Encrypt`: Bạn ký trực tiếp lên dữ liệu rõ, bọc Chữ ký và Dữ liệu thành 1 khối không thể tách rời, sau đó dùng Áo giáp JWE bọc nó lại. Hacker không thể thò tay vào trong Áo giáp để tráo đổi chữ ký của bạn được.

**3. Tại sao chuẩn JWE lại chia làm 5 phần (Khác 3 phần của JWS), và cái `Encrypted Key` sinh ra để làm gì?**
- **Junior:** Thêm 2 phần cho mã hóa cho an toàn.
- **Senior:** 5 phần là do quy trình Envelope Encryption ép buộc.
JWE không dùng 1 Khóa chung để mã hóa nội dung. Mỗi lần tạo 1 JWE mới, Keycloak tự quay random sinh ra 1 cái CEK (Chìa AES) mới cứng, không đụng hàng.
Do chìa CEK sinh ngẫu nhiên liên tục, Keycloak BẮT BUỘC phải đính kèm cái CEK đó (Đã bị RSA mã hóa) dán thẳng vào JWE, gửi cho API. Cái đó chính là phần số 2: **Encrypted Key**. Nhờ có phần số 2 này, Máy chủ nhận được JWE sẽ tự bóc ra, lấy cái CEK, rồi tự giải mã phần số 4 (Ciphertext). Hệ thống hoàn toàn không cần lưu trữ hay trao đổi CEK từ trước. Đây là sự kỳ diệu của JWE.

**4. Ứng dụng Frontend (React/Angular) có thể giải mã được ID Token định dạng JWE không? Nếu có thì làm như thế nào?**
- **Junior:** Có, tải thư viện về giải mã là được.
- **Senior:** VỀ NGUYÊN TẮC LÀ KHÔNG ĐƯỢC PHÉP. Để giải mã JWE, bạn bắt buộc phải có **Private Key** (Ứng với Public Key mà Keycloak đã dùng để bọc JWE).
Nếu bạn nhét Private Key vào mã nguồn React/Angular để tải xuống Trình duyệt của User, bạn đã phơi bày Private Key cho cả thế giới thấy qua F12 (Lỗ hổng sống còn). Trình duyệt (Public Client) chỉ nên nhận và dùng JWS. JWE chỉ dành riêng cho các Giao tiếp Backend-to-Backend (Confidential Clients), nơi Private Key được bảo vệ vĩnh viễn trong RAM Máy chủ sâu bên trong Tường lửa.

**5. Thuật toán `dir` (Direct Encryption) trong Header của JWE có ý nghĩa gì?**
- **Junior:** Là mã hóa đi thẳng không qua trung gian.
- **Senior:** Bình thường JWE mã hóa 2 lớp (KEK bọc CEK). Nhưng nếu bạn dùng thuật toán `dir` (Phần `alg` = "dir"), thì JWE BỎ QUA lớp KEK. Nó chỉ sử dụng duy nhất một Khóa Đối Xứng (Symmetric Shared Secret) dùng làm CEK để mã hóa thẳng Payload luôn. 
Lúc này, cấu trúc JWE phần số 2 (Encrypted Key) sẽ bị trống rỗng. `dir` chỉ được dùng khi 2 hệ thống (Ví dụ 2 Microservices nội bộ) đã bí mật thống nhất với nhau một cái Chìa khóa chung (Shared Secret) tuyệt đối an toàn và không bao giờ chia sẻ chìa đó qua mạng. Rất hiếm khi dùng `dir` trong luồng OIDC diện rộng.

---

## 7. Tài liệu tham khảo (References)
- **RFC 7516:** JSON Web Encryption (JWE).
- **RFC 7518:** JSON Web Algorithms (JWA).
- **Keycloak Documentation:** Fine Grain OIDC Configuration (JWE/JWS).
