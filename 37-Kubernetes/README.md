# Chapter 37: Vương Quốc Trên Mây (Kubernetes & Cloud Native)

## Giới thiệu (Introduction)
Chào mừng bạn đến với Đỉnh Cao Của Vận Hành! Khi hệ thống Identity của bạn cần phục vụ 1 triệu Users mỗi ngày, chạy 1 cái Docker hay 3 cái máy chủ là không đủ. Bạn cần một Dàn Nhạc Giao Hưởng (Orchestration) mang tên Kubernetes (K8s).
Trong chương cuối cùng và đồ sộ nhất này, chúng ta sẽ học cách đóng gói Keycloak thành một khối tài nguyên Khổng Lồ, Tự Động Phình To (Autoscaling), Tự Động Hồi Sinh (Self-healing), và Quản Lý Trạng Thái Vững Chắc Như Bàn Thạch trên Đám Mây.

## Mục lục (Table of Contents)

### Module 1: Bí Tịch Kubernetes Dành Riêng Cho Keycloak
*   **Lesson 1: Operator vs Helm:** Cuộc chiến giữa 2 trường phái Cài Đặt. Nên dùng Trình Quản Lý Gói Helm hay dùng Con Robot Tự Động Operator của Red Hat?
*   **Lesson 2: StatefulSet vs Deployment:** Tại sao Keycloak và Database Lại Sợ Hãi Deployment? Phân tích Kiến Trúc Dữ Liệu Có Trạng Thái (Stateful).
*   **Lesson 3: ConfigMap & Secret:** Kỹ năng Cắt Rời Code khỏi Cấu Hình (Decoupling). Cách giấu nhẹm Mật Khẩu Database bằng Secret K8s.
*   **Lesson 4: Ingress & HPA (Autoscaling):** Xây Cổng Chào Ingress Đón Khách và Cài Đặt Lò Xo HPA để Keycloak Tự Nhân Bản Khi Khách Hàng Ùn Ùn Kéo Tới.

### Labs & Thực hành (Labs)
*   **Lab 1:** Triển Khai Keycloak Lên Minikube: Tự tay viết file cấu hình `values.yaml` và cài đặt 1 Cụm Cluster Keycloak Hoàn Chỉnh Bằng Helm Chart.

## Bắt đầu từ đâu? (Where to start?)
Tiến vào thế giới Đám Mây tại [Lesson 1: Operator vs Helm](Module-1-Concepts/Lesson-1-Operator-vs-Helm.md).
