# Lesson 21: Mã hóa Đối xứng AES (Advanced Encryption Standard)

> [!NOTE]
> **Category:** Theory & Security (Lý thuyết & Bảo mật)
> **Goal:** Hiểu sâu về thuật toán mã hóa đối xứng tiêu chuẩn quốc tế AES. Phân phẫu cơ chế Block Cipher và sự khác biệt sinh tử giữa các Chế độ hoạt động (Modes of Operation) như CBC và GCM.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Khái niệm AES
AES (Advanced Encryption Standard) là thuật toán mã hóa **Đối xứng (Symmetric Encryption)**. Tức là Cùng một chiếc Chìa khóa (Key) được dùng để vừa Khóa (Encrypt) vừa Mở khóa (Decrypt).
- Tốc độ cực nhanh, được tối ưu hóa ở cấp độ phần cứng CPU (AES-NI).
- AES sử dụng cơ chế **Block Cipher (Mã hóa theo Khối)**. Nó cắt dữ liệu (Plaintext) thành các khối nhỏ cố định, mỗi khối 128-bit (16 Bytes), và đem đi nhào nặn qua nhiều vòng lặp toán học (Rounds).
- Các phiên bản: AES-128 (10 vòng lặp), AES-192 (12 vòng lặp), AES-256 (14 vòng lặp).

### 1.2. Vấn đề của Mã hóa Khối và Sự ra đời của Modes (Chế độ)
Nếu bạn có một bức ảnh lớn (Nhiều khối 16 Bytes), và có 2 khối dữ liệu giống hệt nhau. Khi đưa qua AES với cùng 1 Key, nó sẽ sinh ra 2 khối mã hóa (Ciphertext) giống hệt nhau. Điều này làm lộ khuôn mẫu (Pattern) của bức ảnh gốc.
Do đó, AES phải kết hợp với các **Modes of Operation (Chế độ hoạt động)** để làm xáo trộn kết quả.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

So sánh luồng hoạt động của ECB (Tồi tệ) và CBC (Tốt hơn):

```mermaid
graph TD
    subgraph "Chế độ ECB (Electronic Codebook) - KÉM BẢO MẬT"
        Block1[Khối Gốc 1] --> AES_E1((AES Key)) --> Cipher1[Khối Mã 1]
        Block2[Khối Gốc 1 (Giống hệt)] --> AES_E2((AES Key)) --> Cipher2[Khối Mã 2 (Giống hệt Khối 1)]
    end
    
    subgraph "Chế độ CBC (Cipher Block Chaining) - AN TOÀN HƠN"
        IV((IV - Vector Khởi tạo ngẫu nhiên))
        BlockA[Khối Gốc A]
        BlockB[Khối Gốc B (Giống A)]
        
        IV -->|XOR| XOR_A(XOR)
        BlockA --> XOR_A
        XOR_A --> AES_C1((AES Key)) --> CipherA[Khối Mã A]
        
        CipherA -->|Lấy Khối Mã A làm IV mới| XOR_B(XOR)
        BlockB --> XOR_B
        XOR_B --> AES_C2((AES Key)) --> CipherB[Khối Mã B (Hoàn toàn Khác A)]
    end
```
*(Chế độ CBC dùng kết quả của khối trước để làm rối khối sau. Khối đầu tiên được làm rối bằng một đoạn dữ liệu ngẫu nhiên gọi là IV - Initialization Vector).*

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Tuyệt đối KHÔNG dùng chế độ ECB**
> Khi lập trình Java, nếu bạn khai báo `Cipher.getInstance("AES")`, mặc định (tùy JVM) nó có thể sẽ dùng chế độ `AES/ECB/PKCS5Padding`. Đây là thảm họa. Hacker sẽ dùng phân tích tần suất để nhìn xuyên thấu dữ liệu của bạn dù không có Key.
> **Thực hành chuẩn:** LUÔN LUÔN khai báo rõ ràng Mode và Padding. Phải sử dụng **AES-GCM (Galois/Counter Mode)**. AES-GCM không chỉ mã hóa mà còn đính kèm Chữ ký xác thực (Authenticated Encryption with Associated Data - AEAD), giúp chống lại các đòn tấn công thay đổi dữ liệu (Tampering).

> [!IMPORTANT]
> **IV (Initialization Vector) phải là Độc nhất (Unique)**
> Khác với Key (Phải giữ bí mật), IV có thể gửi kèm dưới dạng bản rõ cùng với Dữ liệu đã mã hóa.
> Tuy nhiên, Tối kỵ dùng lại (Reuse) một số IV cho 2 lần mã hóa khác nhau với cùng một Key (Đặc biệt là trong chế độ GCM hoặc CTR). Việc dùng lại IV sẽ phá vỡ hoàn toàn toán học, cho phép Hacker khôi phục được toàn bộ dữ liệu gốc mà không cần Key. LUÔN sinh IV ngẫu nhiên (SecureRandom) cho MỖI LẦN gọi hàm mã hóa.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Đoạn code Java tiêu chuẩn mã hóa dữ liệu nhạy cảm bằng **AES-256-GCM**:

```java
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.spec.GCMParameterSpec;
import java.security.SecureRandom;

public class AESGCM {
    public static final int GCM_TAG_LENGTH = 128; // Tính bằng bit
    public static final int IV_LENGTH = 12; // GCM tối ưu nhất với IV 12 bytes

    public static byte[] encrypt(byte[] plaintext, SecretKey key) throws Exception {
        // Sinh IV ngẫu nhiên duy nhất cho lần mã hóa này
        byte[] iv = new byte[IV_LENGTH];
        new SecureRandom().nextBytes(iv);

        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, gcmSpec);

        byte[] ciphertext = cipher.doFinal(plaintext);
        
        // Trả về phải KÈM THEO IV (Để bên nhận còn biết đường giải mã)
        // Thường người ta nối IV vào đầu của Ciphertext: [IV (12 bytes)] + [Ciphertext]
        return concat(iv, ciphertext); 
    }
}
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Lỗi Padding Oracle Attack:** Nếu bạn dùng chế độ AES-CBC, nó đòi hỏi kích thước khối dữ liệu gốc phải là Bội số của 16 Bytes. Nếu không đủ, nó sẽ nhét thêm rác vào (Padding). 
Hacker có thể bắn các gói tin chứa rác ngẫu nhiên vào Máy chủ giải mã. Dựa vào việc Máy chủ phản hồi "Lỗi sai Padding" (Padding Exception) hay "Lỗi sai Dữ liệu", Hacker có thể từ từ đoán ra từng Byte của dữ liệu gốc. 
  - **Khắc phục:** Sử dụng AES-GCM (Chế độ AEAD). AES-GCM hoạt động theo kiểu Counter (Stream Cipher), nó KHÔNG CẦN PADDING. Và nó sẽ xác thực toàn bộ gói tin (Authentication Tag) TRƯỚC KHI giải mã. Nếu gói tin bị sửa đổi, nó văng lỗi ngay lập tức, chặn đứng đòn Padding Oracle.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong các hệ thống OAuth2/OIDC (Ví dụ Keycloak), AES được sử dụng ở đâu và vai trò của nó là gì?**
- **Junior:** Nó dùng để mã hóa mật khẩu của người dùng vào database.
- **Senior:** Nhầm lẫn nghiêm trọng. Mật khẩu lưu vào DB KHÔNG ĐƯỢC MÃ HÓA, mà phải dùng Hàm Băm một chiều (Hash như Bcrypt). Mã hóa đối xứng AES được Keycloak sử dụng trong cơ chế **JWE (JSON Web Encryption)** để mã hóa nội dung của ID Token hoặc Access Token, tránh việc trình duyệt người dùng đọc được các thông tin nhạy cảm. AES cũng được dùng để mã hóa nội dung của các Cookie phiên (Session Cookies) hoặc mã hóa dữ liệu liên lạc với các Identity Provider khác.

**2. Nếu tôi mã hóa một chuỗi chữ "Hello" (5 Bytes) bằng `AES/GCM/NoPadding`. Kích thước của mảng byte (Ciphertext) thu được sẽ là bao nhiêu? (Biết GCM Tag Length = 128 bit).**
- **Junior:** 5 bytes vì mã hóa chữ gì thì ra kích thước đó.
- **Senior:** Kích thước đầu ra sẽ là **21 Bytes**. Trong chế độ GCM, nó là Stream Cipher nên phần dữ liệu mã hóa (Ciphertext) sẽ giữ nguyên kích thước của Plaintext (5 Bytes). Tuy nhiên, GCM tự động đính kèm thêm một Thẻ Xác thực (Authentication Tag) dài 128-bit (Tương đương 16 Bytes) vào đuôi của kết quả. Tổng cộng: `5 + 16 = 21 Bytes`. Nếu tính cả việc lập trình viên phải truyền thêm IV (12 Bytes) đi kèm, tổng dữ liệu lưu trữ/truyền tải sẽ là `21 + 12 = 33 Bytes`.

**3. Tại sao người ta nói Chế độ CBC (Cipher Block Chaining) không thể mã hóa Song song (Parallel Encryption), nhưng GCM hoặc CTR lại làm được? Điều này ảnh hưởng thế nào đến hiệu năng?**
- **Junior:** Tại GCM nó xịn hơn nên chạy nhanh hơn.
- **Senior:** Dựa vào cơ chế toán học (Xem biểu đồ mục 2). CBC yêu cầu: Muốn mã hóa Khối số 2, BẮT BUỘC phải chờ mã hóa xong Khối số 1 (Để lấy kết quả của Khối 1 làm đầu vào XOR cho Khối 2). Đây là nút thắt cổ chai tuyến tính (Sequential bottleneck), CPU có 16 nhân cũng đành chịu chết, chỉ dùng được 1 nhân.
Chế độ GCM (Dựa trên CTR - Counter Mode) hoạt động bằng cách mã hóa một chuỗi số Đếm (Counter). Ví dụ: `AES(IV + Khối 1)`, `AES(IV + Khối 2)`. Việc tính toán các Counter này KHÔNG PHỤ THUỘC VÀO NHAU. Do đó, hệ điều hành có thể vứt 100 khối cho 100 luồng CPU mã hóa cùng một lúc (Parallelism). Tốc độ của GCM/CTR trong các hệ thống Big Data đè bẹp CBC hoàn toàn.

**4. Khái niệm AEAD (Authenticated Encryption with Associated Data) trong AES-GCM có chữ "Associated Data" (Dữ liệu đính kèm). Ý nghĩa thực tiễn của nó là gì?**
- **Junior:** Là dữ liệu gắn thêm vào cho nó an toàn.
- **Senior:** Giả sử bạn gửi một gói tin Ngân hàng gồm 2 phần: Header (Chứa tên người nhận `Alice` - Phải để Bản rõ để Router biết đường gửi) và Payload (Chứa số tiền `$1000` - Bắt buộc Mã hóa). 
Nếu dùng mã hóa thường, Hacker có thể bắt gói tin trên đường đi, sửa chữ `Alice` thành `Bob`. Payload `$1000` vẫn giải mã thành công (Do Hacker không đụng vào Payload), và tiền chuyển nhầm cho Bob.
AEAD giải quyết bằng cách: Đưa chữ `Alice` vào làm **Associated Data (AAD)**. Thuật toán GCM sẽ đưa chữ `Alice` vào công thức tính toán cái Thẻ Xác Thực (Authentication Tag), NHƯNG KHÔNG MÃ HÓA chữ `Alice`. Nếu Hacker đổi chữ `Alice` thành `Bob` trên đường đi. Khi Máy chủ bóc gói tin, nó tính lại Tag, thấy Tag bị sai lệch, nó LẬP TỨC QUĂNG LỖI. AAD bảo vệ tính toàn vẹn cho các Dữ Liệu Rõ (Cleartext Headers) đi kèm với Dữ Liệu Mã hóa.

**5. Nếu chìa khóa AES-256 cực kỳ an toàn, tại sao người ta không dùng nó để gửi mã bí mật (Secret) qua Internet cho nhau luôn, mà lại phải dùng thêm mã hóa Bất đối xứng (RSA)?**
- **Junior:** Vì RSA nó khó bị hack hơn AES.
- **Senior:** Đây là Bài toán Phân phối Khóa (Key Distribution Problem). Mã hóa Đối xứng (AES) cực kỳ an toàn và cực kỳ nhanh. Nhược điểm duy nhất của nó là CẢ 2 BÊN (Gửi và Nhận) PHẢI CÓ CÙNG MỘT CHÌA KHÓA BÍ MẬT.
Nếu tôi ở VN, bạn ở Mỹ. Làm sao tôi gửi chìa khóa AES cho bạn qua mạng Internet (Vốn đầy rẫy Hacker rình rập) mà không bị lộ? Nếu khóa bị lộ trên đường gửi, mã hóa AES trở nên vô dụng.
Đó là lý do Mã hóa Bất đối xứng (RSA/ECC) ra đời: Tôi dùng khóa Public (Công khai) của bạn để bọc cái Khóa AES lại, rồi gửi qua mạng. RSA giải quyết khâu "Bắt tay trao chìa khóa", sau đó hai bên dùng chính chìa khóa AES đó để truyền Dữ liệu khối lượng lớn (Vì AES nhanh gấp ngàn lần RSA).

---

## 7. Tài liệu tham khảo (References)
- **NIST FIPS 197:** Advanced Encryption Standard (AES).
- **RFC 5116:** An Interface and Algorithms for Authenticated Encryption.
- **OWASP:** Cryptographic Storage Cheat Sheet.
