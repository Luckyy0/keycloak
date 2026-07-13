# Lesson 4: Triển khai Đám mây (Kubernetes - K8S)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết & Kiến trúc)
> **Goal:** Khi Công Ty Có Tới Hàng Triệu Khách Hàng Truy Cập. Một Máy Chủ Docker Compose Là Đồ Chơi Dễ Vỡ Dành Cho Sinh Viên. Bạn Cần Bắn Keycloak Lên Nền Tảng Đám Mây Đỉnh Cao Nhất Thế Giới: Kubernetes (K8S). Tại Đây, Keycloak Đạt Được Trạng Thái Bất Tử Tự Phục Hồi (Self-Healing).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sự Rời Rạc Của Microservices (Phép Màu Phân Tán)
Ở Docker Compose, Bạn Bị Trói Vùng Vào Cứng 1 Laptop (Hoặc 1 Máy Cày Ảo VPS). Nếu Điện Lưới Nơi Cái Máy Đó Bị Cháy, Toàn Cụm Rụng Hết.
Trên K8S Khác Biệt Một Trời Mực: 
- K8S Cho Phép Bạn Ném Cục Container Keycloak Giao Này Cho 10 Máy Chủ Vật Lý Ở Mỹ, 5 Máy Chủ Ở Châu Á. 
- Nó Bao Bọc Tiến Trình Chạy Lên Gọi Là Một Cái **Pod (Vỏ Kén)**.
- Khi Cái Máy Ở Châu Á Cháy Nổ Hỏng Cáp Nguồn. K8S Lập Tức Nhận Còi Báo Đứt, Nhanh Trí Chụp File Mẫu Đẻ Thêm Rặn Tức Thời 5 Cái Kén Mới Tinh Tại Cụm Máy Mỹ Bù Qua Đắp Lại. Khách Hàng Tiếp Tục Cú Click Nhẹ Tênh Không Bị Báo Văng Mất Nửa Cửa Lỗi!

### 1.2. Mâu Thuẫn Kinh Điển Của Lõi Keycloak Trên K8S (Deployment vs StatefulSet)
K8S Cung Cấp 2 Trọng Mẫu Triển Khai Mầm Trọng Tâm (Kẻ Nào Đóng Lệnh Sai Kẻ Đó Ăn Mầm Lỗi Treo Server):
1. **Deployment (Khung Sinh Tồn Vô Hồn / Stateless):** Pod Sinh Ra Tên Ngẫu Nhiên (Ví dụ: `keycloak-pod-as83x`). Nó Không Cần Nhớ Quá Khứ. Nó Đập Mặt Tường Vỡ Thì Đẻ Ra Thằng Khác Tên Khác Khỏa Lấp Nhanh Gọn.
2. **StatefulSet (Khung Sinh Tồn Giữ Hồn Gắn Rễ / Stateful):** Pod Sinh Ra Có Trọng Đít Số Thứ Tự Cứng (`keycloak-0`, `keycloak-1`). Rớt Chết Mảnh Số 0, K8s Bắt Buộc Rặn Lại Đứng Đợi Tên `keycloak-0` Mọc Lại Từ Lõi Chết Trở Lên. Mọi Dữ RAM DB Giữ Cứng Thường Lệ Gắn Đáy.

**Sự Lựa Chọn Cốt Tử Cho Keycloak Lõi Infinispan:**
Keycloak Bản Thân Trông Giống Vô Trí (Stateless) Vì Có DB Dưới Trục Rồi.
TUYỆT ĐỐI SAI LẦM! Ở Lesson 9 (Cache), Ta Biết Rằng Các Con Máy Chủ Keycloak Cần **GỌI MẠNG LƯỚI CHO NHAU ĐỂ TUNG CHÉO CACHE (JGroups/Infinispan)**. 
- Mạng Lưới Nhện Cache Đóng Sóng Sẽ Cực Nhọc Nhằn Văng Exception Đội Hình Nếu Tên Từng Đứa Bị Đổi Xoay Liên Tục Cấp Rác Đáy (Deployment).
- Do Đó: Giới Siêu Cấp Chuyên Gia (Enterprise Architect) Luôn Thiết Kế Đóng Keycloak Trên Nền **StatefulSet**. Nhờ Có Cái Tên Cứng Ngắc, Bọn Infinispan Dễ Nhớ Mặt Nhau, Giao Thoa Đắp Kế Dữ RAM Tốc Độ Không Bị Mờ Mịt Lủng Lạc Thủng Mạng Chờ Đợi Đuổi IP Chéo (Khống Chế Clustering Đỉnh Chóp Bất Ngờ Giữ Gọn).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Tự Băng Bó Lệnh Hư Lỗi Rớt Máy Khi Keycloak Trúng Cú Tát OOM:

```mermaid
graph TD
    subgraph "Kubernetes Kéo Lên Trục Vực Hồi Sinh"
        Pod0[Pod: keycloak-0 <br/> Chứa Infinispan Đang Bị Căng RAM Cạn Mạng Nhớ Trút 4GB]
        Control[K8s Kubelet Lính Đáy Quản Quản]
        Liveness[Mũi Kim Tiêm Health Check Liveness Probe Cắm Đo Mạch Máu]
        
        Pod0->>Pod0: Chạy Lệnh Import Lớn Mắc Nghẽn Trút Quá OOM Tắt Trí Mạng Lỗi OutOfMemory Chết Lặng Mất Hồi Báo Trả Lời HTTP 200...
        
        Liveness->>Pod0: Gọi Khều /health/live Mỗi 10s... 1 Lần Trượt... 2 Lần Trượt... 3 Lần Báo Trắng Hỏng Code Nhận 500!
        
        Liveness->>Control: Bắn Thư: "Ê Thằng k-0 Chết Tươi Não Rồi, Giết Khẩn Băm Xác Cắt Mạng Đi Khỏi Báo Giao Dịch Lừa Khách Nhầm Lỗi!"
        
        Control->>Pod0: SIGKILL Hủy Diệt Máy Trọng Ảo. Cắt Khung Tức Khắc.
        
        Control->>Control: Rút Bản Vẽ (StatefulSet) Nạp Start Mới Bật 1 Phút Sau Nhanh Đáy Nước Nở Trở Lại Lành Lặn Bóng Tươi Có Cấu Hình Nạp Ổn (Tự Động Hồi Phục Zero Downtime).
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Giải Mã Mạng Mù Ảo Tách Khối JGroups (Headless Service Bắt Sợi Trực Tâm)**
> **Tội Ác Thiết Kế Network:** Khi Bắn StatefulSet Của Keycloak Lên. Trưởng Nhóm DevOps Cho Gọi Sợi Mạch Tên Là `Service Type ClusterIP` Mặc Định.
> **Hậu Quả Thảm Sát Clustering:** Khi Thằng `keycloak-0` Muốn Gọi Rút Trọng Tốc Đồng Bộ Sang Sợ Nhớ Infinispan Ở `keycloak-1`. Nó Kêu Qua Cái Tên Service Của Cụm. Cái Thằng Service Kubernetes Này Là Vị Bác Sĩ Đảo Cửa Lệch Rỗng (Load Balancer Nghịch). Nó Thấy Gọi Lệnh Liền Quăng Ngược Lại Cho Chính Khớp Thằng Khác Hoặc Quăng Random Trả Nhầm Đường Đáy. Infinispan Mù Màu Vỡ Trận Đứt Sóng Đồng Bộ Trắng Nhau (Split-Brain Do Rẽ Băng Tần Sai Khẩu Vị Giao Thức Mạng Đa Điểm).
> **Thép Chặn Vực Thẳm:** Phải Tạo Cục Mạng Rỗng Vô Diện **`Headless Service (ClusterIP: None)`**. Lúc Đâm Lệnh Headless. DNS Trả Kéo Hiện Vạch Trần Toàn Bộ IP Nằm Rút Trụ Rễ Từng Cọng `keycloak-0`, `keycloak-1` Trống Phẳng Cho Lõi Infinispan Khung Chụp Kẻ Đó Móc Giao Gắn Gốc Sát Nhau Nhau Nhất Gắn Đội Cụm Chéo Giết Bỏ Rào Cản Proxy Trái Lệch.

> [!CAUTION]
> **Cắt Trọc Khung HTTPS Ở Vành Đai Bức Tường (TLS Termination Ingress)**
> Có Nhiều Công Ty Lo Lắng Kéo Tít Lệnh HTTPS Bảo Mật Chạy Thẳng Vào Đáy Ruột Của Container Nằm Trong K8S Bắt Nó Gánh Lọc Tải Chữ Ký Mã Hóa Trọng Nghẽn Tắc. Bóp Chết Tài Nguyên Tính Toán Nhựa Bọc.
> **Kiến Trúc Enterprise Đúng Nghĩa:**
> Ở Ranh Giới Vào Nước (Ingress Controller Của K8s Như Nginx Ingress). Đây Là Chỗ Lấy Kéo Cắm Chứng Chỉ Đảo Vành Ngoài Chặn Phủ (SSL Termination). Bức Tường Lửa Ngài Mạng Giải Mã Mở Áo Giáp HTTPS Cho Sạch Sẽ Trút Bỏ Nặng Gánh Gỡ Tải Dữ. Đẩy Giao Lệnh Trống Sạch Vào Ruột Lõi Bụng Keycloak Mạng Kín Nội Bộ (HTTP Nhẹ Gấp Nhịp Băng Trọng Đáy Nhẹ Ruột Không Mã Tán RAM Băng). Đương Nhiên Phải Bơm Biến Lệnh Khuyến Cáo Rỗng Đuôi Cực Hình Vào Khung: `KC_PROXY=edge`. Lúc Này Keycloak Thấy Cục HTTP Phẳng Đi Vô Nó Cũng Vui Vẻ Nạp Hiểu Là Được Bảo Kê Tín Tưởng SSL Sạch Rồi, Nó Phun Trả Về Token Yên Tâm Thép Trực Nền Trọn Vẹn.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Giao Nạp Mũi Kim Tiêm Khám Bệnh Đáy Lõi Ngầm K8s (Probes):
Keycloak Quarkus Dựng Sẵn Tại Chân Ống Lỗ Tiêm Cổng Dữ Liệu `9000` (Management Port Tránh Chồng Đường Dài Tải Client Kẹp Chết Mạch Y Tế 8080).
```yaml
# Trích Đáy Lệnh Sinh Tử Kubernetes Container Deployment (Y Tế Chuẩn Đoán Health)
readinessProbe:
  httpGet:
    path: /health/ready
    port: 9000
  initialDelaySeconds: 20  # Đợi 20s Trục Build Khởi Khí Để Chặn Máy Chưa Lên Kéo Cổng OIDC Đâm
  periodSeconds: 10 # Gõ Cửa Đo 10 Giây Khám Nhịp 1 Cú
livenessProbe:
  httpGet:
    path: /health/live
    port: 9000
  initialDelaySeconds: 20
  failureThreshold: 3 # Gõ Cửa Báo Hư 3 Cú Là Hủy Diệt Đâm Sụp Kén Start Lại.
```
Cái Bề Bắn Health Dữ Trọng Này Cắm Thẳng Gắn Database Nền Đáy Trực (Agroal Pool). Nếu Nhanh Vọt Đứng Bóp Lỗ Kẹp Kẽ Sóng. DB Chết, Cục Probe Báo Thẳng 503 Đỏ Dữ Dội Kéo Giết Khách Rẽ Mạng Đẩy Trả Người Khác. Đỉnh Tối Ưu Bỏ Sạch Thảm Họa App Treo Nằm Rên Sụp Sóng Lỗi Lừa Trắng Bóc Data.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Trận Diệt Chủng Máy Trạm Khớp Đói Ram K8S (OOMKilled Rụng Rời Lõi Thùng):**
  - DevOps Cho Container K8s Limits Rất Đáng Thương Cỡ: `memory: 1Gi`. Nhất Định Không Đọc Vấn Kiến Trúc Trái Phân JVM Java Cực Nhát.
  - Trong Khung Java, Mặc Định Lệnh Chạy Cấu Gấp Thuật Mạng Nén JVM Không Biết Nó Bị Giam Vào Vỏ Kén K8s Dễ Thế Lủng Lỗ Kẽ Đáy. JVM Đọc Ổ Rễ Server Máy K8s Node Tổng Thấy 32GB RAM Đỉnh Rộng Bao La, JVM Liều Tự Uống Đòi RAM 4GB Cache Phẳng Bật Cấu Phình Cơn Dữ. 
  - KHÓC THÉT RỰC TRỜI: Container Vừa Bơm Ăn Lên 1.1GB, Tên Lính K8S Tuần Tra Canh Ngục Đỉnh Limit Thấy Lập Tức Bắn Hạ Rớt Ngòi Gãy Giết Tức Thời Bằng Đòn Thù **OOMKilled** Không Nương Nhe. 
  - Cách Trị Sát Tận Đáy Thép: Phải Ép Chỉ Lệnh Rút Gắn Giới Hạn Từ Khởi Động: Trút Kéo Environment: `JAVA_OPTS_APPEND=-Xmx768m` Dằn Khớp Java Lại Tự Kìm Hãm Ngậm Tiết Giảm Cơm Tránh Vượt Đỉnh Đụng Bờ Tường Của Node Văng Máy!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Việc Dùng Cụm Mạng Lưới Nhện K8s Xé Toạc Node Xa Tít Mạng (Ví dụ 1 Cụm Có 3 Máy Chủ K8S Node Của AWS Rải Trải Ở Cả Availability Zone a, b, c Khác Nhau Mạng). Tầm Tác Động Tới Giao Thức Khung Rễ JGroups Đồng Bộ Bộ Đệm Infinispan Của Các Keycloak Pod Nằm Trong Node Khác Nhau Dữ Thế Nào Gây Nghẽn Nhịp Rác Bắn Mạng?**
- **Junior:** Nó dùng wifi mạng gọi nhau lẹ lắm AWS 0ms. Chả sao.
- **Senior:** Mù Quáng Nền Tảng Kéo Trọng Layer 2 Broadcast. Lỗi Chết Mạch Cấu Trúc Ping Đồng Cỏ JGroups Khung Xưa Cũ Wildfly Dùng Đóng Mệnh Tín Hiệu Multicast/UDP Bắn Khắp Làng Báo Cáo Sự Sống Đáy Mạng Rễ Nội.
Nhưng Đưa Lên AWS Cloud K8S. Mạng Ngầm VPC Chặn Cắt Phăng Tắt Lệnh Đỉnh Tuyến Mạch Cổng Sóng Multicast Dọn Thư (Vì Tránh Đỡ Spam Bão Mạng Đáy Cụm Cloud Node Rác Lớn Lỗ Chỗ Sụp Toàn Amazon). JGroups Không Có Cách Tìm Thấy Bạn Tụ Khung Bị Xé Nhau Khóc Rỗng Khỏi Đan Lưới Clustering Đáy Xong Ai Về Trái Nấy 1 Mình Ôm Buồn Tức Lịch Sử Văng Thủng Infinispan. Lệnh Tối Ưu Giải Trí Nền Này Là Trúc Cụm Khai Mạng Ngầm PING (KUBE_PING / DNS_PING Bằng Cơ Khí Giao Giao Gọi API Kubernetes Rút Tọa Độ Lôi Rễ Đâm Nhau Nhanh Gấp 10 TCP/IP) Phá Hủy Băng Đáy Cũ Xưa Lỗi Lệnh Phế Ngụy Mạch JGroups PING Default Văng Hết Ngòi Lỗi Cụm Ác Tuyệt Nhiên Rút Ngọn Giao Thức (Nền Này Cài Sẵn Phẳng Tại Quarkus K8S Tự Gắn Động Đỉnh Miễn Điền Đuôi Code Chết).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak on K8S:** Running Keycloak in Kubernetes Guides.
- **Kubernetes StatefulSets vs Deployments:** Workload Management and Pod Identity.
