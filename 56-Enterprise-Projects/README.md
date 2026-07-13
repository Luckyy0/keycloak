# Chương 56: Các Dự Án Doanh Nghiệp (Enterprise Projects)

## Giới thiệu (Introduction)
Chào mừng bạn đến với **Chương 56: Các Dự Án Doanh Nghiệp (Enterprise Projects)**. 
Sau khi đã đi qua 55 chương lý thuyết và thực hành từ cơ bản đến chuyên sâu, đây là lúc chúng ta ghép nối tất cả những mảnh ghép lại với nhau thành các bức tranh hoàn chỉnh mang tầm cỡ Enterprise (Doanh nghiệp).

Trong chương này, chúng ta sẽ không học thêm khái niệm lý thuyết mới nào về Keycloak, mà sẽ áp dụng 100% kiến thức đã học để giải quyết 10 bài toán kiến trúc (Projects) thường gặp nhất tại các Tập đoàn Công nghệ, Ngân hàng, và các hệ thống Microservices quy mô lớn. 

Mỗi bài học trong chương này là một bản thiết kế hệ thống (System Design) hoàn chỉnh, từ yêu cầu nghiệp vụ đến các bước cấu hình cụ thể.

## Mục lục (Table of Contents)

### Module 1: Từ Nguyên Bản Đến Vi Dịch Vụ
*   **Lesson 1: Project 01 - Basic Login:** Bài toán căn bản nhất, tích hợp Keycloak cho một ứng dụng Web truyền thống.
*   **Lesson 2: Project 02 - RBAC:** Triển khai hệ thống phân quyền dựa trên vai trò (Role-Based Access Control) khắt khe cho nội bộ doanh nghiệp.
*   **Lesson 3: Project 03 - OAuth2 Client:** Tích hợp với ứng dụng đối tác thứ 3 bằng giao thức OAuth2 chuẩn mực.
*   **Lesson 4: Project 04 - Spring Boot Resource Server:** Bảo vệ các API Backend viết bằng Spring Boot bằng JWT.
*   **Lesson 5: Project 05 - BFF Architecture:** Giải quyết triệt để bài toán bảo mật cho ứng dụng SPA (React/Angular) bằng mô hình Backend-For-Frontend, loại bỏ hoàn toàn Token ở trình duyệt.
*   **Lesson 6: Project 06 - API Gateway:** Tập trung hóa toàn bộ luồng xác thực tại cửa ngõ Spring Cloud Gateway.
*   **Lesson 7: Project 07 - Microservices:** Xử lý xác thực liên dịch vụ (Service-to-Service) và Token Relay trong một cụm Microservices phức tạp.
*   **Lesson 8: Project 08 - LDAP Integration:** Đồng bộ hàng chục ngàn nhân viên từ hệ thống Microsoft Active Directory/LDAP cũ lên Keycloak.
*   **Lesson 9: Project 09 - HA Cluster:** Thiết kế cụm Keycloak sẵn sàng cao (High Availability) với Database Galera và Load Balancer.
*   **Lesson 10: Project 10 - Enterprise IAM Platform:** Mô hình siêu nền tảng Identity and Access Management (IAM) Identity Broker cho tập đoàn đa quốc gia.

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây dựng môi trường giả lập cho một dự án Enterprise mẫu bằng Docker Compose, bao gồm Keycloak, Nginx, Spring Boot Backend và React Frontend.

## Bắt đầu từ đâu? (Where to start?)
Hãy chuẩn bị tinh thần thép để giải quyết bài toán thực tế đầu tiên tại [Lesson 1: Project 01 - Basic Login](Module-1-Concepts/Lesson-1-Project-01-Basic-Login.md).
