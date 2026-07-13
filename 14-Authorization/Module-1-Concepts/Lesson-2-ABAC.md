# Lesson 2: Phân Quyền Theo Thuộc Tính (ABAC - Attribute-Based Access Control)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khi RBAC (Role-Based) bó tay trước những yêu cầu ngữ cảnh như "Chỉ cho phép nhân viên truy cập hệ thống kho trong giờ hành chính từ Thứ 2 đến Thứ 6" hoặc "Chỉ sếp có tuổi > 30 mới được duyệt chi". Lúc này, thế giới bảo mật gọi tên người hùng **ABAC**. Bài học này giúp bạn hiểu cách dùng Thuộc tính (Attribute) để bẻ khóa những logic phân quyền phức tạp nhất.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. ABAC Bản Chất Là Gì?
Attribute-Based Access Control (ABAC) là mô hình cấp quyền động dựa trên việc tính toán tổng hợp các **Thuộc tính (Attributes)** tại thời điểm người dùng yêu cầu truy cập.
ABAC sử dụng các thuật toán If-Else đánh giá 4 nguồn thuộc tính (Attributes) sau:
1. **Subject Attributes (Của Người Dùng):** Tuổi, Chức danh, Phòng ban, Cấp độ bảo mật (Clearance).
2. **Resource Attributes (Của Tài Nguyên):** Bài viết này được gán nhãn "Tuyệt Mật", Tác giả của báo cáo này là ai?
3. **Action Attributes (Của Hành Động):** Hành động đang thực hiện là GET (Đọc), PUT (Sửa) hay DELETE (Xóa)?
4. **Environment/Context Attributes (Của Môi Trường Ngữ Cảnh):** Mấy giờ rồi? IP của người dùng có nằm ở Mỹ không? Hôm nay có phải là ngày lễ không?

- **Ưu điểm:** Cực kỳ linh hoạt, đáp ứng mọi tình huống nghiệp vụ hóc búa nhất (Fine-Grained). Không bị giới hạn bởi các định nghĩa tĩnh như Role.
- **Nhược điểm:** Khó thiết lập. Thuật toán kiểm tra chạy phức tạp nên có thể làm giảm Performance (độ trễ) của hệ thống so với RBAC truyền thống.

### 1.2. ABAC vs RBAC
- **RBAC (Role):** Giống như đưa cho bạn cái Bằng Cử Nhân (Role). Chỉ cần giơ cái bằng ra là qua cửa, không ai quan tâm bạn bao nhiêu tuổi hay đang đứng ở đâu. (Cứng nhắc).
- **ABAC (Attribute):** Giống như gặp cảnh sát giao thông. Cảnh sát kiểm tra (1) Tuổi của bạn (Subject), (2) Dung tích xi-lanh xe bạn đang lái (Resource), (3) Đường này đang cấm giờ cao điểm không (Environment). Tất cả đúng khớp mới được chạy qua. (Mềm dẻo).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Token Đem Attribute Bơm Vào Động Cơ ABAC Keycloak Quyết Định:

```mermaid
flowchart TD
    A[User (Attribute: Department=Sales, IP=Office) Yêu cầu xóa Báo Cáo X (Attribute: Type=Financial)] --> B[Client App Gửi Request Lên Policy Enforcer (PEP)]
    
    B --> C[PEP Chuyển Giao Data Về Keycloak Authorization Server (PDP)]
    
    C --> D{Chạy Bộ Máy Đánh Giá ABAC Policy}
    D --> E{Rule 1: User Department Có Bằng 'Finance' Không?}
    
    E -- Trả Về 'False' Do Là Sales --> F[Máy Chủ Đánh Giá Trượt]
    E -- (Ví dụ Khác Trả Về True) --> G{Rule 2: Môi Trường (Giờ) Có Phải Giờ Hành Chính 8h-17h?}
    
    F --> H[Keycloak Dập Dòng Quyết Định (Decision) Là DENY]
    H --> I[Backend Trả 403 Forbidden]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tối Ưu Tốc Độ Máy Chủ (Performance) Bằng Phễu Lọc Hỗn Hợp Trọng Lực (RBAC + ABAC)**
> **Tội Ác Thiết Kế:** Do thấy ABAC hay quá, bạn vứt bỏ hoàn toàn RBAC. Mọi quyền từ lớn đến bé bạn đều viết Policy ABAC kiểm tra hầm bà lằng các kiểu.
> **Hậu Quả:** ABAC tính toán rất tốn RAM và CPU của máy chủ. Khi bạn có 10.000 User cùng bấm vào nút "Xem bài viết", máy chủ gồng mình tính toán IP, Giờ giấc, Nhãn dữ liệu 10.000 lần. Server Keycloak bị treo cứng "Out of Memory", API Backend Timeout.
> **Biện Pháp Sống Còn Lớp Bảo Vệ:** Bạn phải thiết kế theo hình Phễu (Trục Đứng Mạch Nhựa):
> - **Lớp 1 (Vòng gửi xe - Coarse-grained):** Dùng RBAC. Nếu thằng này còn chả có Role "Editor", đuổi cổ nó ngay lập tức bằng lệnh Check String nhanh như chớp. 
> - **Lớp 2 (Kiểm duyệt vòng trong - Fine-grained):** Những ai đã đi lọt cửa Role "Editor", ta mới bắt đầu lôi động cơ ABAC ra tính toán xem "Mày có được sửa bài báo này trong khung giờ 20h đêm nay không?". Bằng cách này bạn tiết kiệm được 90% hiệu năng thừa mứa cho server!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Thiết Kế Chiến Lược Thuộc Tính (Attribute) Dưới Database Người Dùng Của Keycloak Để Chạy Được ABAC:
1. Vào menu **Users**, chọn một người dùng tên Nguyễn Văn A.
2. Sang tab **Attributes** (Thuộc tính).
3. Thêm một cặp Key-Value siêu cấp: Key = `clearance_level`, Value = `top_secret`. Lưu lại.
4. Ở tab Clients của Keycloak, khi bạn kích hoạt Authorization Services, bạn sẽ tạo ra một Policy (Quy tắc) loại hình là "User-Based". 
5. Thay vì check Role như cũ, bạn code/chọn Logic kiểm tra rằng: "Nếu trường `clearance_level` của thằng User này bằng chữ `top_secret` thì tao mới cho Decision trả về PASS".
6. Mở rộng thêm sự tàn bạo: Bơm tiếp một thuộc tính Custom vào Token có tên là IP Address. Bạn tạo thêm 1 Policy thứ 2 loại Script (JavaScript) hoặc Rule kiểm tra: Nếu Thời Gian hiện tại (Time) mà > 18:00 (Hết giờ làm), thì lập tức Decision rớt đài về Fail (DENY).
7. Gắn 2 cái Policy đó chung vào bảo vệ 1 Resource Báo Cáo. Bạn đã có hệ thống ABAC đỉnh cao thế giới.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Khách Hàng Yêu Cầu Một Tính Năng: "Một Bác Sĩ Chỉ Được Xem Hồ Sơ Bệnh Án Của Bệnh Nhân NẾU Bệnh Nhân Đó Đang Được Xếp Lịch Khám Trong Ngày Hôm Nay Tại Phòng Khám Của Bác Sĩ Đó. Qua Ngày Hôm Sau Bác Sĩ Không Còn Quyền Truy Cập Nữa". Cậu Chọn Thiết Kế Access Control Theo RBAC Hay ABAC? Tại Sao Và Triển Khai Thế Nào?**
- **Senior:** Dạ thưa sếp, Yêu cầu này dính tới 2 yếu tố Ngữ cảnh cực kỳ biến động: "Ngày Khám (Thời Gian - Environment)" và "Mối quan hệ động BácSĩ-BệnhNhân (Resource Attribute)". Khẳng định 100% RBAC không thể làm được (Vì chẳng lẽ mỗi ngày lại đi gỡ Role và add Role mới cho Bác Sĩ, làm vậy máy nổ banh xác mất).
Em bắt buộc chọn **ABAC (Attribute-Based Access Control)** để xử lý. Triển khai em sẽ để RBAC làm lớp vỏ, check Role "Doctor" trước. Sau đó xuống Tầng Backend Resource (Bệnh Án API), em sẽ thiết lập PDP (Policy Decision Point) tính toán ABAC:
- **Condition 1:** Check Subject ID (Doctor_ID) có khớp với cột `assigned_doctor_id` trong Bảng Cuộc_Hẹn_Hôm_Nay.
- **Condition 2:** Check Timestamp Environment (Date_Now) có đúng bằng `appointment_date` hay không.
Nếu 2 Attribute này khớp lệnh True, máy chủ mới nhả Dữ liệu. Xong một tính toán ABAC cực bén không dính rác Role!

---

## 6. Tài liệu tham khảo (References)
- **NIST ABAC Guide:** Guide to Attribute Based Access Control.
- **Keycloak Documentation:** Authorization Services - Policies.
