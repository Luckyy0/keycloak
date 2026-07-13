# Chapter 22: Authorization Services (Cỗ Máy Phân Quyền UMA Đỉnh Cao)

Chào mừng bạn đến với **Chương 22: Authorization Services (Cỗ Máy Phân Quyền)**.
Lâu nay chúng ta dùng Keycloak chủ yếu để giải quyết bài toán "Mày là ai?" (Authentication - Đăng nhập). Còn bài toán "Mày được làm gì?" (Authorization) chúng ta vẫn đẩy về cho Spring Boot/NodeJS tự đọc thẻ Role và tự viết If-Else (Quyền RBAC cơ bản).
Nhưng ở các hệ thống phức tạp, quyền hạn không chỉ phụ thuộc vào cái Mác (Role), mà còn phụ thuộc vào Thời gian (Giờ hành chính), Địa điểm (IP công ty), hoặc phụ thuộc vào việc Người Khác có Chia Sẻ Quyền cho bạn hay không (Ví dụ: Tính năng Share File của Google Drive).
Và đây là lúc Keycloak rút ra thanh gươm quyền lực nhất của nó: **User-Managed Access (UMA) và Fine-Grained Authorization**.

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững:
1. Cuộc chiến các mô hình quyền lực: RBAC vs ABAC vs UMA.
2. Kiến trúc Resource Server: Biến App của bạn thành "Kẻ Được Bảo Vệ".
3. Nghệ thuật ghép hình: Resources, Scopes, Policies, Permissions.
4. Giao thức Protection API: Máy chém RPT Token cắt đứt mọi truy cập trái phép.

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-UMA/`: 4 bài lý thuyết giải mã sức mạnh phân quyền tối thượng của Keycloak.
- `Labs/`: Thực hành dựng một Resource Server và viết các Policy phức tạp.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: RBAC vs ABAC vs UMA (Các Trường Phái Phân Quyền)
- Lesson 2: Resource Server Architecture (Kiến Trúc Lâu Đài Bảo Vệ)
- Lesson 3: Policies and Permissions (Nghệ Thuật Pháp Điển Hóa)
- Lesson 4: Protection API and Tokens (Cửa Ải Đòi Mạng)

Sẵn sàng khám phá cấp độ bảo mật cao nhất của một hệ thống Enterprise!
