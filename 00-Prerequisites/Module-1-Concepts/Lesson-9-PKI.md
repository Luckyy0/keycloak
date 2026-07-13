# Lesson 9: PKI (Public Key Infrastructure)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Nắm vững kiến trúc cốt lõi của Hạ tầng Khóa Công khai (PKI), hiểu rõ cơ chế "Ủy quyền Niềm tin" (Chain of Trust) và sự phân biệt sống còn giữa Keystore và Truststore trong môi trường Java/Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Vấn đề của Mã hóa Bất đối xứng
Mã hóa Bất đối xứng (như RSA) giải quyết bài toán mã hóa mà không cần chia sẻ chung một khóa bí mật: Server công bố `Public Key` cho cả thế giới, ai cũng có thể dùng nó để mã hóa tin nhắn, nhưng chỉ Server (người giữ `Private Key`) mới giải mã được.
- **Lỗ hổng (The Man-in-the-Middle Problem):** Làm sao trình duyệt của bạn chắc chắn rằng cái `Public Key` nó vừa nhận được thực sự là của ngân hàng `bank.com`, chứ không phải là `Public Key` của một tên Hacker đang đứng giữa đường truyền (cáp quang, wifi)?
- **Giải pháp:** Cần một "Bên thứ ba" uy tín đứng ra bảo lãnh. PKI (Public Key Infrastructure) chính là hệ thống quản lý các bên thứ ba đó.

### 1.2. Các thành phần của PKI
1. **CA (Certificate Authority - Tổ chức Cấp phát Chứng chỉ):** Là những công ty an ninh mạng khổng lồ (như DigiCert, Let's Encrypt, GlobalSign) được toàn thế giới tin tưởng. Họ làm nhiệm vụ xác minh danh tính của người mua, sau đó "ký điện tử" vào `Public Key` của người mua để tạo thành một "Chứng chỉ" (Certificate).
2. **Root CA (CA Gốc):** Là thực thể tối cao trong PKI. Chứng chỉ của Root CA được cài đặt sẵn sâu bên trong mã nguồn của mọi Hệ điều hành (Windows, macOS) và Trình duyệt (Chrome, Firefox). Sự tin tưởng vào Root CA là sự tin tưởng vô điều kiện (Implicit Trust).
3. **Intermediate CA (CA Trung gian):** Root CA quá quan trọng, nếu khóa riêng của nó bị hack, toàn bộ Internet sẽ sụp đổ. Do đó, Root CA thực tế luôn bị tắt nguồn và cất trong két sắt ngoại tuyến (Offline). Nó chỉ được bật lên vài năm một lần để ký ra các `Intermediate CA`. Các máy chủ `Intermediate CA` này mới là máy chủ online chạy hàng ngày để cấp phát chứng chỉ cho khách hàng (Leaf Certificate).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Quá trình "Xác minh Chuỗi Niềm tin" (Certificate Chain of Trust Validation) diễn ra tự động trong trình duyệt:

```mermaid
graph TD
    A[Root CA (DigiCert Root)] -->|Ký điện tử bảo lãnh| B(Intermediate CA)
    B -->|Ký điện tử bảo lãnh| C(Leaf Certificate: auth.enterprise.com)
    
    D[Hệ điều hành / Trình duyệt] -->|Chứa sẵn danh sách| E[Trust Store]
    E -.->|Tin tưởng tuyệt đối| A
    
    C -.->|Trình duyệt kiểm tra chữ ký| B
    B -.->|Trình duyệt kiểm tra chữ ký| A
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
    style E fill:#bfb,stroke:#333,stroke-width:2px
```

1. Máy chủ Keycloak gửi xuống Client cả Chứng chỉ của nó (Leaf) và Chứng chỉ Trung gian (Intermediate).
2. Client dùng `Public Key` của Intermediate để giải mã chữ ký trên Chứng chỉ Leaf -> Thấy khớp -> Tin Leaf.
3. Client dò tìm trong máy tính xem có `Public Key` của Root CA không. Nếu có, dùng nó để giải mã chữ ký trên Intermediate -> Thấy khớp -> Tin Intermediate.
4. Chuỗi niềm tin được xác lập liên tục đến tận Root CA (đã nằm sẵn trong máy tính). Quá trình kết nối TLS bắt đầu.

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Phân biệt Keystore và Truststore trong hệ sinh thái Java**
> Keycloak là một ứng dụng Java. Để thiết lập TLS/HTTPS, nó cần tương tác với 2 kho chứa khóa:
> - **Keystore (Kho chứa khóa):** Chứa Khóa riêng (Private Key) và Chứng chỉ (Certificate) CỦA CHÍNH MÁY CHỦ KEYCLOAK. Nó dùng để chứng minh với thế giới: "Tôi là Keycloak". Tuyệt đối bảo mật.
> - **Truststore (Kho chứa niềm tin):** Chứa danh sách các `Public Key` của các Tổ chức (CA) mà Keycloak tin tưởng. Nếu Keycloak đóng vai trò là Client (ví dụ: Keycloak gọi API sang máy chủ LDAP nội bộ hoặc Identity Provider khác), Keycloak sẽ từ chối kết nối nếu chứng chỉ của máy chủ kia không được ký bởi một CA có mặt trong Truststore của Keycloak.

> [!WARNING]
> **Mối nguy hiểm của Private CA (CA Nội bộ)**
> Trong các ngân hàng lớn, họ thường tự dựng hệ thống Microsoft AD CS để làm Root CA riêng (gọi là Private CA) nhằm tiết kiệm chi phí sinh chứng chỉ nội bộ. Mặc định, Java/Keycloak KHÔNG HỀ biết Root CA nội bộ này là ai. Nếu bạn kết nối Keycloak đến LDAP nội bộ qua LDAPS (Secure LDAP), kết nối sẽ vỡ nát với lỗi `PKIX path building failed`. Bắt buộc quản trị viên phải lấy file `.crt` của Root CA nội bộ đó và import (nhúng) thủ công vào Truststore của Java JVM chạy Keycloak.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lệnh tiêu chuẩn (dùng công cụ `keytool` của JDK) để nhúng chứng chỉ của Root CA nội bộ công ty (Private CA) vào Truststore mặc định của JVM (giúp Keycloak tin tưởng các dịch vụ nội bộ khác):

```bash
# 1. Định vị file Truststore mặc định của Java (thường là cacerts)
JAVA_HOME=/usr/lib/jvm/java-17-openjdk
TRUSTSTORE=$JAVA_HOME/lib/security/cacerts

# 2. Chạy lệnh keytool với quyền sudo, mật khẩu mặc định của cacerts luôn là "changeit"
sudo keytool -importcert \
  -alias "MyCompanyInternalRootCA" \
  -file /path/to/my_company_root_ca.crt \
  -keystore $TRUSTSTORE \
  -storepass changeit \
  -noprompt

# Sau khi chạy lệnh này, phải Restart dịch vụ Keycloak để JVM nạp lại Truststore vào RAM.
```

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Thiếu Chứng chỉ Trung gian (Missing Intermediate Certificate):** Một lỗi cấu hình kinh điển ở các máy chủ Nginx/Apache. Quản trị viên chỉ cấu hình file `auth.crt` (Leaf Certificate) mà quên không nối đuôi (concatenate) file `intermediate.crt` vào cùng. Kết quả: Truy cập bằng Chrome/Firefox trên máy tính có thể vẫn vào được bình thường (vì trình duyệt máy tính có cơ chế thông minh tự tải Intermediate CA thiếu từ Internet bằng AIA fetching), nhưng các ứng dụng Mobile App (Android/iOS) hoặc ứng dụng Java gọi API (RestTemplate/cURL) sẽ lập tức báo lỗi `CERTIFICATE_VERIFY_FAILED` vì chúng không có cơ chế tự tải bù.
  - **Khắc phục:** Luôn cung cấp file "Full Chain" (chuỗi đầy đủ bao gồm cả Leaf và Intermediate) cho cấu hình máy chủ Web.
- **Cross-Signed Certificates (Chứng chỉ ký chéo):** Khi một Root CA cũ sắp hết hạn, họ có thể yêu cầu một Root CA khác ký chéo cho Intermediate CA của mình để duy trì tính tương thích với các thiết bị đời cổ. Đôi khi điều này tạo ra 2 đường đi (2 chuỗi niềm tin) khác nhau cho cùng một chứng chỉ. Lập trình viên phải cẩn thận để thư viện TLS của ứng dụng chọn đúng chuỗi có CA chưa hết hạn.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Root CA khác biệt gì với Intermediate CA? Tại sao không dùng Root CA để cấp phát chứng chỉ cho khách hàng luôn?**
- **Junior:** Root CA là cái to nhất. Không dùng trực tiếp vì sợ bị hack.
- **Senior:** Về mặt mật mã học, cả hai đều là các cặp khóa công khai/bí mật, nhưng khác biệt ở mô hình quản trị rủi ro. Khóa riêng (Private Key) của Root CA là nền tảng tối cao, nếu bị lộ lọt, toàn bộ các chứng chỉ do nó từng phát hành sẽ bị hủy bỏ (Revoked), gây ra thảm họa toàn cầu. Để giới hạn rủi ro (Blast Radius), Root CA được cất giữ ngoại tuyến trong két sắt an toàn (Offline Root). Nó chỉ dùng để ký phát hành chứng chỉ cho một vài Intermediate CAs. Các Intermediate CAs này mới nằm trên máy chủ Online để xử lý nghiệp vụ tự động hóa hàng ngày. Nếu Intermediate CA bị hack, ta chỉ cần dùng Root CA (vẫn an toàn trong két) để hủy bỏ cái Intermediate CA đó.

**2. Nếu tôi chạy một Docker Container chứa Keycloak và nó báo lỗi `PKIX path building failed` khi kết nối đến LDAPS nội bộ. Nguyên nhân và cách xử lý?**
- **Junior:** Do chưa tắt xác thực SSL. Lên code thêm hàm tắt xác thực là được.
- **Senior:** Lỗi này nghĩa là Keycloak nhận được Chứng chỉ X.509 từ máy chủ LDAPS, nhưng khi truy vết "Chain of Trust", JVM của Keycloak không tìm thấy Root CA nội bộ của công ty trong Truststore (do Container Ubuntu mặc định chỉ chứa Public CAs như DigiCert, Let's Encrypt). Giải pháp cấm kỵ là tắt xác thực SSL. Giải pháp chuẩn xác là map (mount) một tệp `cacerts` tùy chỉnh (đã được import Root CA của công ty) vào container thông qua Docker Volume, hoặc dùng tham số cấu hình của Keycloak (SPI Truststore) để trỏ đến tệp JKS chứa Root CA đó.

**3. Làm sao trình duyệt (Client) nhận biết được Public Key do Server gửi về không bị thay đổi (giả mạo) dọc đường?**
- **Junior:** Dựa vào ổ khóa màu xanh trên trình duyệt.
- **Senior:** Nhờ vào chữ ký điện tử (Digital Signature) của CA trên chứng chỉ. CA ban đầu đã dùng `Private Key` của CA để mã hóa một hàm băm (Hash) của thông tin Server (bao gồm cả Public Key của Server). Khi Client nhận được chứng chỉ, nó dùng `Public Key` của CA (có sẵn trong máy) để giải mã chữ ký đó lấy ra mã băm gốc, sau đó tự băm nội dung chứng chỉ một lần nữa. Nếu 2 mã băm khớp nhau (Integrity Check passed), Client khẳng định chắc chắn rằng Public Key này nguyên vẹn và thực sự do CA uy tín xác nhận.

**4. Khái niệm "Self-Signed Certificate" (Chứng chỉ tự ký) là gì và khi nào nên sử dụng?**
- **Junior:** Là chứng chỉ tự mình tạo ra không tốn tiền. Chỉ nên dùng lúc code test.
- **Senior:** Chứng chỉ tự ký là chứng chỉ mà ở đó `Subject` (Người sở hữu) và `Issuer` (Người phát hành) là một. Bản thân Server tự sinh ra khóa và tự ký xác nhận cho mình mà không thông qua bất kỳ CA nào. Trình duyệt mặc định sẽ ném ra màn hình cảnh báo đỏ rực ("Your connection is not private") vì không có Chain of Trust nào liên kết đến Root CA của hệ điều hành. Chỉ nên sử dụng chứng chỉ tự ký ở môi trường Development hoặc Localhost. Khi lên Production, BẮT BUỘC phải mua chứng chỉ thương mại hoặc dùng CA nội bộ (Private CA) đã được import Truststore vào các máy trạm của nhân viên.

**5. Keystore format `JKS` và `PKCS12` khác nhau như thế nào? Chuẩn nào được khuyến nghị hiện tại?**
- **Junior:** Cả hai đều dùng để lưu chứng chỉ. PKCS12 mới hơn.
- **Senior:** `JKS` (Java KeyStore) là định dạng độc quyền (proprietary format) do Sun/Oracle phát triển, chỉ hoạt động hoàn hảo trong hệ sinh thái Java. Nó sử dụng thuật toán mã hóa tương đối yếu. `PKCS12` (thường có đuôi .p12 hoặc .pfx) là tiêu chuẩn công nghiệp (Industry Standard) quốc tế, hoạt động chéo trên mọi nền tảng (Java, Windows, Nginx, OpenSSL) và hỗ trợ thuật toán mã hóa mạnh hơn. Kể từ Java 9, định dạng mặc định đã được chuyển từ JKS sang PKCS12, do đó các quản trị viên hệ thống luôn được khuyến nghị sử dụng PKCS12 để tối đa hóa tính di động (Portability) và bảo mật.

---

## 7. Tài liệu tham khảo (References)
- **RFC 5280:** Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile.
- **Keycloak Official Documentation:** Configuring outgoing HTTP requests (Truststore).
- **Oracle Java Security Documentation:** KeyStore and TrustStore.
