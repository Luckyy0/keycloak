# Chương 08: Lãnh Chúa Và Chư Hầu (Realm Roles vs Client Roles)

> [!NOTE]
> Roles (Quyền / Vai Trò) là trái tim của RBAC (Role-Based Access Control). Keycloak không chỉ có 1 bọc Quyền đơn giản nằm phẳng lỳ. Nó chia cắt Quyền lực thành 2 tầng không gian: Quyền Trấn Phái Toàn Cụm (Realm Roles) và Quyền Địa Phương của từng Ứng Dụng (Client Roles). Đỉnh cao của Nghệ Thuật IAM là Phép Hợp Thể (Composite Roles) để thu nhỏ hàng ngàn Quyền con phức tạp thành 1 Mệnh lệnh Duy nhất.

## Mục tiêu của chương
- Nắm bắt sự khác biệt Cốt Tử giữa Lãnh Chúa (Realm Role) và Chư Hầu (Client Role). Tội ác của việc nhét Quyền lung tung gây sập Bảo mật Toàn Cụm.
- Tuyệt Kỹ Hợp Thể (Composite Roles): Nghệ thuật gói hàng chục Quyền nhỏ vào 1 cái Hộp. Rút ngắn 90% công sức cấp quyền cho Khách hàng.
- Thuật toán Gộp Nước (Effective Roles): Trí tuệ AI ngầm của OIDC khi nó trộn lẫn Quyền trực tiếp, Quyền từ Nhóm (Groups) và Quyền Hợp thể để Bơm căng Cục JWT Token.

## Cấu trúc bài học
Chương này đi sâu vào Bảng Cấu Hình Admin của Nhóm Quyền Lực:

- `Lesson-1-Realm-Roles.md`: Lãnh Chúa Toàn Cụm. Sức mạnh Bao trùm 100% Hệ sinh thái Realm. Cảnh báo Lỗi Kéo Data Mở Rộng Quá Tay.
- `Lesson-2-Client-Roles.md`: Quyền lực Địa Phương (Chư Hầu). Tại sao Quyền "Quản trị" của App A không được phép dính dáng tới App B. Giải phẫu Bụng `resource_access` trong JWT.
- `Lesson-3-Composite-Roles.md`: Phép Màu Hợp Thể. Cách Gói 100 Client Roles rải rác lại thành 1 Realm Role duy nhất (Ví dụ: `Super-Manager`).
- `Lesson-4-Effective-Roles.md`: Lưới Quét Cuối Cùng. Keycloak làm cách nào quét Cây Kế Thừa để Đẩy Dòng Chảy Quyền lực vào Token mà không làm Sập RAM (Tránh Token Bloat).

## Hướng dẫn thực hành (Labs)
- Tạo các Client Roles rải rác. Dùng phép Hợp Thể (Composite) để đúc thành 1 Thanh Kiếm Realm Role duy nhất. Cấp cho Khách hàng và Dùng Công Cụ Giải Mã Base64 soi thấu Tận Đáy Bụng JWT Token xem Quyền OIDC chạy bằng Mạch Nào!
