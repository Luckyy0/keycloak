# Chương 10: Cỗ Máy Lọc Quyền (Client Scopes & Scope Evaluation)

> [!NOTE]
> Khách hàng (User) có thể nắm giữ 1.000 Quyền (Roles). Nhưng nếu họ đăng nhập vào App Chấm Công, Token sinh ra cho App Chấm Công CÓ ĐƯỢC PHÉP chứa Quyền Kế Toán không? Câu trả lời là: KHÔNG!
> Trái tim của việc Ràng Buộc Dữ Liệu và Phóng Chống Thảm Họa Tràn RAM (Token Bloat) chính là hệ thống **Client Scopes**. Đây là Cỗ Máy Lọc khổng lồ quyết định cái gì được bay vào Token, và cái gì bị ném thùng rác.

## Mục tiêu của chương
- Cắt nghĩa sự khác nhau rạch ròi giữa Default Scope (Ép buộc tiêm vào Token) và Optional Scope (Khách hàng xin mới cho).
- Thấu hiểu Cơ chế Scope Mapping: Cách ghim 1 Role/Attribute cụ thể vào trong 1 cái Scope để tiện quản lý hàng loạt.
- Tận mắt chứng kiến Cỗ Máy Lọc Scope Evaluation gạt bỏ hàng trăm Quyền rác để trả về 1 Cục JWT siêu nhẹ cho Ứng Dụng.
- Cách thiết kế Client Scopes theo chuẩn OIDC (OpenID Connect) cho Hệ sinh thái Doanh nghiệp K8s.

## Cấu trúc bài học

- `Lesson-1-Default-Scope.md`: Đội Quân Mặc Định. Tại sao Client nào sinh ra cũng bị nhét Scope `email`, `profile` và `roles` vào họng. 
- `Lesson-2-Optional-Scope.md`: Đội Quân Tùy Chọn. Bí mật đằng sau tham số `scope=address phone` trên thanh Address Bar của Trình Duyệt.
- `Lesson-3-Scope-Mapping.md`: Tuyệt kỹ Đổ Khuôn. Cách gom 5 cái Role và 3 cái Attribute cá nhân nhét chung vào 1 cái Gói Scope tên là `vip_data`.
- `Lesson-4-Client-Scope-Evaluation.md`: Máy Chém Khảo Sát Cuối Cùng. Phân tích cặn kẽ thuật toán tính Effective Scope trước khi in JSON ra Token và Cảnh báo Tắt Mặc Định `Full Scope Allowed`.

## Hướng dẫn thực hành (Labs)
- Tạo 1 Optional Scope chứa thông tin Lương Dạy Học. Dùng Postman gọi API Login 2 lần: Lần 1 không truyền tham số Scope (Token sạch), Lần 2 truyền `scope=luong_day_hoc` (Token phình to chứa Lương). Giải mã JWT để thấy sức mạnh Dynamic Scopes của OIDC.
