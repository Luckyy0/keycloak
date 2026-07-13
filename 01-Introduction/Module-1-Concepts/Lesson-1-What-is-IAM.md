# Lesson 1: Quản trị Định danh và Truy cập (What is IAM?)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Nắm vững Khái niệm Cốt lõi của IAM (Identity and Access Management). Đập bỏ tư duy "IAM chỉ là cái Form Đăng nhập". Hiểu vì sao IAM lại được ví như "Bộ Công An" và "Cửa Khẩu" của toàn bộ hệ sinh thái Công nghệ thông tin Doanh nghiệp.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sự ảo tưởng về Form Đăng Nhập
Khi nhắc đến User, 99% Junior Developer nghĩ ngay đến việc: "Tạo một cái Bảng `Users` trong MySQL, gắn thêm cột `Password_Hash`, viết API Login trả về JWT. XONG!". 
Tư duy này chỉ đúng ở các dự án sinh viên. Trong môi trường Doanh nghiệp (Enterprise), việc Đăng nhập chỉ chiếm 5% bức tranh khổng lồ mang tên **IAM (Identity and Access Management - Quản trị Định danh và Truy cập)**.

### 1.2. IAM thực chất là gì?
IAM là một Khung Kiến trúc Công nghệ (Framework of policies and technologies). Nhiệm vụ Tối thượng của nó được đúc kết trong 1 câu thần chú duy nhất:
> **"Đảm bảo ĐÚNG NGƯỜI có được ĐÚNG QUYỀN TRUY CẬP vào ĐÚNG TÀI NGUYÊN tại ĐÚNG THỜI ĐIỂM với ĐÚNG LÝ DO".**

Nó bao gồm 4 Trụ cột vô hình:
1. **Identity Management (Quản lý Định danh):** Bạn là ai? Tạo tài khoản, Khôi phục mật khẩu, Xóa tài khoản khi Nhân viên nghỉ việc (Lifecycle).
2. **Authentication - AuthN (Xác thực):** Chứng minh bạn là người đó. Bằng Password, bằng Vân tay (Biometrics), bằng Thẻ từ (FIDO2/MFA).
3. **Authorization - AuthZ (Phân quyền):** Bạn là Giám đốc, bạn được quyền ký duyệt. Ban là Nhân viên, bạn chỉ được quyền xem (RBAC/ABAC).
4. **Auditing & Governance (Kiểm toán & Quản trị):** Ghi hình lại TOÀN BỘ lịch sử: "Lúc 2h sáng, Ai là người bấm nút Xóa Dữ liệu Khách hàng?".

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bức tranh Toàn cảnh về Vòng đời Định danh (Identity Lifecycle) mà hệ thống IAM quản lý, hoàn toàn vượt xa quy trình `INSERT INTO users` thông thường:

```mermaid
graph TD
    subgraph "Vòng Đời Định Danh (Identity Lifecycle)"
        A[1. Onboarding <br/> Nhân sự mới vào Công ty] -->|Tạo User, Gửi Email cấp Pass| B[2. Provisioning <br/> Cấp Máy tính, Thẻ từ]
        B --> C[3. Authentication <br/> Đăng nhập hàng ngày]
        C -->|Thăng chức lên Trưởng Phòng| D[4. Privilege Escalation <br/> Thêm Role: Manager]
        D --> E[5. Auditing <br/> Theo dõi hành vi]
        E -->|Nghỉ việc / Bị đuổi| F[6. Offboarding & Deprovisioning]
        F -->|Thu hồi MỌI Token, Xóa Pass| G[Khóa Tài khoản vĩnh viễn]
    end
    
    Note over A,G: Tưởng tượng nếu không có Hệ thống IAM Tập trung (Centralized).<br/>Khi 1 Nhân viên bị đuổi việc, Bộ phận IT phải login vào 50 hệ thống App khác nhau<br/>để xóa bằng tay 50 cái tài khoản. Cực kỳ rủi ro và thiếu sót.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tư duy Tập trung hóa (Centralization)**
> Thực hành chuẩn mực nhất của IAM là Tuyệt đối không để cho mỗi Ứng dụng (Sales, HR, Kế toán) tự quản lý Bảng Users của riêng nó (Decentralization).
> Mọi ứng dụng (Client) BẮT BUỘC phải ủy thác (Delegate) niềm tin cho một "Trái tim" duy nhất (Identity Provider - Ví dụ: Keycloak). Hệ thống này đóng vai trò là "Sổ Hộ Khẩu Quốc Gia". Mọi công dân mạng nội bộ đều phải do Bộ Công An (Keycloak) cấp Căn cước.

> [!CAUTION]
> **Quyền Lực Rìa Lỗ Hổng (Privilege Creep)**
> Trong công ty, một Kế toán thuyên chuyển sang làm Sale. IT cấp thêm Quyền Sale cho người đó, nhưng LƯỜI thu hồi Quyền Kế Toán. Qua 5 năm làm việc, người này sở hữu TẤT CẢ các Quyền hạn cao nhất của mọi bộ phận (Creeping).
> IAM giải quyết vấn đề này bằng các luồng Phê duyệt tự động (Access Reviews). Cứ 6 tháng, hệ thống gửi Email ép Trưởng phòng Phê duyệt lại quyền, nếu không được duyệt, Quyền tự động bị tước (Time-bound access).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ánh xạ 4 trụ cột của IAM vào Hệ thống Keycloak:
1. **Identity Management:** Menu `Users` và `Groups` trong Keycloak. Chức năng `Required Actions` (Bắt người dùng phải đổi pass ở lần đăng nhập tới).
2. **Authentication:** Menu `Authentication Flows`. Nơi Admin kéo thả các Block Logic: "Nhập Pass xong thì Nhập OTP, OTP xong thì bắt Quét Khuôn mặt".
3. **Authorization:** Hệ thống RBAC (Realm Roles / Client Roles) và hệ thống mạnh mẽ nhất: `Keycloak Authorization Services` (UMA 2.0).
4. **Auditing:** Menu `Events`. Ghi chép 100% mọi hành động Đăng nhập Thành công, Thất bại, Nhập sai Pass bao nhiêu lần, IP nào gọi tới.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **CIAM (Customer IAM) vs EIAM (Enterprise IAM):**
  - **EIAM (Cho Nhân viên Nội bộ):** Ví dụ như Microsoft Active Directory. Yêu cầu bảo mật cực gắt. Phải có VPN, phải xài Thẻ YubiKey. Nhân viên có ghét cũng BẮT BUỘC phải xài.
  - **CIAM (Cho Khách hàng bên ngoài - B2C):** Ví dụ Tiki, Shopee. Yêu cầu: Trải nghiệm mượt mà, Đăng nhập siêu tốc bằng Google/Facebook (Social Login). Tránh hỏi quá nhiều thông tin (Hỏi nhiều khách bỏ đi). Hỗ trợ hàng triệu User cùng lúc (Scale khủng khiếp).
  - *Keycloak là một công cụ hiếm hoi trên thế giới Hỗ Trợ Tuyệt Đối cả 2 bài toán CIAM và EIAM trong cùng một bộ mã nguồn.*

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "Identity" (Định danh) và "Account" (Tài khoản) khác nhau như thế nào trong kiến trúc IAM?**
- **Junior:** Identity là tên, Account là cái dùng để đăng nhập.
- **Senior:** **Identity** là THỰC THỂ DUY NHẤT (Ví dụ: Bạn là anh Nguyễn Văn A, có số CMND là 123). Identity là duy nhất và không bao giờ thay đổi.
**Account (Tài khoản)** là Cách Thức mà Identity đó dùng để đi vào hệ thống. Một Identity có thể sở hữu HÀNG CHỤC Accounts. (Ví dụ: Anh A có Account truy cập vào Windows, Account truy cập Email, Account VPN). Nếu Hệ thống IAM đỉnh cao (Single Sign-On) được áp dụng, ta Hợp Nhất (Federate) tất cả Account thành 1 Account Duy Nhất đại diện cho Identity đó.

**2. Khái niệm "Deprovisioning" (Thu hồi tài nguyên) tại sao lại được đánh giá là Khó Cốt Lõi (Hardcore) hơn rất nhiều so với "Provisioning" (Cấp phát) trong IAM?**
- **Junior:** Thu hồi chỉ việc bấm xóa là xong, dễ hơn.
- **Senior:** Provisioning (Tạo mới) rất dễ: Bạn gọi API tạo User trên 10 hệ thống, cấp Pass cho họ.
Deprovisioning là cơn ác mộng vì **Tài sản Kế thừa (Orphaned Assets)**. Khi 1 Kế toán trưởng nghỉ việc, bạn bấm nút Xóa Account Kế toán trưởng. NHƯNG:
1. Hàng ngàn Hóa đơn do người đó tạo ra trong CSDL sẽ bị mất liên kết (Lỗi Khóa Ngoại - Foreign Key Null).
2. Chiếc Laptop người đó đang cầm có chứa Dữ liệu Nội bộ.
3. Các Cronjob/Dịch vụ chạy ngầm mang tên người đó (Service Accounts) sẽ lập tức sụp đổ.
Một hệ thống IAM đúng nghĩa không xóa Database ngay, mà thực hiện "Vô hiệu hóa" (Disable/Suspend), chuyển quyền sở hữu (Transfer Ownership) các file Document, Hủy mọi Session Token hiện tại trên Redis, và Lưu trữ Log (Archiving) vĩnh viễn trước khi chấm dứt vòng đời.

**3. IAM giải quyết bài toán "Siloed Systems" (Hệ thống biệt lập) như thế nào trong Quá trình Sát nhập Công ty (M&A)?**
- **Junior:** Nó copy database của công ty kia sang công ty này.
- **Senior:** Giải pháp không phải là Gộp Database (Việc này tốn hàng năm trời và rủi ro cực lớn). 
IAM giải quyết bằng **Identity Federation (Liên hiệp Định danh)**. 
Ví dụ: Công ty A mua Công ty B. Thay vì bắt nhân viên Công ty B tạo lại tài khoản bên A. Máy chủ IAM của A thiết lập niềm tin (Trust Relationship) qua giao thức SAML/OIDC với Máy chủ IAM của B. 
Nhân viên B bấm Login vào hệ thống A, hệ thống A tự động đá họ về trang Login quen thuộc của B. Login xong B phát Token cho A. Mọi thứ được kết nối trong 1 nốt nhạc mà không cần đụng 1 dòng code sửa Database.

---

## 7. Tài liệu tham khảo (References)
- **NIST SP 800-63:** Digital Identity Guidelines.
- **Gartner:** Identity and Access Management (IAM).
- **Keycloak Documentation:** Keycloak Concepts.
