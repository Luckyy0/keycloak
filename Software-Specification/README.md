# Software Requirements Specification (SRS)
# Đặc tả Yêu cầu Phần mềm (SRS)

## 1. Purpose / Mục đích
This Software Requirements Specification (SRS) defines the overall architecture, standards, and generation rules for the Keycloak Enterprise Curriculum.
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) này xác định kiến trúc tổng thể, các tiêu chuẩn và quy tắc sinh nội dung cho Giáo trình Keycloak Enterprise.

The curriculum is designed to take learners from Beginner to Advanced levels, covering theory, practical applications, enterprise architecture, and best practices.
Giáo trình được thiết kế để đưa người học từ cấp độ Người mới bắt đầu đến Nâng cao, bao gồm lý thuyết, ứng dụng thực tế, kiến trúc doanh nghiệp và các thực hành tốt nhất.

## 2. Scope / Phạm vi
The curriculum covers Keycloak fundamentals, IAM concepts, standard protocols (OAuth2, OIDC, SAML), advanced Keycloak features, high availability, security hardening, and integrations with modern stacks (Spring Boot, React, Kubernetes).
Giáo trình bao gồm các kiến thức cơ bản về Keycloak, các khái niệm IAM, các giao thức tiêu chuẩn (OAuth2, OIDC, SAML), các tính năng nâng cao của Keycloak, tính sẵn sàng cao, tăng cường bảo mật và tích hợp với các công nghệ hiện đại (Spring Boot, React, Kubernetes).

## 3. Target Audience / Đối tượng Mục tiêu
- Fresher to Senior Backend Engineers.
- Từ Kỹ sư Backend Fresher đến Senior.
- Solution Architects.
- Kiến trúc sư Giải pháp.
- Identity & Access Management (IAM) Engineers.
- Kỹ sư Quản lý Định danh & Truy cập (IAM).

## 4. Document Structure / Cấu trúc Tài liệu
This directory (`/Software-Specification`) contains the complete set of rules and guidelines for generating the curriculum.
Thư mục này (`/Software-Specification`) chứa toàn bộ tập hợp các quy tắc và hướng dẫn để sinh giáo trình.

The documents include:
Các tài liệu bao gồm:
- **Curriculum-Overview.md**: High-level overview of the curriculum chapters. / Tổng quan cấp cao về các chương của giáo trình.
- **Learning-Path.md**: Recommended paths for different roles. / Các lộ trình học tập được đề xuất cho các vai trò khác nhau.
- **Folder-Structure.md**: Standardized directory structure for lessons and code. / Cấu trúc thư mục được chuẩn hóa cho các bài học và mã nguồn.
- **Naming-Convention.md**: Rules for naming files, variables, and components. / Các quy tắc đặt tên cho tệp, biến và thành phần.
- **Markdown-Convention.md**: Formatting standards for documentation. / Các tiêu chuẩn định dạng cho tài liệu.
- **Code-Convention.md**: Coding standards for examples and projects. / Các tiêu chuẩn mã hóa cho các ví dụ và dự án.
- **Diagram-Convention.md**: Rules for generating Mermaid diagrams. / Các quy tắc sinh sơ đồ Mermaid.
- **Curriculum-Generation-Rules.md**: Step-by-step workflow for generating lessons. / Quy trình làm việc từng bước để sinh các bài học.
- **Example-Generation-Rules.md**: Standards for writing practical examples. / Các tiêu chuẩn để viết các ví dụ thực tế.
- **Exercise-Generation-Rules.md**: Guidelines for creating exercises. / Các hướng dẫn tạo bài tập.
- **Quiz-Generation-Rules.md**: Rules for creating quizzes and assessments. / Các quy tắc tạo câu hỏi trắc nghiệm và đánh giá.
- **Interview-Generation-Rules.md**: Standards for generating interview questions. / Các tiêu chuẩn sinh câu hỏi phỏng vấn.
- **Best-Practice-Rules.md**: Guidelines for documenting enterprise best practices. / Các hướng dẫn tài liệu hóa các thực hành tốt nhất của doanh nghiệp.
- **EdgeCase-Rules.md**: Rules for detailing edge cases and troubleshooting. / Các quy tắc chi tiết hóa các trường hợp ngoại lệ và khắc phục sự cố.
- **Project-Rules.md**: Standards for end-of-chapter and capstone projects. / Các tiêu chuẩn cho các dự án cuối chương và dự án tốt nghiệp.
- **Lab-Rules.md**: Guidelines for hands-on labs. / Các hướng dẫn cho các bài thực hành.
- **Versioning.md**: Version control and updating strategies. / Kiểm soát phiên bản và chiến lược cập nhật.
- **Glossary.md**: Definitions of IAM and Keycloak terminology. / Định nghĩa các thuật ngữ IAM và Keycloak.
- **Dependency-Map.md**: Prerequisites and dependencies between modules. / Các điều kiện tiên quyết và sự phụ thuộc giữa các mô-đun.
- **Roadmap.md**: Implementation timeline and milestones. / Lộ trình triển khai và các cột mốc.
- **References.md**: Official documentation, RFCs, and external resources. / Tài liệu chính thức, RFC và các tài nguyên bên ngoài.
- **Progress-Tracking.md**: Mechanisms to track content generation progress. / Các cơ chế theo dõi tiến độ sinh nội dung.
- **Assessment.md**: Evaluation criteria for learners. / Tiêu chí đánh giá cho người học.
- **FAQ.md**: Frequently asked questions formatting. / Định dạng các câu hỏi thường gặp.
- **Curriculum-Index.md**: Master index of all generated content. / Chỉ mục chính của toàn bộ nội dung đã sinh.

## 5. Workflow Execution / Quy trình Thực thi
The generation of the curriculum must follow a strict step-by-step process.
Việc sinh giáo trình phải tuân theo một quy trình từng bước nghiêm ngặt.
The AI will generate one file, module, or lesson at a time, and pause execution.
AI sẽ sinh từng tệp, mô-đun hoặc bài học một lúc, và tạm dừng thực thi.
It will wait for the user to input "tiếp tục" before proceeding to the next item.
Nó sẽ chờ người dùng nhập "tiếp tục" trước khi chuyển sang mục tiếp theo.
