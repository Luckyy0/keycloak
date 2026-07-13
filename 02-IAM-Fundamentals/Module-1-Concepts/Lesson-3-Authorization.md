# Lesson 3: Phân quyền (Authorization - AuthZ)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trả lời cho câu hỏi: *"Bạn được phép làm gì?"* (What are you allowed to do?). Phá vỡ sai lầm "Gộp chung AuthN và AuthZ vào 1 cục" và làm quen với các Học thuyết Phân quyền Cấp cao (RBAC, ABAC, PBAC).
> *Thuật ngữ viết tắt chuẩn quốc tế của Authorization là **AuthZ**.*

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. AuthN vs AuthZ (Đừng bao giờ nhầm lẫn)
- **AuthN (Xác thực):** Kiểm tra xem cái Vé máy bay bạn cầm có phải là Hàng Thật không, tên trên Vé có khớp tên bạn không. Khi bạn qua cổng, bạn ĐÃ AUTHENTICATED.
- **AuthZ (Phân quyền):** Bạn ĐÃ VÀO TRONG sân bay. Nhưng vé của bạn là vé Hạng Phổ Thông (Economy). Bạn đi vào Phòng chờ Thương gia (VIP Lounge). Bảo vệ chặn bạn lại: *"Anh được vào sân bay, nhưng KHÔNG CÓ QUYỀN vào phòng này"*. Đó chính là AUTHORIZATION.

### 1.2. Các Trường Phái Phân Quyền Kinh Điển
Sự tiến hóa của AuthZ đi từ Đơn giản đến Cực kỳ phức tạp:
1. **MAC/DAC (Mandatory / Discretionary Access Control):** Cũ kỹ. File của ai người nấy có quyền (Linux File System). Ít dùng trong Web.
2. **RBAC (Role-Based Access Control):** Mọi thứ dựa vào **VAI TRÒ (Role)**. Bạn là `MANAGER` -> Bạn được Xóa bài viết. Bạn là `USER` -> Chỉ được Xem. Đây là tiêu chuẩn 90% Website thế giới đang dùng.
3. **ABAC (Attribute-Based Access Control):** Đỉnh cao của Phân quyền Động. Không quan tâm Role là gì. Nó kiểm tra **THUỘC TÍNH (Attributes)**. 
   - Ví dụ: *"Chỉ cho phép Tài khoản Khách hàng Rút tiền NẾU số dư > 0 VÀ Giờ hiện tại là Giờ Hành Chính VÀ IP đăng nhập đến từ Việt Nam"*. Rõ ràng RBAC không thể diễn đạt được chữ "Giờ Hành Chính". ABAC làm được việc đó.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Kiến trúc UMA (User-Managed Access) và Luồng Quyết định (Decision Workflow):

```mermaid
graph TD
    subgraph "Hệ thống Bệnh Viện (Tách bạch Code và Quyền)"
        API[API Server: Nơi lấy Hồ sơ Bệnh Án <br/> (Policy Enforcement Point - PEP)]
        KC{Keycloak: Máy chủ Phán Xử <br/> (Policy Decision Point - PDP)}
        
        Doc(Bác sĩ Alice)
        
        Doc -->|Xin Lấy Bệnh Án số 123| API
        API -->|Khoan đã! Tạm Dừng! Gửi thông tin Alice sang Keycloak hỏi ý kiến| KC
        
        Note over KC: Keycloak chạy Logic ABAC:<br/>1. Alice có phải là Bác sĩ không? (Có)<br/>2. Alice có được phân công khám Bệnh nhân 123 không? (Không)<br/>=> KẾT LUẬN: TỪ CHỐI.
        
        KC -->|Trả lệnh Từ Chối (HTTP 403)| API
        API -->|Đá Bác sĩ Alice ra ngoài| Doc
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Quy tắc Tách rời Logic Quyền ra khỏi Code API (Decouple AuthZ from Code)**
> **Cách Làm Tồi:** Code Java Backend nhét đầy lệnh `if (user.role == "ADMIN") { xóa_dữ_liệu(); }`. Khi Giám đốc yêu cầu "Thêm role SUB-ADMIN cũng được xóa dữ liệu", Dev phải SỬA CODE JAVA, Re-build lại Server, gây gián đoạn hệ thống.
> **Cách Làm Kiến Trúc Sư:** Backend KHÔNG CHỨA 1 chữ `if` nào về quyền. Nó ủy thác (Delegate) hoàn toàn việc ra quyết định (Decision) cho Keycloak Authorization Services (PDP). Khi Giám đốc đổi ý, SysAdmin chỉ cần lên giao diện Web của Keycloak, Sửa cái Rule (Policy), Bấm Save. Toàn bộ Hệ thống ngay lập tức nhận luật mới mà KHÔNG CẦN CHẠM 1 DÒNG CODE BACKEND NÀO. Đó gọi là **PBAC (Policy-Based Access Control)**.

> [!CAUTION]
> **Quá tải Role (Role Explosion)**
> Trong mô hình RBAC, cứ mỗi một tính năng mới ra đời, bạn lại tạo 1 Role mới (`CAN_EDIT_POST`, `CAN_DELETE_POST`, `CAN_APPROVE_POST`). Một thời gian sau, số lượng Role phình to lên HÀNG NGÀN Roles. Khối lượng Token JWT phình to (Bloat) quá 8KB, gây sập Web Server (Lỗi Header Too Large). 
> **Giải pháp:** Phải chia Nhóm (Groups) và Map Roles, hoặc chuyển hẳn sang ABAC để giảm số lượng Role xuống.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Trong Keycloak, để áp dụng **Authorization Services (UMA 2.0)**:
1. Bạn bật cờ `Authorization Enabled` trong một Client.
2. Keycloak sẽ mở ra 1 Táp mới gồm: `Resources` (Tài nguyên: VD File Lương), `Scopes` (Hành động: Đọc, Ghi), `Policies` (Luật: Chỉ giờ hành chính), `Permissions` (Cột dây Luật vào Tài nguyên).
3. Backend (Spring Boot) của bạn chỉ cấu hình 1 thư viện `keycloak-policy-enforcer`. Nó tự động gọi về Keycloak để phán xử mỗi khi có Request tới API.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Ủy quyền Cấp độ Người dùng (Delegated Authorization - Trái tim của OAuth2):**
  - **AuthZ tĩnh (Admin cấp quyền):** Giám đốc (Admin) tự quyết định Bác sĩ A được xem Bệnh án B.
  - **AuthZ ủy quyền (User-Managed):** Bệnh nhân (User) có dữ liệu trên Ứng dụng Y tế. Ứng dụng Tập Gym bên ngoài XIN PHÉP Bệnh nhân cho phép nó đọc Nhịp Tim từ Ứng dụng Y tế. Bệnh nhân bấm "TÔI ĐỒNG Ý" (Consent Screen). Lúc này, QUYỀN TRUY CẬP ĐƯỢC CẤP BỞI CHÍNH NGƯỜI DÙNG, chứ không phải do Admin. Đây chính là luồng **OAuth 2.0** tiêu chuẩn.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. "Authentication luôn đi trước Authorization". Khẳng định này có trường hợp nào bị sai hoặc bị đảo ngược không?**
- **Junior:** Nó luôn đúng, phải đăng nhập mới biết có quyền gì.
- **Senior:** Về mặt Lô-gic Hệ thống thì đúng (Phải biết Anh là ai rồi mới soi Quyền).
Nhưng về mặt **Trải nghiệm Kiến trúc (User Experience / OIDC Flow)**, có những lúc **Authorization kích hoạt Authentication**.
Ví dụ: Bạn vào trang web Ẩn danh (Guest). Bạn bấm Nút "Thanh Toán" (Hành động yêu cầu Quyền Hạn - AuthZ). Máy chủ Web Đánh giá Quyền (Policy Enforcement Point) thấy bạn CHƯA ĐỦ QUYỀN. Nó lập tức ĐÁ BẠN SANG MÀN HÌNH LOGIN (Ép buộc Authentication). Nghĩa là Cú Kích Hoạt Quyền (AuthZ Exception) là nguyên nhân sinh ra Luồng Xác thực (AuthN).

**2. Nếu hệ thống xài RBAC (Role-Based). Khi User đang đăng nhập, Admin ở nhà Xóa cmn Role của User đó. Hệ thống có ngay lập tức cấm User đó không? Tại sao?**
- **Junior:** Có, vì Role bị xóa rồi.
- **Senior:** **KHÔNG THỂ CẤM NGAY LẬP TỨC** (Nếu dùng JWT Stateless).
Vì JWT là "Tờ vé có thời hạn" (15 phút). Lúc User đăng nhập, Keycloak đóng dấu `role=ADMIN` vào ruột JWT và gửi cho User.
5 phút sau Admin xóa Role trên Database của Keycloak. NHƯNG User VẪN ĐANG CẦM TỜ JWT ĐÓ đập vào API. API chỉ verify Chữ ký RSA (Hợp lệ) và đọc ruột (Thấy có chữ ADMIN). API không chọc về Keycloak nên KHÔNG BIẾT Role đã bị xóa. User đó vẫn tiếp tục lộng hành thêm 10 phút nữa cho đến khi Token hết hạn.
Để fix triệt để: Hoặc là Chuyển sang dùng Token có Trạng thái (Opaque Token / Introspection), hoặc là Kiến trúc API phải thiết kế "Danh sách Đen" (Token Revocation List / Redis) để kiểm tra chéo (Đánh đổi tốc độ lấy sự An toàn).

**3. Phân biệt PEP (Policy Enforcement Point) và PDP (Policy Decision Point)? Nginx / API Gateway thường đóng vai trò nào?**
- **Junior:** Nginx là chặn người dùng, Keycloak là cấp quyền.
- **Senior:** 
- **PDP (Điểm Đưa Ra Quyết Định):** Nơi chứa chất xám, não bộ, CSDL luật lệ (Database Policies). Nó nhận Câu hỏi và Trả lời YES/NO. (Chính là Máy chủ Keycloak).
- **PEP (Điểm Thực Thi Quyết Định):** Nơi đóng vai trò "Bảo vệ đứng gác". Nó mù tịt về luật lệ, nó chỉ biết cầm CMND của khách, Chạy vào hỏi Ông Chủ (PDP). Ông chủ bảo Đuổi, nó Rút súng bắn (HTTP 403). Ông chủ bảo Vào, nó Mở cửa (HTTP 200).
- **Nginx / API Gateway / Spring Enforcer** đóng vai trò là **PEP**. Việc rạch ròi 2 thành phần này giúp hệ thống Scale (Mở rộng) cực tốt. Ta có 100 cái PEP rải rác khắp thế giới, nhưng chỉ cần 1 cụm PDP Trung tâm để quản trị luật lệ.

---

## 7. Tài liệu tham khảo (References)
- **NIST:** Role Based Access Control (RBAC) and Attribute Based Access Control (ABAC).
- **Kantara Initiative:** User-Managed Access (UMA) 2.0.
