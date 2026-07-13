# Chương 04: Nghệ thuật Triển khai (Installation & Deployment)

> [!NOTE]
> Chào mừng bạn đến với Chương 4. Sau khi mổ xẻ nội tạng của Động cơ ở Chương 3, giờ là lúc chúng ta bước ra thực chiến. Bạn sẽ học cách Kéo Con Quái Vật này ra ngoài ánh sáng, từ việc chạy thử đồ chơi trên Laptop cá nhân đến việc Vận Hành Sẵn Sàng Chiến Đấu (Production-Ready) với Cụm Máy Chủ High-Availability trên Kubernetes.

## Mục tiêu của chương
- Lĩnh hội các phương pháp Cài đặt: Khởi thủy từ việc giải nén File ZIP truyền thống đến việc Kéo Trọng Lõi Docker Image hiện đại.
- Thấu hiểu Ranh Giới Sinh Tử: Sự khác biệt một trời một vực giữa Chế độ `start-dev` (Đồ chơi/Thử nghiệm) và Chế độ `start` (Vũ khí Thực chiến).
- Khám phá Đỉnh cao Cloud-Native: Chạy Keycloak trên Kubernetes (K8S), cài đặt Tự động hóa bằng Helm Charts và Quyền năng tối thượng của Người Máy Quản Gia (Keycloak Operator).
- Thiết lập Cấu trúc Sẵn sàng Cao (High Availability - HA): Chống sập, Cân bằng tải, và Khắc phục Thảm họa Xuyên Data Center.

## Cấu trúc bài học
Chương này chia làm 3 Nhóm Triển khai từ Thấp đến Cao:

- **Nhóm 1: Triển khai Cục bộ & Docker (Dành cho Lập trình viên)**
  - `Lesson-1-Local-Installation.md`: Cài đặt Gốc File ZIP (Trải nghiệm Java Thuần).
  - `Lesson-2-Docker.md`: Bọc Khối Container Siêu nhẹ.
  - `Lesson-3-Docker-Compose.md`: Dựng cụm Đơn Giản Keycloak + Database Cục bộ.
- **Nhóm 2: Triển khai Đám mây (Dành cho DevOps/K8S)**
  - `Lesson-4-Kubernetes.md`: Ném Keycloak lên Khung Sinh Tồn K8s.
  - `Lesson-5-Operator.md`: Tự động hóa Vận Hành bằng Keycloak Operator.
  - `Lesson-6-Helm.md`: Triển khai Đóng gói Chuẩn mực bằng Helm Chart.
- **Nhóm 3: Tiêu chuẩn Production (Dành cho Kiến trúc sư)**
  - `Lesson-7-Production-Installation.md`: Lên Đồ Ra Trận (SSL, Reverse Proxy, Bảo Mật Biên).
  - `Lesson-8-HA-Installation.md`: Bất Tử Hóa Bằng Cụm HA (Infinispan Clustering, JGroups).

## Hướng dẫn thực hành (Labs)
- Bài Lab cuối chương sẽ yêu cầu bạn dựng một Cụm HA Keycloak Gồm 2 Node, Đứng đằng sau Nginx Reverse Proxy, Trỏ chung vào 1 Cụm PostgreSQL để Giả Lập Môi Trường Hoạt Động Của Một Tập Đoàn.

Chuẩn bị sẵn Terminal và Bàn Phím, Chúng ta bắt đầu gõ lệnh!
