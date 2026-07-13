# Chapter 23: Production Setup (Triển Khai Mạng Lưới Lõi Lên Production)

Chào mừng bạn đến với **Chương 23: Production Setup**.
Tất cả những gì chúng ta làm từ đầu khóa học (chạy lệnh `start-dev`, dùng thẻ Admin/admin, dùng Database ngầm H2, chạy HTTP cổng 8080) chỉ là trò chơi ở Local.
Khi đưa Keycloak lên Môi Trường Thực Tế (Production), mọi thứ phải lột xác. Chúng ta cần Cấu Trúc Database Riêng Biệt, Kiến Trúc Chạy Đa Máy Chủ (Clustering), Và Lớp Vỏ Bọc Chống Đạn (Reverse Proxy / TLS).

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững:
1. Đấu nối Database: Cấu hình External Database (PostgreSQL/MySQL) và Tuning Connection Pool.
2. Nghệ thuật Clustering: Khám phá lưới bộ nhớ phân tán Infinispan và giao thức JGroups (UDP/TCP).
3. Đứng sau bóng tối: Cấu hình NGINX/HAProxy để hứng đạn TLS Termination và truyền Header (X-Forwarded-For).

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Database/`: 2 bài lý thuyết về Mạch máu Database.
- `Module-2-Clustering/`: 2 bài lý thuyết về Bộ não Đa nhân (Infinispan).
- `Module-3-Reverse-Proxy/`: 2 bài lý thuyết về Lớp giáp sắt chặn cửa.
- `Labs/`: Thực hành dựng Cụm 2 máy chủ Keycloak kết nối chung Database và NGINX Load Balancer.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- **Module 1**
  - Lesson 1: External Database Setup (Cấy Ghép Tim PostgreSQL)
  - Lesson 2: Database Tuning (Ép Xung Bộ Cấp Huyết)
- **Module 2**
  - Lesson 1: Infinispan and Caching (Mạng Lưới Thần Kinh Phân Tán)
  - Lesson 2: JGroups and Discovery (Tiếng Hú Gọi Bầy)
- **Module 3**
  - Lesson 1: TLS Termination (Áo Giáp Chống Đạn)
  - Lesson 2: Header Forwarding (Đánh Lừa Khẩu Vị)

Hãy Chuẩn Bị Tinh Thần Để Bước Vào Thế Giới Dành Riêng Cho Kiến Trúc Sư Hệ Thống!
