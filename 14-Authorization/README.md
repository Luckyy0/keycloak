# Chapter 14: Authorization (Phân Quyền)

Chào mừng bạn đến với **Chương 14: Authorization**.
Nếu "Authentication (Xác thực)" là để trả lời câu hỏi *"Bạn là ai?"*, thì "Authorization (Phân quyền)" là để trả lời câu hỏi *"Bạn được phép làm gì?"*. 

Trong các hệ thống đơn giản, việc kiểm tra xem User có mang "Role Admin" hay không (RBAC) là đủ. Tuy nhiên, khi hệ thống doanh nghiệp lớn lên, bạn sẽ cần các kiểm tra phân quyền phức tạp hơn rất nhiều. Ví dụ: "Chỉ cho phép Bác Sĩ xem hồ sơ Bệnh Nhân trong ca trực của họ (từ 8h-17h)" - đây là ABAC. Hoặc "Chỉ cho phép sửa bài viết do chính bạn tạo ra" - đây là Fine-Grained. 
Keycloak cung cấp một nền tảng **Authorization Services** cực kỳ mạnh mẽ hỗ trợ đầy đủ các tiêu chuẩn này dựa trên **UMA 2.0 (User-Managed Access)**.

## Mục Tiêu Học Tập (Learning Objectives)

Sau khi hoàn thành chương này, bạn sẽ nắm vững:

1. Phân biệt rõ sự khác nhau giữa RBAC (Role-Based), ABAC (Attribute-Based), và PBAC (Policy-Based).
2. Hiểu cấu trúc phân quyền mịn (Fine-Grained Authorization) của Keycloak bao gồm: Resources, Scopes, Policies, và Permissions.
3. Cách thiết lập các thuật toán ra quyết định (Decision Strategies) khi nhiều Policy chồng chéo nhau.
4. Cách tích hợp Authorization Services vào một ứng dụng Client bảo mật.

## Cấu Trúc Thư Mục (Directory Structure)

- `Module-1-Concepts/`: Các bài lý thuyết chuyên sâu về mọi khía cạnh của Phân Quyền.
- `Labs/`: Các bài tập thực hành thiết lập Policy, Permission và mô phỏng cấp quyền API.
- `code/`: Chứa file `docker-compose.yml` để khởi động môi trường Keycloak phục vụ cho Labs.

## Danh Sách Bài Học (Lesson List)

1. **Lesson 1:** RBAC (Role-Based Access Control)
2. **Lesson 2:** ABAC (Attribute-Based Access Control)
3. **Lesson 3:** Fine-Grained Authorization (Phân Quyền Mịn)
4. **Lesson 4:** Policy-Based Access Control (PBAC)
5. **Lesson 5:** Decision Strategies (Chiến Lược Ra Quyết Định)

Hãy bắt đầu với bài đầu tiên: **RBAC** để nắm được nền móng truyền thống nhất của Phân quyền!
