# Chương 11: Xưởng Đúc Dữ Liệu (Protocol Mappers & Custom Claims)

> [!NOTE]
> Giai đoạn đánh giá Client Scopes (Scope Evaluation) chỉ là bộ lọc xem "Khách có quyền được lấy cái gì". Nhưng LÀM SAO để biến 1 cái cột Tên Đường trong Database thành 1 dòng chữ `address` trong file Token JSON?
> Chào mừng đến với **Protocol Mappers** - Xưởng đúc khuôn khổng lồ của Keycloak, nơi nhào nặn mọi cấu trúc dữ liệu theo đúng ý muốn của các lập trình viên Backend khó tính nhất.

## Mục tiêu của chương
- Thấu hiểu khái niệm Mapper: Ống dẫn dữ liệu từ Database vào bên trong Token OIDC/SAML.
- Tự tay sử dụng các Mappers tích hợp sẵn (Built-in) cực kỳ mạnh mẽ để rút ruột User (Attribute), Group, Role mà không cần viết 1 dòng code.
- Kỹ thuật nâng cao: Viết Javascript (Script Mapper) để nhào nặn Data động lúc Runtime.
- Đỉnh cao Backend: Code Custom Mapper bằng Java để Keycloak trực tiếp gọi qua API hệ thống ngoài lấy Data nhét vào Token.

## Cấu trúc bài học

- `Lesson-1-Built-in-Mapper.md`: Điểm danh các Ống Bơm có sẵn mạnh mẽ nhất.
- `Lesson-2-User-Attribute-Mapper.md`: Móc cột dữ liệu (Ví dụ: `chuc_vu`) từ Bảng User đẩy vào Claim `title` của JWT.
- `Lesson-3-Group-Mapper.md`: Giải quyết bài toán kéo danh sách Phòng ban vào Token để Backend phân quyền theo Tổ chức.
- `Lesson-4-Role-Mapper.md`: Kỹ thuật Prefix `ROLE_` cứu sống hàng triệu dự án Spring Boot đời cũ.
- `Lesson-5-Script-Mapper.md`: (Nâng cao) Nhúng mã Javascript trực tiếp vào Cỗ Máy Nhào Nặn Data.
- `Lesson-6-Custom-Mapper.md`: (Chuyên gia) Deploy 1 file `.jar` Java vô Keycloak để tự chế tạo Cỗ Máy Bơm Data riêng.

## Hướng dẫn thực hành (Labs)
- Tạo 1 User Attribute Mapper móc cột `ma_nhan_vien` vào JWT.
- Sử dụng Javascript Script Mapper để kiểm tra: Nếu `ma_nhan_vien` bắt đầu bằng `VIP-`, tự động chèn thêm Claim `"is_vip": true` vào Token trước khi Token bắn về React App.
