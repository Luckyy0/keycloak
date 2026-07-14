# Chương 57: Capstone Project (Đồ án tốt nghiệp)

> [!NOTE]
> **Category:** Capstone/Project
> **Goal:** Tổng hợp và vận dụng toàn bộ kiến thức từ 56 chương trước để thiết kế, triển khai và bảo vệ một hệ thống Quản lý Định danh và Truy cập (IAM) cấp độ doanh nghiệp (Enterprise-Grade) hoàn chỉnh.

## 1. Giới thiệu (Introduction)

Chào mừng bạn đến với **Capstone Project** – điểm đến cuối cùng trong hành trình chinh phục Keycloak và kiến trúc bảo mật phân tán. Đây không phải là một bài học lý thuyết thông thường, mà là một **Đồ án thực chiến**. 

Bạn sẽ đóng vai trò là một **Security Architect / DevSecOps Engineer** tại một tập đoàn tài chính. Nhiệm vụ của bạn là thay thế hệ thống xác thực cũ kỹ của tập đoàn bằng một nền tảng IAM hiện đại dựa trên Keycloak. 

Hệ thống này phải thỏa mãn các tiêu chí khắt khe nhất của môi trường Production (Production-Ready):
1. **High Availability (HA):** Chịu được lỗi máy chủ (Fault Tolerance) mà không rớt Session của người dùng.
2. **Zero Trust Architecture:** Các Microservices bên trong mạng nội bộ không tin tưởng lẫn nhau, mọi giao tiếp đều phải có JWT và xác thực chữ ký (Token Relay / M2M).
3. **Bảo vệ SPA:** Tuyệt đối không lưu Access Token trên trình duyệt. Sử dụng kiến trúc BFF (Backend-For-Frontend) với HttpOnly Cookie và SameSite.
4. **Tích hợp doanh nghiệp:** Đồng bộ tự động với máy chủ danh bạ Microsoft Active Directory (LDAP).

## 2. Cấu trúc chương (Chapter Structure)

Đồ án này được thiết kế để dẫn dắt bạn qua các giai đoạn vòng đời phát triển phần mềm (SDLC):

- **Lesson 1: Architecture Design (Thiết kế Kiến trúc Tổng thể):** Phác thảo sơ đồ mạng, vùng DMZ, và chiến lược kết nối các thành phần.
- **Lesson 2: Deployment Strategy (Chiến lược Triển khai):** Kế hoạch đóng gói Docker, triển khai lên Kubernetes với cấu hình Horizontal Pod Autoscaler (HPA) và Database Connection Pooling.
- **Lesson 3: Security & Performance Checklist (Rà soát Bảo mật & Hiệu năng):** Bảng kiểm tra cuối cùng (Final Checklist) trước khi Go-live, rà soát các rủi ro bảo mật (CORS, CSRF, Token Bloat) và tinh chỉnh hiệu năng.
- **Lab 1: The Final Defense (Bảo vệ Đồ án):** Bạn sẽ được cung cấp một bộ mã nguồn lỗi (vulnerability-injected code). Nhiệm vụ của bạn là debug, vá lỗi và dựng thành công toàn bộ kiến trúc lên môi trường local.

## 3. Cách tiếp cận (How to Approach)

Đây là chương kiểm tra kỹ năng tư duy hệ thống (Systems Thinking). 
- Đừng vội vã chạy code. Hãy lấy giấy bút và vẽ lại sơ đồ kiến trúc.
- Nếu gặp lỗi trong phần Lab, hãy tham khảo lại các chương *35-Troubleshooting* và *42-Troubleshooting* để tự mình chẩn đoán lỗi (Log Analysis) thay vì tra Google ngay lập tức.

Chúc bạn bảo vệ Đồ án thành công!
