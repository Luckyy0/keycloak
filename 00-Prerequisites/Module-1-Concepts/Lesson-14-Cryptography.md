# Lesson 14: Cryptography (Mật mã học cơ sở)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trang bị nền tảng tư duy mật mã học cốt lõi. Phân biệt rõ ràng định nghĩa vật lý và mục đích sử dụng của ba khái niệm thường xuyên bị nhầm lẫn nhất trong ngành lập trình: Encoding (Biểu diễn), Hashing (Băm), và Encryption (Mã hóa).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Tam giác Bảo mật CIA (CIA Triad)
Mọi hệ thống bảo mật sinh ra (bao gồm cả Keycloak, OAuth2, hay SSL/TLS) đều phải thỏa mãn 3 trụ cột của hệ thống An toàn thông tin, gọi là CIA:
1. **Confidentiality (Tính Bí mật):** Đảm bảo dữ liệu không bị lộ lọt, người không có thẩm quyền không thể đọc được nội dung (Giải quyết bằng: *Encryption*).
2. **Integrity (Tính Toàn vẹn):** Đảm bảo dữ liệu nguyên bản từ gốc, không bị cắt xén hay thay đổi nội dung (thêm bớt dấu phẩy) dọc đường (Giải quyết bằng: *Hashing*, *MAC*).
3. **Availability (Tính Sẵn sàng):** Đảm bảo hệ thống luôn trong trạng thái sẵn sàng phục vụ người dùng hợp lệ (Giải quyết bằng: *Load Balancers*, *Cluster*, *Anti-DDoS*).

*(Khái niệm phụ trợ tối quan trọng trong IAM: **Non-repudiation (Chống chối bỏ)** - Không một ai có thể chối cãi rằng họ KHÔNG PHẢI là người đã thực hiện giao dịch (Giải quyết bằng: *Digital Signatures - Chữ ký điện tử*)).*

### 1.2. Phân định Encoding, Hashing, và Encryption
Đây là 3 khái niệm dễ gây tai nạn bảo mật nghiêm trọng nhất nếu sử dụng sai mục đích.

| Đặc điểm | Encoding (Biểu diễn / Mã hóa ký tự) | Hashing (Hàm băm) | Encryption (Mã hóa) |
| :--- | :--- | :--- | :--- |
| **Mục đích** | Biểu diễn dữ liệu nhị phân thành văn bản an toàn để vận chuyển qua Internet. | Xác thực tính toàn vẹn (Integrity). Ẩn giấu mật khẩu một chiều. | Che giấu nội dung (Confidentiality). Trao đổi bí mật hai chiều. |
| **Bảo mật** | **Không hề bảo mật (Bản rõ).** | Rất cao (Không thể đảo ngược ra bản gốc). | Rất cao (Cần có khóa bí mật để giải mã). |
| **Tính thuận nghịch** | Có (Ai cũng dịch ngược được nếu biết chuẩn format). | **Không (One-way). Dịch 1 chiều.** | Có (Two-way). Dịch ngược nếu có chìa khóa. |
| **Dữ liệu đầu ra** | Thay đổi theo kích thước file gốc. | Độ dài LUÔN cố định (ví dụ 256-bit). | Thay đổi theo kích thước file gốc. |
| **Thuật toán ví dụ**| Base64, ASCII, URL-Encoding. | SHA-256, bcrypt, PBKDF2. | AES (Đối xứng), RSA (Bất đối xứng). |
| **Tình huống sử dụng**| JSON Web Token (Phần Header/Payload), đính kèm file ảnh trong JSON. | Lưu mật khẩu người dùng vào DB. Ký điện tử. | Truyền Access Token qua mạng. Gửi thông điệp bí mật. |

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

### Nguyên lý Kerckhoffs (Kerckhoffs's principle)
Được phát biểu vào thế kỷ 19, đây là nền tảng của mật mã học máy tính hiện đại: 
> "Một hệ thống mật mã phải an toàn ngay cả khi mọi chi tiết về hệ thống (ngoại trừ chìa khóa bí mật - the Key) đều bị công khai phơi bày ra toàn thế giới."

- Các hệ thống cố gắng bảo mật bằng cách viết một thuật toán rắc rối "nhà làm" và giấu nhẹm bộ code đó đi (Security through obscurity) chắc chắn sẽ bị hacker dịch ngược (Reverse-engineer) và phá giải trong thời gian ngắn.
- Các thuật toán chuẩn công nghiệp như AES hay RSA có mã nguồn hoàn toàn nguồn mở (Open-source) để hàng ngàn tiến sĩ toán học thi nhau tìm lỗ hổng. Sự an toàn không nằm ở việc thuật toán bí mật, mà nằm ở độ khó của bài toán phân tích thừa số nguyên tố (để phá cái Khóa 2048-bit). 

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!CAUTION]
> **Base64 KHÔNG PHẢI là Mã hóa (Encryption)**
> Cấu trúc của JWT (JSON Web Token) bao gồm 3 phần được nối bằng dấu chấm. Rất nhiều lập trình viên nghĩ rằng JWT là "đã được mã hóa". Sự thật là, phần Header và Payload của JWT chỉ được `Base64-URL Encoded` (Biểu diễn Base64). Bất kỳ ai bắt được JWT đều có thể copy lên trang `jwt.io` và dịch ngược ra văn bản thuần túy trong 1 giây để đọc toàn bộ thông tin (Email, ID, Roles) bên trong. 
> 
> **Thực hành tốt nhất:** Không bao giờ đưa dữ liệu cực kỳ nhạy cảm (như Mật khẩu, Số thẻ tín dụng, API Key) vào bên trong Payload của một JWT thông thường. Nếu bắt buộc phải đưa, bạn phải dùng khái niệm JWE (JSON Web Encryption) để mã hóa thật sự.

> [!IMPORTANT]
> **Đừng bao giờ tự phát minh mật mã (Never Roll Your Own Crypto)**
> Trong mọi dự án Enterprise, việc tự viết một hàm "đảo lộn chuỗi ký tự kết hợp phép XOR" để lưu mật khẩu bị coi là vi phạm đặc biệt nghiêm trọng. BẮT BUỘC sử dụng các thư viện đã được thẩm định (như BouncyCastle trong Java, WebCrypto trong JS) và sử dụng các thuật toán chuẩn như Argon2, bcrypt cho băm mật khẩu, AES-GCM cho mã hóa đối xứng.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sự khác biệt trong việc sử dụng 3 khái niệm bằng ngôn ngữ Java:

```java
import java.util.Base64;
import java.security.MessageDigest;
import javax.crypto.Cipher;

public class CryptoDemo {
    public static void main(String[] args) throws Exception {
        String data = "MySecretData";

        // 1. ENCODING (Base64) - Để biểu diễn, không bảo mật
        String encoded = Base64.getEncoder().encodeToString(data.getBytes());
        // Kết quả: "TXlTZWNyZXREYXRh". Ai cũng có thể decode ngược lại.

        // 2. HASHING (SHA-256) - Để toàn vẹn, đi một chiều
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] hashed = md.digest(data.getBytes());
        // Kết quả: Chuỗi byte cố định 256-bit. KHÔNG THỂ dịch ngược ra chữ "MySecretData".

        // 3. ENCRYPTION (AES) - Để giấu thông tin, có khóa
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        // (Yêu cầu phải có một SecretKey AES)
        // cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        // byte[] encrypted = cipher.doFinal(data.getBytes());
        // Kết quả bị mã hóa. Chỉ người giữ khóa AES mới có thể cipher.init(DECRYPT_MODE) để đọc.
    }
}
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mối đe dọa Máy tính Lượng tử (Quantum Computing Threat):** Hệ thống mã hóa khóa công khai hiện tại (RSA, ECC) dựa trên nền tảng toán học là các bài toán chưa thể giải bằng máy tính truyền thống (Phân tích thừa số nguyên tố, Đường cong Elliptic). Theo lý thuyết của thuật toán Shor, khi Máy tính lượng tử ra đời và đủ mạnh, nó có thể giải các bài toán này chỉ trong vài phút. Kẻ thù của bảo mật hiện nay đang thu thập toàn bộ dữ liệu mã hóa TLS trên mạng và lưu trữ (Harvest Now, Decrypt Later). Ngành mật mã học đang ráo riết dịch chuyển sang Kỷ nguyên Mật mã Hậu Lượng tử (Post-Quantum Cryptography - PQC). Tuy nhiên, hàm băm (Hashing) và mã hóa đối xứng (AES) bằng khóa kích thước lớn (256-bit) vẫn được coi là an toàn trước máy tính lượng tử.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Nếu một hệ thống lưu mật khẩu người dùng trong cơ sở dữ liệu bằng thuật toán AES-256 (Mã hóa hai chiều), điều này có an toàn không? Tại sao?**
- **Junior:** AES-256 là thuật toán cực kỳ mạnh của quân đội Mỹ nên lưu mật khẩu bằng nó là rất an toàn.
- **Senior:** Đây là thiết kế vi phạm nguyên tắc bảo mật tồi tệ. AES là thuật toán Encryption (Mã hóa hai chiều có thể giải mã). Nếu hệ thống bị hack hoặc kẻ nội gián (DBA, DevOps) truy cập được vào cơ sở dữ liệu và cướp được Secret Key, họ có thể giải mã ngược toàn bộ kho mật khẩu ra văn bản thuần túy (Plaintext). Mật khẩu BẮT BUỘC phải được lưu bằng hàm Băm một chiều (Hashing) đi kèm với Salt (như bcrypt, Argon2, PBKDF2). Với hàm băm, kể cả người quản trị hệ thống cũng không thể biết được mật khẩu gốc của người dùng là gì, họ chỉ có thể lấy mật khẩu người dùng vừa gõ để băm lại và so sánh.

**2. Base64 được dùng rất nhiều trong JSON Web Token (JWT). Công dụng thực sự của nó là gì nếu nó không hề bảo mật?**
- **Junior:** Để làm cho nội dung rối mắt, hacker lười dịch.
- **Senior:** Encoding không liên quan gì đến bảo mật hay làm rối. Mục đích duy nhất của Base64 là "Biểu diễn dữ liệu nhị phân an toàn". HTTP Transport và các Header không thể vận chuyển các ký tự điều khiển (Control Characters) hoặc dữ liệu nhị phân (như chữ ký mã hóa bằng RSA) một cách chính xác. Base64 biên dịch toàn bộ khối dữ liệu phức tạp đó thành các ký tự bảng chữ cái Latin an toàn tuyệt đối (A-Z, a-z, 0-9), giúp chuỗi JWT có thể dễ dàng nhét vào trong HTTP Authorization Header hoặc URL Query mà không làm gãy đường truyền hay bị HTTP Server cắt xén.

**3. Làm sao để đảm bảo "Tính Toàn Vẹn" (Integrity) của một file gửi qua mạng? Giải thích cơ chế?**
- **Junior:** Mã hóa file đó lại, nếu sai mật khẩu thì tức là file bị sửa.
- **Senior:** "Toàn vẹn" không cần đến mã hóa dữ liệu. Ta sử dụng Hàm Băm mật mã học (như SHA-256). Bản chất của hàm băm là: Bất kỳ thay đổi nào dù chỉ là 1 bit ở file gốc, kết quả mã băm đầu ra sẽ thay đổi hoàn toàn (Hiệu ứng tuyết lở - Avalanche Effect). Quy trình: Người gửi băm file A ra mã `Hash_1`, rồi gửi đi file A kèm `Hash_1`. Người nhận nhận file A, tự chạy hàm băm ra `Hash_2`. Nếu `Hash_1 == Hash_2`, đảm bảo 100% file không bị suy suyển một bit nào dọc đường. (Để an toàn hơn chống Man-in-the-Middle thay đổi cả file lẫn Hash, ta phải dùng chữ ký điện tử hoặc HMAC).

**4. Kerckhoffs's Principle phát biểu rằng sự an toàn của mật mã nằm ở chiếc chìa khóa chứ không nằm ở thuật toán. Hãy lấy một ví dụ trong thế giới thực để so sánh?**
- **Junior:** Giống như ổ khóa cửa, cấu tạo ổ khóa thì ai cũng biết nhưng không có chìa thì không mở được.
- **Senior:** Ví dụ hoàn hảo nhất là khóa cửa nhà (Door Lock). Cấu tạo cơ khí của một ổ khóa (Tương đương Thuật toán AES/RSA) là tiêu chuẩn công nghiệp và hoàn toàn công khai, nhà sản xuất sẵn sàng in bản vẽ giải phẫu cấu trúc. Nhưng dù trộm biết nguyên lý hoạt động của ổ khóa, chúng vẫn không thể mở được cửa một cách dễ dàng nếu không có mảnh kim loại răng cưa khớp với rãnh (Tương đương Khóa bí mật 256-bit). Ngược lại, nếu bạn thuê một thợ rèn làm một ổ khóa "tự chế" (Tự viết thuật toán), dù cái rãnh có dị hợm đến đâu, một tay thợ khóa rành nghề cũng có thể tìm ra điểm yếu vật lý (lỗi thuật toán) để nạy bung nó trong vài giây mà không cần đoái hoài gì tới chìa khóa.

**5. Định lý "Chống chối bỏ" (Non-repudiation) trong giao dịch số được giải quyết bằng khái niệm mật mã nào?**
- **Junior:** Dùng camera quay lại màn hình lúc thực hiện giao dịch.
- **Senior:** Chống chối bỏ được giải quyết triệt để bằng `Chữ ký điện tử` (Digital Signatures) ứng dụng Mã hóa Bất đối xứng (Public Key Cryptography). Ví dụ trong ngân hàng: Bob muốn chuyển tiền. Bob lấy thông tin lệnh chuyển, băm ra, rồi dùng `Private Key` (khóa bí mật chỉ mình Bob biết) để mã hóa kết quả băm đó tạo thành chữ ký điện tử. Ngân hàng dùng `Public Key` của Bob (ai cũng biết) để giải mã chữ ký và kiểm chứng. Về sau, Bob không thể ra tòa chối cãi rằng "Tôi không làm lệnh này", bởi vì toán học chứng minh RẰNG: Chỉ có kẻ sở hữu `Private Key` của Bob mới có khả năng tạo ra một chữ ký có thể giải mã thành công bằng `Public Key` của Bob.

---

## 7. Tài liệu tham khảo (References)
- **NIST:** FIPS 197 - Advanced Encryption Standard (AES).
- **OWASP:** Cryptographic Storage Cheat Sheet.
- **RFC 4648:** The Base16, Base32, and Base64 Data Encodings.
