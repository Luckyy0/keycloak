# Lesson 3: Mũi Kim Xuyên Giáp (Fine-Grained Authorization)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Bạn đã học RBAC và ABAC. Tuy nhiên, ở tầm vóc doanh nghiệp, việc phân quyền không chỉ dừng lại ở mức "Cho phép truy cập Trang Admin hay không". Ta cần phân quyền sâu đến tận cấp độ chi tiết như "Cho phép chỉnh sửa Dòng Dữ Liệu Số 5 của Bảng Báo Cáo". Tầm nhìn vi mô này được gọi là **Fine-Grained Authorization (Phân quyền hạt mịn)**. Keycloak hỗ trợ kiến trúc chuẩn **UMA (User-Managed Access)** để bạn xây dựng hệ thống phân quyền ở mức đỉnh cao này.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Coarse-Grained vs Fine-Grained
1. **Coarse-Grained (Phân Quyền Thô):**
   - **Bản chất:** Giống như ông bảo vệ canh cửa chính tòa nhà. Chỉ xét xem bạn có thẻ nhân viên không.
   - **Mức độ kiểm soát:** Cấp độ Tên miền, Cấp độ Ứng dụng, Cấp độ Module (VD: Cho phép vào `/admin`).
   - **Công cụ thường dùng:** RBAC (Chỉ kiểm tra chuỗi Role "admin").

2. **Fine-Grained (Phân Quyền Mịn):**
   - **Bản chất:** Giống như các tủ sắt chứa hồ sơ có khóa riêng bên trong tòa nhà. Dù bạn đã vào được tòa nhà, nhưng bạn chỉ mở được tủ số 5 do bạn quản lý.
   - **Mức độ kiểm soát:** Cấp độ Bản ghi dữ liệu (Data Row), Cấp độ Trường dữ liệu (Data Field), Cấp độ Nút bấm cụ thể (Button).
   - **Công cụ thường dùng:** ABAC, PBAC (Policy), UMA. (VD: User A chỉ được Sửa Bài Viết ID=10).

### 1.2. Kiến Trúc 4 Cột Trụ Của Keycloak Authorization (UMA Model)
Để làm được phân quyền hạt mịn, tính năng Authorization Services của Keycloak chia thế giới ra làm 4 khái niệm (cực kỳ quan trọng, phải thuộc lòng):
1. **Resource (Tài nguyên):** Cái "Cục dữ liệu" mà bạn cần bảo vệ. VD: Máy chủ, Một album ảnh, Một bài báo (ID=101), Số tài khoản ngân hàng.
2. **Scope (Phạm vi thao tác):** Hành động bạn định làm với Cục tài nguyên đó. VD: `read` (xem), `write` (sửa), `delete` (xóa), `approve` (duyệt).
3. **Policy (Chính sách/ Quy tắc):** Các luật lệ kiểm tra an ninh (RBAC, ABAC). VD: "Chỉ sếp mới được duyệt", "Chỉ tác giả mới được xóa".
4. **Permission (Quyền hạn):** Mảnh ghép thần thánh kết hợp 3 thằng trên lại với nhau! Nó phán quyết: *"Tài nguyên (Resource) A, đối với Hành động (Scope) B, sẽ chịu sự cai quản và thẩm định của Bộ Luật (Policy) C"*.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Token Xuyên Qua Kiến Trúc 4 Cột Trụ Của Keycloak UMA:

```mermaid
flowchart TD
    A[Khách Hàng Yêu Cầu: GET /api/photo/1001] --> B[API Gửi Yêu Cầu Lên Keycloak: Thằng này có quyền Scope 'read' trên Resource 'Photo-1001' không?]
    
    B --> C[Keycloak Mở Cấu Trúc Khớp Lệnh Lõi]
    C --> D{1. Tìm Resource ID?}
    D -- Khớp Khối Tài Nguyên 'Photo-1001' --> E{2. Có Thằng Permission Nào Trói Vào Nó Bằng Hành Động Scope 'read' Không?}
    
    E -- Có (Photo-Read-Permission) --> F[3. Chạy Vào Permission Kéo Danh Sách Các Policy Đang Cột Ở Trong Nó]
    
    F --> G{4. Policy 1 (RBAC): Đòi Role Viewer. Có Không?}
    G -- Pass Role --> H{5. Policy 2 (ABAC): Đòi Giờ Hành Chính. Có Khớp Không?}
    
    H -- Pass Policy --> I[Quyết Định Cuối Cùng: Decision = PERMIT]
    I --> J[Keycloak Nhả Token Có Cấp Quyền RPT (Requesting Party Token) Chứa Permission Bên Trong]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Trải Nghiệm Mạch (Tái Sử Dụng Policy Tránh Trùng Lặp)**
> **Tội Ác Thiết Kế:** Bạn muốn cấp quyền "Sửa Hóa Đơn" và "Sửa Bài Báo" cho ông Kế Toán Trưởng. Lập trình viên thiết kế Permission A (Sửa Hóa Đơn) và tạo mới hoàn toàn Policy tên là "KiemTraKeToan1". Xong qua bên Permission B (Sửa Bài Báo), lại cặm cụi đi tạo mới một cái Policy y chang tên là "KiemTraKeToan2".
> **Hậu Quả:** Khi quy định thay đổi, Sếp đuổi Kế Toán Trưởng và giao quyền cho ông Giám Đốc Tài Chính. Bạn phải lục tung toàn bộ hệ thống để sửa lại hàng trăm cái Policy rác bị copy-paste rải rác.
> **Biện Pháp Sống Còn Lớp Bảo Vệ (Decoupling):** Triết lý số 1 của UMA Keycloak là TÁCH RỜI LUẬT (Policy) KHỎI QUYỀN (Permission). 
> Cùng một cái Luật kiểm tra (Policy: KiemTraSuaTaiChinh), bạn có thể tái sử dụng để Móc nó vào hàng trăm cái Permission khác nhau. Sửa 1 lần ở Policy, trăm Permission dưới đáy lập tức tự động update theo! Hãy Code Policy như những mảnh Lego rời!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cơ Chế Phân Quyền Hạt Mịn 4 Bước Kinh Điển Trong Giao Diện Keycloak Admin:
*(Lưu ý: Tính năng này chỉ mở ra khi bạn truy cập vào cấu hình của 1 Client có gạt công tắc Bật Authorization Services ON)*
1. Vào Tab **Authorization** của Client. Sang mục **Scopes**. Bấm Create tạo một cục tên `view_secret_report`.
2. Sang mục **Resources**. Bấm Create tạo cục Tài Nguyên tên `Quy-1-Financial-Report-2023`. Trong lúc tạo, móc cái Scope `view_secret_report` vừa nảy vào bụng nó.
3. Sang mục **Policies**. Bấm Create. Chọn Loại (Type) là **User**. Đặt tên: `Only-Boss-John-Policy`. Chọn thẳng tên tài khoản sếp John trong danh sách DB thả xuống. Save lại.
4. Sang mục quyền lực nhất: **Permissions**. Bấm Create. Chọn Loại là **Scope-Based** (Dựa theo hành động). 
   - Name: `Allow-Boss-View-Report-Permission`.
   - Cột Resource móc chọn: `Quy-1-Financial-Report-2023`.
   - Cột Scopes móc chọn: `view_secret_report`.
   - Cột Policies móc chọn: `Only-Boss-John-Policy`.
5. BÙMMM! Kiến trúc hạt mịn đã giăng lưới thành công. Bất kỳ ai yêu cầu Action View vào Báo cáo này, thằng Permission sẽ dội lệnh gọi Policy kiểm tra. John pass, Admin trượt!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Hệ Thống Phân Quyền Hạt Mịn Của UMA, Khái Niệm Policy Và Permission Giao Thoa Nhau Ở Chỗ Nào? Nếu Cậu Bỏ Qua Việc Tạo Permission Mà Gắn Trực Tiếp Policy Vào Resource Thì Hệ Thống Có Chạy Được Không?**
- **Senior:** Dạ thưa sếp, Cực kỳ rõ ràng: 
  - **Policy** chỉ là một tập hợp các thuật toán IF-ELSE ngu ngốc (VD: If Role == Admin, If Tuoi > 18). Bản thân Policy không hề biết nó đang được dùng để bảo vệ Dữ Liệu Nào (Resource nào).
  - **Permission** chính là Sợi Dây Thừng để Trói cái Policy (Luật) vào cái Resource (Tài Nguyên) mục tiêu. Nó kết hôn 2 thế giới lại với nhau.
  - Về mặt Kiến trúc Core của Keycloak, ta KHÔNG THỂ vứt bỏ Permission để ép trực tiếp Policy vào Resource được. Permission là cầu nối bắt buộc (Mandatory Entity). Nếu không có mặt Permission, cái Policy cậu tạo ra mãi mãi trôi nổi như rác bộ nhớ, không có hiệu lực áp đặt lên bất kỳ bảng dữ liệu nào của hệ thống. Đây là sức mạnh của tính trừu tượng hóa!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Authorization Services - Architecture.
- **UMA 2.0 Spec:** User-Managed Access Profile.
