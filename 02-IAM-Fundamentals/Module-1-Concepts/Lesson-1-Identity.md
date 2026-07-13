# Lesson 1: Bản thể và Định danh (Identity vs Account)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Giải quyết triệt để sự nhầm lẫn Kinh điển nhất của mọi Junior Developer: Coi `Identity` (Định danh) và `Account` (Tài khoản) là Một. Nắm được định nghĩa chuẩn xác nhất để xây dựng Bảng Database không bị rác.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Hãy tưởng tượng bạn bước vào Công ty:
- Bạn là anh Nguyễn Văn A, sinh ngày 1/1/1990, số CMND là 123456. Bạn là con người BẰNG XƯƠNG BẰNG THỊT. Đó chính là **Identity (Định danh / Bản thể)**.
- Khi bạn làm việc, IT cấp cho bạn 1 email `nguyenvana@gmail.com` để vào máy tính. Đó là một **Account (Tài khoản)**.
- Sếp cấp cho bạn 1 cái Thẻ từ Quẹt cửa. Đó cũng là một **Account**.

### 1.1. Sự Khác Biệt Sống Còn
- **Identity (Định danh):** Đại diện cho một con người thật. Nó là Độc Nhất (Unique). Không ai giống ai, và KHÔNG BAO GIỜ THAY ĐỔI.
- **Account (Tài khoản):** Đại diện cho PHƯƠNG TIỆN để cái Identity đó đi vào Hệ thống. Một Identity có thể SỞ HỮU HÀNG CHỤC Accounts. Account có thể Bị Khóa, Bị Xóa, Bị Đổi Pass liên tục.

### 1.2. Mối quan hệ 1-N (One-to-Many)
Một sai lầm nghiêm trọng khi thiết kế Database là tạo một Bảng tên là `Users`, trong đó chứa Mật khẩu + Tên + Số CMND.
Khi anh A (1 Identity) muốn đăng nhập cả bằng Mật khẩu và bằng Google, hệ thống lại tạo ra 2 Dòng trong bảng `Users`. (Gây ra hiện tượng Dữ liệu Mồ Côi - Orphaned Data và Phân mảnh - Fragmentation).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Sơ đồ Kiến trúc Chuẩn (Identity Centric Architecture):

```mermaid
graph TD
    subgraph "Tầng Bản Thể (Không bao giờ thay đổi)"
        ID[Identity: Anh A <br/> CMND: 123 <br/> Nhóm máu: O]
    end
    
    subgraph "Tầng Tài Khoản (Thay đổi liên tục)"
        Acc1[Account 1: Đăng nhập bằng Google <br/> Email: a@gmail.com]
        Acc2[Account 2: Đăng nhập bằng Password <br/> Username: admin_A]
        Acc3[Account 3: Đăng nhập bằng Cảm Biến Vân Tay]
    end
    
    Acc1 -->|Thuộc sở hữu của| ID
    Acc2 -->|Thuộc sở hữu của| ID
    Acc3 -->|Thuộc sở hữu của| ID
    
    Note over ID,Acc3: Hệ thống xịn (Như Keycloak) cho phép Link (Nối) vô số Account vào 1 Identity.<br/>Anh A Login bằng cách nào thì cũng quy về đúng 1 CSDL duy nhất.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Nguyên tắc "Bất Tử" của Identity**
> Tuyệt đối KHÔNG BAO GIỜ viết lệnh `DELETE` một Identity trong Database của hệ thống doanh nghiệp cốt lõi (Trừ khi Khách hàng B2C yêu cầu Xóa để tuân thủ luật GDPR của Châu Âu).
> Khi nhân viên nghỉ việc, chúng ta chỉ được phép `DISABLE` (Vô hiệu hóa) cái Identity đó. Xóa Identity sẽ gây ra Lỗi Khóa Ngoại (Foreign Key) dây chuyền lên hàng vạn Hóa đơn, Biên lai mà người đó từng xuất.

> [!CAUTION]
> **Tài khoản Dùng Chung (Shared Accounts) - Tội ác Bảo mật**
> Có một tài khoản tên là `admin_ketoan / pass: 123456`. Cả 5 người trong phòng Kế Toán đều dùng chung tài khoản này để đăng nhập (Tiết kiệm tiền mua License).
> Đây là ANTI-PATTERN. Nếu buổi sáng có 1 Lệnh Xóa Dữ liệu Kế toán được phát ra, Hệ thống Log chỉ ghi nhận `Tài khoản admin_ketoan đã thực hiện`. KHÔNG AI BIẾT CỤ THỂ 1 TRONG 5 NGƯỜI KIA LÀ THỦ PHẠM. 
> IAM bắt buộc: **Mỗi Người Phải Có Một Định Danh (Identity) Độc Lập.**

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Keycloak giải quyết bài toán "1 Identity - N Accounts" một cách hoàn hảo thông qua tính năng **Identity Brokering & Account Linking**.

- Bước 1: Keycloak tạo 1 Dòng dữ liệu Cốt lõi (Identity) với ID: `user-uuid-1234`.
- Bước 2: Anh A đăng nhập bằng Mật khẩu nội bộ. Keycloak tạo bảng Credential gắn pass vào `user-uuid-1234`.
- Bước 3: Ngày mai, Anh A bấm "Đăng nhập bằng Google". Keycloak nhận thấy Email Google trùng với Email của Anh A. Keycloak lập tức HỎI: *"Tôi thấy bạn đã có tài khoản. Bạn có muốn LINK (Nối) Google vào tài khoản cũ không?"*
- Nếu Anh A đồng ý, Keycloak ghi vào Bảng `Federated Identities`: Identity `user-uuid-1234` vừa nhận thêm 1 đường vào là Google_ID `google-8888`.
Kể từ giờ phút đó, Anh A Login bằng Pass hay bằng Google thì Keycloak đều phát ra đúng MỘT cái Token chứa cái ID gốc là `user-uuid-1234`. Developer Backend không bao giờ phải đau đầu xử lý Dữ liệu rác.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Machine Identity (Định danh Máy móc):**
  - Không phải Identity nào cũng là Con Người. Một Máy tính (Server A) gọi API sang Máy tính (Server B) cũng CẦN ĐƯỢC ĐỊNH DANH. 
  - Đây gọi là **Service Account** (Hoặc Bot/Daemon Account). Trong Keycloak, khi bạn bật tính năng `Service Account Enabled` trên 1 Client, Keycloak sẽ tự động sinh ra một Identity Ẩn nằm sau Client đó. Nó dùng Client ID / Client Secret để Login (Tương đương Username / Password của Người) và hệ thống Auditing vẫn ghi Log rành mạch: "Lệnh xóa do Bot A thực hiện lúc 2h sáng".

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Luật GDPR của Châu Âu có quyền "Right to be Forgotten" (Quyền được lãng quên). Khách hàng ép tôi phải XÓA TOÀN BỘ DATA của họ (Delete Identity). Nhưng nếu tôi xóa thì Database Hóa Đơn của tôi bị Nổ Lỗi Khóa Ngoại. Kiến trúc IAM giải quyết nghịch lý này như thế nào?**
- **Junior:** Bỏ khóa ngoại đi là xóa được.
- **Senior:** Tuyệt đối không phá vỡ tính toàn vẹn của CSDL. Ta dùng kỹ thuật **Data Anonymization (Ẩn danh hóa / Xóa Mềm Soft-delete)**.
Khi Khách hàng yêu cầu xóa:
- Ta Disable Identity đó (Không cho Login nữa).
- Ta Cập nhật đè toàn bộ thông tin cá nhân: Đổi Tên `Nguyễn Văn A` thành `Deleted_User_123`, Đổi Số điện thoại thành `0000000`, Xóa sạch Email.
Lúc này, cái Identity vẫn nằm đó (Để Hóa Đơn vẫn trỏ vào nó, không bị Lỗi Khóa Ngoại), nhưng KHÔNG AI BIẾT CÁI IDENTITY ĐÓ THỰC RA LÀ CỦA AI. Dữ liệu đã bị Ẩn Danh, tuân thủ 100% Luật GDPR mà không phá nát hệ thống. (Đồng thời phải xóa hết Accounts liên kết).

**2. Nếu hệ thống công ty có 1 Bảng `Users` cũ rích, đang để chung Cột `Password` với Cột `Tên, Tuổi, Giới tính`. Tại sao Kiến trúc IAM khuyên phải Tách Bảng ra (Identity Store vs Credential Store)?**
- **Junior:** Tách ra cho code cho lẹ.
- **Senior:** Việc gộp chung vi phạm quy tắc Bảo Mật Vùng (Security Zoning).
Cột `Password` (Dù đã Băm) là Trái Tim Nhạy Cảm. Cột `Tên/Tuổi` là Dữ liệu Công khai. 
Khi User truy cập Profile để đổi Tên, Lệnh SQL Update sẽ chọc vào bảng `Users`. Nếu Lập trình viên code ẩu (SQL Injection) trên form Đổi Tên, Hacker có thể Dump luôn cả Cột Password ra ngoài.
Kiến trúc chuẩn (Giống Keycloak) luôn Tách: Bảng `USER_ENTITY` (Chỉ chứa Tên Tuổi Email) và Bảng `CREDENTIAL` (Chứa Password Hash, Mã OTP). Bảng `CREDENTIAL` được khóa chặt chẽ, chỉ có Lõi của Module Đăng Nhập mới được phép chạm vào (Read/Write).

**3. Khái niệm "Shadow Account" (Tài khoản Bóng ma) xuất hiện khi nào trong Hệ thống Federated (Liên hiệp)? Nó tốt hay xấu?**
- **Junior:** Bóng ma chắc là hacker tạo ra để hack.
- **Senior:** Shadow Account hay "Just-In-Time (JIT) Provisioning" là một **Tính năng Kiến trúc Cực kỳ Thông Minh**.
Khi Công ty A xài Keycloak, kết nối với LDAP của Công ty B (Hoặc kết nối Google). Ban đầu, trong Database của Keycloak (Công ty A) HOÀN TOÀN KHÔNG CÓ DATA của nhân sự B.
Khi Nhân sự B gõ Email và Pass, Keycloak quăng cái đó sang LDAP để hỏi. LDAP bảo: "Pass đúng!".
Lúc này, Màn Ảo thuật xảy ra: Keycloak TỰ ĐỘNG TẠO NHANH một "Shadow Account" (Một cái Identity trống không) vào Database của mình, rồi Copy Tên/Email từ LDAP đắp vào cái bóng ma đó. (Quá trình này gọi là Just-In-Time Provisioning).
Nhờ có Shadow Account, Keycloak (A) giờ đây có Thể gán Role, gán Quyền, và Ghi Log cho Nhân sự B vào cái Database Nội bộ của mình, dù Pass thật của người đó vẫn nằm ở nhà Công ty B.

---

## 7. Tài liệu tham khảo (References)
- **NIST SP 800-63-3:** Digital Identity Guidelines.
- **Keycloak Documentation:** Identity Brokering and Account Linking.
