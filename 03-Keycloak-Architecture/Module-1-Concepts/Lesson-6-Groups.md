# Lesson 6: Cấu trúc Tổ chức (Groups)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Giải bài toán Phân quyền Hàng loạt (Bulk Authorization). Hiểu được Sức mạnh của Hệ thống Nhóm (Groups) Dạng Cây (Hierarchical) và Tránh thảm họa Cấp Quyền Lẻ Tẻ Bằng Tay cho từng Người dùng.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Group Là Cái Sọt Rác Có Kỷ Luật
Nếu bạn có 10.000 Nhân Viên, bạn không thể mở Bảng User ra, tìm từng người và Bấm nút Cấp quyền `Xem_Bao_Cao` cho 10.000 người đó. (Đó là Thảm Họa Vận Hành - Operational Nightmare).
**Group (Nhóm)** ra đời như một Cái Rổ. 
- Bạn ném 10.000 nhân viên vào Nhóm `Phong_Ke_Toan`.
- Sau đó, Bạn Chỉ Cần Cầm Cái Quyền `Xem_Bao_Cao` Gắn Đúng 1 Lần Vào Cái Rổ Đó. Tự Khắc 10.000 Con Người BÊN TRONG Bụng Cái Rổ Đó Đều Được Hưởng Sái Cái Quyền Này.
- Ngày mai, có 1 người Nghỉ Việc. Bạn Rút Cổ Thằng Đó Ra Khỏi Rổ. Ngay Lập Tức Nó Mất Quyền. Không Bao Giờ Lo Sót Quyền.

### 1.2. Cấu Trúc Cây Gia Phả (Hierarchical Tree)
Không Giống Nhau Các Framework Khác Chỉ Hỗ Trợ Nhóm Phẳng (Flat Groups). Keycloak Hỗ Trợ Nhóm Đa Tầng (Phả Hệ).
Ví Dụ:
- Nhóm Cha: `TruSo_Chinh`
  - Nhóm Con: `Phong_IT`
  - Nhóm Con: `Phong_Nhan_Su`
Nếu Bạn Gắn Quyền `Vao_Cong_Chinh` Cho Nhóm Cha `TruSo_Chinh`. Toàn Bộ Nhân Viên Của Cả IT Lẫn Nhân Sự Đều Được Vào Cổng Chính (Nhận Kế Thừa Từ Cha Thuận Theo Chiều Dọc). Đẳng cấp tổ chức dữ liệu Doanh nghiệp B2B (Enterprise Org-Chart) Thể Hiện Ở Đây.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Phép Lai Ghép Kế Thừa Chéo (Cross-Inheritance): Làm sao Lõi Keycloak Tính Ra Số Lượng Quyền Thực Tế Của 1 User?

```mermaid
graph TD
    subgraph "Toán Học Tập Hợp Của Keycloak (Union Set)"
        User[Alice]
        
        G1[Group: Châu Á] -->|Gắn Role: Đọc_Bản_Đồ| User
        G2[Group: Marketing] -->|Gắn Role: Đăng_Bài| User
        
        UserDirect[Gắn Trực Tiếp Cho Alice] -->|Gắn Role: Xóa_Bài| User
        
        Note over User,UserDirect: Lúc Đăng Nhập, Keycloak Dùng Thuật Toán Gom Bi (Flattening).<br/>Nó Trải Dài Toàn Bộ Cây Của Châu Á, Marketing.<br/>Cộng Dồn Với Quyền Cá Nhân.<br/>Kết Quả: Alice = Đọc_Bản_Đồ + Đăng_Bài + Xóa_Bài.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Nguyên Tắc Cấm Kỵ Cấp Quyền Trực Tiếp (No Direct Role Assignment)**
> **Tội ác Thường Gặp:** Trưởng Phòng IT Yêu Cầu Cấp Quyền Admin Cho Anh Dev Tên Bob. Bạn Bèn Mở Tên Bob Ra, Gắn Role `Super_Admin` Vào Trực Tiếp Mõm Của Bob.
> Sau Này Trưởng Phòng Mới Lên, Hỏi Báo Cáo: "Có Bao Nhiêu Thằng Đang Cầm Quyền Admin Thế?". Bạn Sẽ Bị Mù Hoàn Toàn Vì Bạn Phải Query Quét Cả Triệu User Mới Lòi Ra Thằng Bob Mắc Kẹt Ở Trong.
> **Quy Luật Vàng:** TUYỆT ĐỐI KHÔNG GÁN ROLE TRỰC TIẾP CHO USER.
> Bắt Buộc Tạo 1 Group Tên Là `DevOps_Admins`. Gắn Role `Super_Admin` Vào Group Này. Rồi Nhét Bob Vào Trong. Khi Kiểm Toán Viên Hỏi, Chỉ Cần Mở Group Ra Sẽ Thấy Sạch Sẽ Danh Sách Kẻ Cầm Quyền. 

> [!CAUTION]
> **Thảm Họa Bùng Nổ Token (Token Bloat Due To Deep Trees)**
> Bạn Lạm Dụng Vẽ Cây Group Tới 10 Tầng (Từ Tổng Công Ty -> Tập Đoàn -> Nhánh -> Phòng -> Tổ -> Cá Nhân).
> Khi Thằng User Thuộc Tầng Cuối Đăng Nhập. Hàm Flattening Của Keycloak Phải Quét Ngược Lên 10 Tầng Cha Để Lôi Hết Mọi Role Gắn Xuống Cho Nó. Cái JWT Token Đẻ Ra Chứa Tận 100 Cái Roles Khác Nhau. Token Dài Quá 8KB Sẽ Bị Nginx Của App Backend Chặn Đứng (Lỗi HTTP 431 Request Header Fields Too Large). 
> **Cách Vá:** Chặn Thuật Toán Auto-Mapping. Chỉnh Sửa `Client Scope` Để Keycloak KHÔNG Nhét Group Lên JWT, Chuyển Sang Dùng Opaque Token, Hoặc Bắt Backend Tự Gọi API Hỏi Quyền Thay Vì Nhồi Data Cứng Vào Token.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Tự Động Hóa Với **Default Groups (Nhóm Mặc Định)**:
Bình thường, Khách Lạ Đăng Ký Tài Khoản Ở Web (B2C). Họ Lọt Vào CSDL Bằng Tư Cách Rỗng Tuếch, Không Thuộc Nhóm Nào. IT Phải Add Tay Vào Group Tốn Thời Gian.

Trong Keycloak: Bạn Mở Menu `Groups` -> `Default Groups`. 
Bạn Lựa Chọn Một Nhóm Tên Là `Khach_Hang_Moi_Kham_Pha`.
BÙM! Từ Giây Phút Đó Trở Đi, Bất Kỳ Kẻ Nào Dùng Tính Năng "Register" Tự Đăng Ký Trên Trình Duyệt. CSDL Khi Tạo User Lập Tức Auto-Link Thằng Khách Đó Vào Cái Group Này Luôn. Cấp Phát Sẵn Các Quyền Sơ Cấp (Như Được Xem Hàng Tồn Kho, Nhưng Không Được Mua Dưới Giá Sỉ) Hoàn Toàn Tự Động.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Lưỡng Tính Của Bức Tường Lửa (Group Based Policy):**
  - Group Không Chỉ Dùng Để Cấp Role (Cho Phép Mở Cổng Tòa Nhà).
  - Nó Còn Dùng Để **Chặn Đứng Vòng Đời Trải Nghiệm**.
  - Ví Dụ: Có Thể Cấu Hình Authentication Flows: "Mọi Khách Hàng Ở Mức Độ Tin Cậy Thường Đều Có Thể Login Bằng Password. NHƯNG, Nếu Keycloak Xét Thấy Thằng Chả Nằm Trong Group `VIP_Trading`. Lập Tức Khóa Họng Máy Đăng Nhập Password. Bắt Buộc Rẽ Nhánh (Condition) Bắt Thằng Chả Lôi YubiKey Ra Cắm Vào".
  - Như Vậy, Group Trở Thành Yếu Tố Điều Hướng (Contextual Routing) Dòng Chảy Xác Thực chứ không đơn thuần Bó Hẹp Trong Ranh Giới Ủy Quyền (AuthZ).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Keycloak, Giữa Thuộc Tính Của Nhóm (Group Attributes) Và Thuộc Tính Của Người Dùng (User Attributes). Nếu Cùng Chứa Một Chìa Khóa (Ví Dụ Cùng Có Khóa Là `Muc_Giam_Gia`), Khi User Thuộc Nhóm Đó Đăng Nhập Thì Thằng Nào Đè Thằng Nào?**
- **Junior:** Thằng User đè thằng Group vì User ở sau.
- **Senior:** Keycloak Xài Cơ Chế **Ghi Đè Kế Thừa (Inheritance Overriding)**.
Theo thứ tự ưu tiên Từ Gốc Ra Ngọn:
1. Nhóm Cha (Yếu Nhất).
2. Nhóm Con (Đè Cha).
3. Người Dùng (Đỉnh Chuỗi Thức Ăn).
Nếu Group Ghi `Muc_Giam_Gia = 10%`. Nhưng Riêng Ở Cái Cục Của User Ghi Rõ `Muc_Giam_Gia = 50%`. Hàm Tính Toán Cuối Cùng Sẽ Lấy Mức 50% Gắn Vào Token Đẩy Đi Cho Ứng Dụng Khách. Cá Nhân Hóa Đè Đạp Mọi Luật Lệ Tập Thể.

**2. Nếu Nhân Viên Alice Nằm Thuộc 2 Group Khác Nhau (Group Sales Và Group Marketing). Group Sales Cấm Truy Cập Thư Mục A. Group Marketing Cho Phép Truy Cập Thư Mục A. Lõi OIDC Sẽ Giao Tiếp Phân Quyền Thế Nào Với Backend? Alice Có Bị Chặn Không?**
- **Junior:** Bị lỗi xung đột, không biết cho vô hay chặn.
- **Senior:** Lại Sinh Ra Nhầm Lẫn Quyền Lực Chết Người Khác Nhau Giữa Identity Provider (IdP) Và Policy Enforcement Point (PEP).
- Keycloak (Với Tư Cách Cấp Token): TUYỆT ĐỐI KHÔNG BIẾT Sự Mâu Thuẫn Này Cấm Hay Mở. Nhiệm Vụ Của Nó Là Một Người Máy Đưa Đồ Lấy Gì Gói Đấy. Nó Sẽ Lôi Sạch Cả Role Cấm Lẫn Role Mở Nhét Tất Tần Tật Vào 1 Cái Token Ném Trả Về.
- Ứng Dụng Backend (App Khách - SP): Khi Mổ Cái Bụng Token Ra Xem. CHÍNH NÓ Mới Là Kẻ Dùng Lệnh `if/else` Chạy Thuật Toán Phân Giải Xung Đột (Ví dụ Nguyên Lý Deny-Overrides: Hễ Thấy Bất Kỳ 1 Chữ CẤM Nào Trong Mảng Quyền Thì Trảm Không Thương Tiếc, Dù Có 10 Chữ MỞ Kế Bên). Do Đó, Trách Nhiệm Quyết Định Sinh Sát Của Xung Đột Nhóm Nằm Hoàn Toàn Ở Khung Thực Thi Của Mã Nguồn Mọi App Backend Khác. Trừ phi Dùng Hệ Thống Keycloak Authorization UMA 2.0.

**3. Tại Sao Việc Sắp Xếp Dữ Liệu Users Vào "Group" Lại Hiệu Quả Hơn Cực Nhiều Về Hiệu Năng So Với Việc Thiết Kế Bằng Code Dựa Theo Lọc "User Attribute" Để Ra Quyết Định Chặn Cổng?**
- **Junior:** Tại Group có giao diện bấm dễ hơn.
- **Senior:** Tại Gốc Database! Cấu Trúc Bảng Dữ Liệu Khác Nhau Xa Lắc.
- **Group (Membership):** Bảng Quan Hệ `USER_GROUP_MEMBERSHIP`. Đây là Bảng Nối (Join Table) Thuần Trị Chứa Khóa Chính Cực Gọn. Tốc Độ Quét JOIN Bằng Index SQL Cực Kỳ Thủng Sàn. Tốn 0.5 Giây Cho 1 Triệu Người Dùng.
- **User Attribute:** Bảng Chứa Lỗ Hổng Hư Hỏng Khóa-Giá Trị. Cột Chứa Giá Trị Là Kiểu String Đa Hệ (Varchar). Khi Query: `WHERE attribute_name = 'PhongBan' AND attribute_value = 'IT'`. Database Bắt Buộc Phải Chạy Quét Chuỗi Ký Tự Bề Mặt (Full Table Scan Đôi Lúc Xảy Ra Nếu Index Kém Tối Ưu). 1 Triệu Người Chạy Có Khi Nín Mất 15 Giây Máy Chủ Treo Đơ Nòng. Xài Group Luôn Luôn Chiến Thắng Áp Đảo Về Chuẩn Hóa RDBMS.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Groups and Roles.
