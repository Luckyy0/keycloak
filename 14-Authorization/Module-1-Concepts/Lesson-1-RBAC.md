# Lesson 1: Phân Quyền Theo Vai Trò (RBAC - Role-Based Access Control)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** RBAC là mô hình phân quyền "quốc dân" xuất hiện ở mọi framework. Mặc dù dễ học, nhưng RBAC thường bị thiết kế sai cách, dẫn đến hiện tượng "Bùng nổ Role" (Role Explosion). Bài học này không dạy bạn cách tạo Role (vì đã học ở Chapter 8), mà dạy bạn tư duy thiết kế RBAC chuẩn mực trong hệ thống lớn.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. RBAC Bản Chất Là Gì?
Role-Based Access Control (RBAC) là mô hình cấp quyền truy cập dựa trên **Vai trò (Role)** của người dùng trong tổ chức. 
Thay vì gán trực tiếp Quyền (Permission) cho Người dùng (User): `User A -> Quyền Xóa Báo Cáo`, người ta sẽ gán qua một cầu nối trung gian gọi là Role: `User A -> Role Kế Toán -> Quyền Xóa Báo Cáo`.
- **Ưu điểm:** Dễ hiểu, dễ cài đặt. Phù hợp với cấu trúc phòng ban công ty (Admin, User, Manager).
- **Nhược điểm:** Cực kỳ thiếu linh hoạt. Không thể xử lý ngữ cảnh (Context). (Ví dụ: "Kế toán chỉ được xóa báo cáo do chính họ tạo ra" -> RBAC thuần túy hoàn toàn bó tay).

### 1.2. Vấn Nạn "Role Explosion" (Bùng Nổ Vai Trò)
Đây là "căn bệnh ung thư" của các lập trình viên khi dùng RBAC.
Khi gặp yêu cầu mới từ Sếp: "Tạo quyền cho ông Kế toán nhưng ổng chỉ được xem báo cáo khu vực Miền Bắc thôi".
- **Lập trình viên non tay:** Sẽ lao vào tạo ngay một Role mới tinh là `KeToan_MienBac_XemBaoCao`.
- **Hậu quả:** 1 năm sau, hệ thống sinh ra 500 cái Roles: `KeToan_MienNam_Sua`, `Admin_Tầng1_Delete`,... Việc quản lý trở thành thảm họa không thể cứu vãn. Token phình to quá giới hạn Header HTTP (4KB).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Token Bơm Data RBAC Vào Backend API:

```mermaid
flowchart TD
    A[User Request Login] --> B[Keycloak Load Danh Sách Roles Của User]
    B --> C{Mapper: Role Mapper}
    C --> D[Nhúng Danh Sách Roles Vào Mảng 'realm_access.roles' Trong JWT Token]
    
    D --> E[User Nhận Token Mang Đi Gọi API Backend]
    
    E --> F[API Backend (Spring Boot / Nodejs)]
    F --> G{Trình Đọc Lệnh Nội Bộ: @RolesAllowed('admin')}
    G -- Token Có Chữ 'admin' --> H[Cho Phép Vào Tầng Service Data]
    G -- Token Không Có 'admin' --> I[Ném Lỗi HTTP 403 Forbidden - Cấm Truy Cập]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Thiết Kế (Chỉ Dùng RBAC Cho Phân Quyền Thô - Coarse-Grained)**
> **Quy Tắc Vàng:** Trong hệ thống hiện đại, KHÔNG BAO GIỜ dùng Role để chứa Data hoặc Ngữ Cảnh (Context). 
> - **Tội Ác Thiết Kế:** Tạo Role có tên là `admin_công_ty_A`. Nếu có 100 công ty, bạn phải sinh 100 Role? Thật tệ hại.
> - **Biện Pháp Sống Còn:** Role chỉ mang tính chất **Chức Danh**. Bạn chỉ nên tạo Role tên là `Company_Admin`. Sau đó, việc kiểm tra xem User này có phải là Admin của "Công ty A" hay không thì KHÔNG PHẢI việc của RBAC. Đó là việc của Logic Backend (so sánh ID công ty) hoặc chuyển sang dùng ABAC/PBAC. RBAC chỉ dùng làm cánh cửa sơ cấp ngoài cùng (Coarse-Grained). 

> [!WARNING]
> **Hiểm Họa Token Phình To Kéo Sập Server Lỗi 431**
> Khi số lượng Role của một người dùng quá lớn (Ví dụ: Một tổng giám đốc kiêm nhiệm 50 cái Role), mảng String Roles bị nhét vào JWT Token sẽ làm kích thước Token vượt quá 4KB. Lúc này, các Reverse Proxy như Nginx hay Apache sẽ lập tức văng lỗi `HTTP 431 Request Header Fields Too Large` chặn sạch mọi yêu cầu của sếp tổng!
> **Khắc Phục:** 
> 1. Xóa bớt Role thừa bằng cách dùng Composite Role (Gộp nhóm).
> 2. Đừng nhét tất cả Role vào Access Token. Dùng Token Scope để Client chỉ yêu cầu nhóm Role nó cần.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cơ Chế RBAC Mềm Mỏng Thông Qua Groups Chống "Role Explosion":
1. Bạn có 3 Role cơ bản trên Keycloak: `read_report`, `write_report`, `delete_report`. (Đây là quyền thô, không gắn tên chức danh vào nó).
2. Tới menu **Groups**, bạn tạo Group `Phòng Kế Toán`.
3. Tới Tab Role Mapping của Group `Phòng Kế Toán`, bạn gán 2 Role `read_report` và `write_report` vào Group này.
4. Tới menu **Users**, bạn gán nhân viên Nguyễn Văn A vào Group `Phòng Kế Toán`.
5. Vậy là A tự động có 2 Role. Nếu A chuyển sang phòng Giám Đốc, bạn chỉ cần tháo A khỏi Group Kế Toán và ném qua Group Giám Đốc. Không cần phải tự đi mò 100 cái Role để tháo gỡ thủ công. Đây là thiết kế chuẩn mực RBAC cấp độ doanh nghiệp lớn.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Sếp Yêu Cầu Tính Năng Cấp Quyền Đọc/Sửa Bài Viết Báo Chí. Ban Đầu Cậu Thiết Kế RBAC Với 2 Role: 'Editor' (Được Sửa) Và 'Reader' (Được Đọc). Nhưng Sếp Đổi Yêu Cầu: "Thằng Editor Chỉ Được Sửa Bài Báo Do Chính Tay Nó Viết Ra Thôi, Bài Của Đứa Editor Khác Thì Cấm Chạm Vào". Cậu Sẽ Giải Quyết Bằng Mô Hình RBAC Này Thế Nào?**
- **Junior:** Dạ dễ ợt, em sẽ tạo ra role động theo ID bài viết. Ví dụ `Editor_Sua_BaiViet_101`. Nếu bài ID 102 thì em tạo `Editor_Sua_BaiViet_102`.
- **Senior:** Dạ thưa sếp, Yêu cầu "Chỉ được sửa bài của CHÍNH MÌNH" gọi là "Resource Ownership" (Quyền Sở Hữu Tài Nguyên). Yêu cầu này ĐÃ VƯỢT QUÁ KHẢ NĂNG của mô hình RBAC thuần túy. Nếu cố đấm ăn xôi sinh ra Role có ID Bài Viết sẽ gây ra Role Explosion (Có 1 triệu bài viết thì sinh 1 triệu cái Role). 
  - Cách giải quyết chuẩn là: RBAC chỉ đứng ngoài làm "Bảo vệ cổng" (Kiểm tra mày có role Editor không). 
  - Sau khi vào cửa, em sẽ chặn cấp độ "Bảo vệ phòng" (Fine-Grained) ngay tại Code Backend Hoặc Database: Mệnh đề `WHERE author_id = {user_id_trong_token}` để đảm bảo nó chỉ query được data của chính nó! RBAC không thể và không nên dùng để giữ bảo mật tầng Resource!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Server Administration Guide - Roles.
