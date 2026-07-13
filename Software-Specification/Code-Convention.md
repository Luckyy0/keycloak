# Code Convention Specification
# Đặc tả Chuẩn Mã nguồn

## 1. No Pseudo-code / Không dùng Mã giả
- ALL code examples provided in the curriculum MUST be syntactically correct and fully runnable.
- TOÀN BỘ ví dụ mã nguồn được cung cấp trong giáo trình PHẢI đúng cú pháp và có thể chạy được hoàn toàn.
- Avoid ellipsis (`...`) substituting important logic. If omitted, clearly state that it is boilerplate.
- Tránh dùng dấu chấm lửng (`...`) thay thế cho logic quan trọng. Nếu lược bỏ, phải nêu rõ đó là mã rập khuôn.

## 2. Explanation by Comments / Giải thích bằng Chú thích
- Code blocks must contain inline comments that explain the "WHY" (the reasoning), not just the "WHAT" (the action).
- Các khối mã phải chứa chú thích nội tuyến giải thích "TẠI SAO" (lý do), không chỉ là "CÁI GÌ" (hành động).
- Example:
  ```java
  // TỐT: Chặn yêu cầu nếu không có ROLE_ADMIN để ngăn chặn leo thang đặc quyền
  @PreAuthorize("hasRole('ROLE_ADMIN')") 
  
  // XẤU: Kiểm tra role admin
  @PreAuthorize("hasRole('ROLE_ADMIN')")
  ```

## 3. Technology Stack Versions / Phiên bản Công nghệ
The curriculum targets modern, production-ready enterprise stacks:
Giáo trình nhắm đến các công nghệ doanh nghiệp hiện đại, sẵn sàng cho môi trường sản xuất:
- **Keycloak**: 22.x+ (Quarkus distribution)
- **Java**: 17+
- **Spring Boot**: 3.x
- **Spring Security**: 6.x
- **Docker / Kubernetes**: Latest stable

## 4. Configuration Completeness / Tính hoàn thiện của Cấu hình
- Whenever a Java snippet is shown, the corresponding `application.yml` or `application.properties` MUST be provided.
- Bất cứ khi nào một đoạn mã Java được hiển thị, PHẢI cung cấp tệp `application.yml` hoặc `application.properties` tương ứng.
- Do not assume the learner knows the hidden properties.
- Không được mặc định rằng người học đã biết các thuộc tính ẩn.
