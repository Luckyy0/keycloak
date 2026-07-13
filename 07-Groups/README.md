# Chương 07: Kỷ Luật Phân Cấp (Groups & Hierarchy)

> [!NOTE]
> Khi số lượng User của bạn chạm mốc 1 Triệu. Việc bạn Nhấp chuột vào từng User để cấp Quyền (Roles) hay cấu hình Attribute là Hành Vi Của Những Kẻ Rảnh Rỗi và Nghiệp Dư! Khái niệm **Groups** (Nhóm) ra đời để bạn gom Hàng Ngàn Người Dùng thành các Binh Đoàn (Đội IT, Đội Sales, Phòng Kế Toán). Bạn chỉ cần Cấp Quyền cho Binh Đoàn, và toàn bộ 1 Triệu Binh Lính bên trong sẽ được hưởng lợi ích ngay lập tức nhờ Quyền Năng Kế Thừa.

## Mục tiêu của chương
- Thấu hiểu Sức mạnh của Cây Gia Phả (Group Hierarchy). Cấp bậc Nhóm Cha - Nhóm Con và Dòng chảy Kế thừa Quyền lực Rễ Dọc.
- Áp dụng Roles vào Groups (Group Roles Mapping): Bí quyết cấp Quyền Hàng Loạt Giảm Tải OOM Bể Server.
- Cấu hình Group Attributes: Gắn mác Tĩnh Cục Bộ cho cả một Bộ phận để Lọc Data dễ dàng, Không Đụng Chạm Profile Từng Người.
- Auto-Pilot với Default Groups: Nghệ thuật Khởi Sinh "Đẻ ra là tự chui vào Hàng Lối" Dành Cho Hệ Thống Mở.

## Cấu trúc bài học
Chương này Hướng Bạn Tới Nghệ Thuật Vận Hành IAM Enterprise Cực Lớn:

- `Lesson-1-Group-Hierarchy.md`: Kiến trúc Cây Phả Hệ (`/Vingroup/Vinmec/IT`). Quyền Kế thừa Lan Truyền (Transitive Inheritance) Hoạt Động Ra Sao.
- `Lesson-2-Group-Roles.md`: Ép Roles (Realm/Client) cho Group. Lõi OIDC Phẳng Map "Effective Roles" Khớp Giao Khách Nhanh Mạch.
- `Lesson-3-Group-Attributes.md`: Đính Kèm Key-Value cho Group. Tuyệt Kỹ Kéo Token Mappers Đẩy Đường Dẫn Group Đáy Mạng Nhựa Vô Bụng JWT Oanh Liệt.
- `Lesson-4-Default-Groups.md`: Robot Lọc Cửa: Bắt User Vừa Mở Mắt Đăng Ký Chui Tọt Vào Nhóm Mặc Định (`Trial`, `Guest`) Cắt Cụm Băng Bó Lệnh Rỗng.

## Hướng dẫn thực hành (Labs)
- Dựng Cây Tổ Chức Vingroup (Tập Đoàn -> Vinmec -> IT). Gán Quyền Lực Ngầm cho Nhánh IT. Đẩy Khách Hàng (User) Vô Nhánh Đó Và Decode Móc Lõi Token JWT Ra Xem Dòng Chảy Quyền Lực Có Lan Truyền Xuống Tận Đáy Bụng JWT Của Khách Bằng Phép Protocol Mappers Hay Không!
