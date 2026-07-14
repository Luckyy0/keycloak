# Lesson 2: Deployment Strategy (Chiến lược Triển khai K8s)

> [!NOTE]
> **Category:** Infrastructure/Deployment
> **Goal:** Xây dựng chiến lược triển khai hệ thống Keycloak lên môi trường Kubernetes (Production), quản lý vòng đời Pod, Cấu hình bảo mật, và Tự động mở rộng (Autoscaling).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Sau khi đã chốt Sơ đồ Kiến trúc (Lesson 1), bước tiếp theo của Đồ án Capstone là quyết định "Làm thế nào để đưa hệ thống này chạy thực tế?". Đối với môi trường Enterprise-grade, **Kubernetes (K8s)** là sự lựa chọn duy nhất.

Khi triển khai Keycloak lên K8s, bài toán lớn nhất là **Quản lý Trạng thái (State Management)**. Keycloak tuy là ứng dụng Stateless ở tầng xử lý logic, nhưng lại mang tính Stateful ở tầng Cache (Infinispan).

Do đó, chiến lược triển khai chuẩn xác nhất là:
1. **Dùng StatefulSet thay cho Deployment:** Để các Pod Keycloak có tên miền cố định (ví dụ `keycloak-0`, `keycloak-1`), giúp giao thức JGroups tìm thấy nhau dễ dàng và ổn định hơn.
2. **Tối ưu hóa Image (Pre-build):** Không sử dụng Image mặc định quay từ Docker Hub và cấu hình lúc khởi động. Phải chạy lệnh `kc.sh build` trong Dockerfile để sinh ra một Custom Image "đã được tối ưu sẵn" rồi mới đẩy lên K8s.
3. **External Database:** PostgreSQL bắt buộc phải nằm NGOÀI Kubernetes Cluster (sử dụng các dịch vụ Managed Database như AWS RDS, Google Cloud SQL) để đảm bảo độ an toàn dữ liệu và dễ dàng backup.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Sơ đồ dưới đây mô tả quá trình Request đi từ ngoài Internet vào tận bên trong Pod Keycloak thông qua các thành phần của Kubernetes:

```mermaid
flowchart TD
    User([Người dùng Internet])
    
    subgraph K8s Cluster [Kubernetes Cluster (Production)]
        Ingress[Ingress Controller\n(Nginx/Traefik)]
        Service[K8s Service\n(ClusterIP - Headless)]
        
        subgraph StatefulSet [StatefulSet (Keycloak)]
            Pod0[Pod: keycloak-0\n(Cache: 1GB)]
            Pod1[Pod: keycloak-1\n(Cache: 1GB)]
            Pod2[Pod: keycloak-2\n(Cache: 1GB)]
        end
        
        HPA[Horizontal Pod Autoscaler]
    end

    DB[(AWS RDS\nPostgreSQL)]

    %% Traffic Flow
    User -->|HTTPS :443| Ingress
    Ingress -->|HTTP :8080| Service
    Service --> Pod0 & Pod1 & Pod2

    %% HPA Monitoring
    HPA -.->|Giám sát CPU/Memory| StatefulSet

    %% Database Connection
    Pod0 -->|JDBC| DB
    Pod1 -->|JDBC| DB
    Pod2 -->|JDBC| DB
```

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Quản lý Bí mật (Secret Management)**
> Tuyệt đối KHÔNG cấu hình Mật khẩu Database, Admin Password, hay LDAP Bind Password vào trong file `ConfigMap` hoặc file YAML. Kẻ gian có quyền read K8s sẽ lấy được toàn bộ. Phải sử dụng K8s `Secret` hoặc tích hợp với **HashiCorp Vault / External Secrets Operator** để tự động bơm mật khẩu vào Environment Variable của Pod lúc runtime.

> [!TIP]
> **Tùy chỉnh Liveness & Readiness Probes**
> K8s sử dụng "Probes" để kiểm tra xem Keycloak đã sẵn sàng nhận Request chưa. Keycloak mất từ 10-30 giây để khởi động. Nếu cấu hình thời gian kiểm tra quá ngắn, K8s sẽ tưởng Keycloak bị treo và liên tục chém chết (kill) Pod đó. Bạn phải trỏ đường dẫn kiểm tra vào API Health Check chuẩn của Keycloak: `/health/ready` và `/health/live`.

> [!WARNING]
> **OOMKilled (Out of Memory)**
> Keycloak chạy bằng máy ảo Java (JVM). Nếu JVM dùng quá nhiều RAM (vượt quá `limits.memory` bạn cấu hình trong K8s), K8s sẽ ném ra lỗi `OOMKilled` và tiêu diệt Pod. **Quy tắc vàng:** Cấu hình `limits.memory` của K8s phải luôn lớn hơn cờ `Xmx` (Max Heap) của JVM ít nhất 25% để dành chỗ cho Metaspace và Off-heap memory.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ví dụ trích đoạn cấu hình K8s StatefulSet cho Keycloak trên Production:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: keycloak
spec:
  serviceName: keycloak-headless # Dành cho JGroups DNS_PING
  replicas: 3
  template:
    spec:
      containers:
        - name: keycloak
          image: myregistry.com/neobank-keycloak:v1.0.0
          args: ["start", "--optimized"] # Đã build sẵn
          env:
            - name: KC_DB_URL
              value: "jdbc:postgresql://rds-neobank.aws.com/keycloak"
            - name: KC_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak-db-secret
                  key: password
            - name: JAVA_OPTS_APPEND
              value: "-Xms1024m -Xmx1024m" # Cố định Heap size
          resources:
            requests:
              cpu: "1000m"
              memory: "1536Mi"
            limits:
              cpu: "2000m"
              memory: "1536Mi" # Lớn hơn Xmx 512MB
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 9000 # Cổng Management
            initialDelaySeconds: 20
            periodSeconds: 10
```

## 5. Trường hợp ngoại lệ (Edge Cases)

### 5.1. Thảm họa Scale Down mất Session
- **Vấn đề:** 8h00 sáng, hệ thống tự Scale (HPA) từ 3 Pod lên 10 Pod để chịu tải. 12h00 trưa, tải giảm, HPA tự động chém chết 7 Pod để đưa về 3 Pod. Đột nhiên, hàng chục ngàn khách hàng bị mất Session (văng ra màn hình đăng nhập). Lý do: K8s chém Pod quá nhanh, Infinispan chưa kịp chép dữ liệu Session từ các Pod bị chém sang các Pod còn sống.
- **Giải pháp:** Cấu hình **Graceful Shutdown** (Kéo dài thời gian chết). Thêm cấu hình `terminationGracePeriodSeconds: 60` vào K8s, và sử dụng `preStop` hook trong Container để gọi lệnh tạm dừng nhận request, cho phép Infinispan có đủ 60 giây để di tản dữ liệu Session sang các Node an toàn trước khi tắt máy.

### 5.2. Chết tiệt vì Health Check
- **Vấn đề:** Một Node Keycloak đang chịu tải rất lớn (100% CPU). Nó vẫn đang xử lý Request nhưng hơi chậm. Hệ thống K8s Liveness Probe kiểm tra `/health/live` và bị timeout do CPU bận. K8s nghĩ rằng Pod này bị treo nên "nhẫn tâm" restart nó. Việc restart làm mất luôn 1 phần ba bộ Cache của cả hệ thống, gây quá tải domino lên các Node còn lại.
- **Giải pháp:** Chỉnh thời gian `timeoutSeconds` và `failureThreshold` của Liveness Probe lên cao hơn trong môi trường Production (ví dụ: cho phép kiểm tra thất bại 3 lần, mỗi lần cách nhau 15 giây mới được restart).

## 6. Câu hỏi Bảo vệ Đồ án (Defense Questions)

**1. (DevOps) Tại sao bạn lại chọn lưu trữ Database của Keycloak (PostgreSQL) ra một dịch vụ Managed Service của AWS/GCP thay vì chạy nó thành một StatefulSet chung trong K8s?**
- *Đáp án:* Cơ sở dữ liệu (Database) là "trái tim" của hệ thống. Chạy DB trong K8s tiềm ẩn rủi ro khổng lồ về Quản lý ổ đĩa lưu trữ (Persistent Volumes), I/O thắt cổ chai, và khó khăn trong việc sao lưu (Backup) tự động, phục hồi thảm họa (Point-in-Time Recovery). Giao phó DB cho một dịch vụ Cloud (RDS) giúp đội DevOps kê cao gối ngủ, đảm bảo tính vẹn toàn dữ liệu, để K8s chỉ lo việc chạy tính toán (Compute).

**2. (Architect) Nếu tôi thay thế Keycloak Container mặc định bằng phiên bản Keycloak Operator do JBoss cung cấp thì có điểm gì lợi hơn so với tự viết file YAML?**
- *Đáp án:* Dùng **Keycloak Operator** (mô hình CRD - Custom Resource Definition) là một Best Practice ở level cao hơn. Operator không chỉ tự sinh file YAML, mà nó còn đóng vai trò như một "Quản trị viên ảo" chạy bên trong K8s. Nó biết cách tự động cấu hình JGroups, biết tự động Rebalance Cache một cách an toàn khi Scale up/down, và tự động xử lý quá trình Upgrade phiên bản Keycloak mà không gây Downtime. Tuy nhiên, nó đòi hỏi kiến thức vận hành Operator phức tạp hơn.

## 7. Tài liệu tham khảo (References)
- **Keycloak K8s Guide:** Running Keycloak in Kubernetes.
- **Kubernetes Documentation:** StatefulSets and Probes.
