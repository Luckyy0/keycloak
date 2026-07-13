> [!NOTE]
> **Category:** Theory
> **Goal:** Nắm vững kiến trúc và phương pháp thực hiện Zero-Downtime Upgrades (Nâng cấp không gián đoạn) trong môi trường Keycloak High Availability (HA) cluster, giảm thiểu tối đa sự cố mất kết nối trong môi trường production.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong các hệ thống Enterprise đóng vai trò là xương sống như Single Sign-On (SSO), bất kỳ khoảng thời gian Downtime (gián đoạn) nào cũng đồng nghĩa với việc toàn bộ hệ sinh thái ứng dụng của công ty không thể đăng nhập. Do đó, kỹ thuật **Zero-Downtime Upgrades (ZDU)** là bắt buộc.

ZDU trong Keycloak liên quan đến việc nâng cấp từng node trong cụm (Cluster) một cách tuần tự (Rolling Upgrade) từ phiên bản N lên phiên bản N+1, đồng thời vẫn phải duy trì tính toàn vẹn của Distributed Cache (Infinispan) và Database.

**Các thách thức chính trong ZDU Keycloak:**
- **Schema Database**: Phiên bản mới có thể yêu cầu thay đổi cấu trúc bảng (ALTER TABLE). Sự thay đổi này phải tương thích ngược (Backward Compatible) để các Node phiên bản cũ vẫn có thể đọc/ghi được.
- **Infinispan Cache Cluster**: Các node ở hai phiên bản khác nhau (Mixed Cluster) có thể không đọc được dữ liệu Session của nhau nếu cấu trúc Java Serialization bị thay đổi.
- **Session Loss**: Khi khởi động lại một Node, các phiên (sessions) được cache trên node đó phải được chuyển đổi (state transfer) sang node khác một cách an toàn.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Để thực hiện nâng cấp không gián đoạn, Keycloak (thường được cấu hình qua Infinispan và Liquibase) sẽ thực hiện qua mô hình "Rolling Upgrade".

```mermaid
flowchart TD
    subgraph Cluster V1
        NodeA[Keycloak V1]
        NodeB[Keycloak V1]
    end
    DB[(Database - Schema V1)]
    LoadBalancer{Load Balancer}

    LoadBalancer --> NodeA
    LoadBalancer --> NodeB
    NodeA <--> DB
    NodeB <--> DB

    NodeA -.->|JGroups| NodeB

    step1[Bước 1: Ngắt kết nối Node A khỏi Load Balancer]
    step2[Bước 2: Cập nhật Database Schema lên V2 bằng Node A]
    step3[Bước 3: Khởi động Node A (V2)]
    step4[Bước 4: Node A vào lại Load Balancer]
    step5[Bước 5: Lặp lại quá trình cho Node B]

    step1 --> step2 --> step3 --> step4 --> step5
```

**Cơ chế cấp thấp (Mixed Version Cluster):**
Từ Keycloak bản Quarkus trở đi, nó không hỗ trợ mô hình *Mixed Version Cluster* (các node khác phiên bản giao tiếp trong cùng một JGroups channel) một cách dễ dàng và hoàn hảo do thay đổi cấu trúc internal data. Giải pháp được khuyến cáo là chia Cluster ra, hoặc sử dụng cơ chế Cross-Datacenter Replication để tạo ra một cụm V2 độc lập, sau đó chuyển dần lượng truy cập (Traffic) qua Load Balancer (ví dụ: mô hình Blue/Green Deployment).

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!WARNING]
> **Backup là Bắt buộc**: Zero-downtime không có nghĩa là 100% an toàn tuyệt đối. Luôn luôn thực hiện Full Database Backup và Export cấu hình Realm sang file JSON trước khi bắt đầu. Nếu xảy ra lỗi nghiêm trọng về data corruption, bạn phải phục hồi lại toàn bộ từ đầu.

> [!IMPORTANT]
> **Drain Sessions trước khi tắt (Graceful Shutdown)**: Đừng bao giờ tắt đột ngột tiến trình Keycloak bằng `kill -9`. Hãy dùng `SIGTERM` hoặc các API của server để Keycloak có thời gian đẩy (evict/rebalance) các session đang lưu trong bộ nhớ RAM (Infinispan) sang các Node khác trong Cluster, tránh việc người dùng bị văng ra ngoài.

- **Blue/Green Deployment**: Thực hành tốt nhất cho các bản Major Upgrade là tạo ra một cụm mới (Green) chạy song song, kết nối chung vào Database. Test kỹ trên cụm Green, sau đó trỏ DNS hoặc config Load Balancer sang Green.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sử dụng Kubernetes / OpenShift để thực hiện Rolling Update (một biến thể của ZDU nếu schema thay đổi nhỏ).
Cấu hình Deployment với `strategy` đảm bảo luôn có pod phục vụ:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1 # Không bao giờ tắt quá 1 pod cùng lúc
      maxSurge: 1       # Cho phép tạo thêm 1 pod V2 trước khi tắt pod V1
  template:
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:24.0.0
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 20
            periodSeconds: 10
```
Trong cấu hình này, Load Balancer (Kubernetes Service) chỉ gửi traffic đến pod nếu nó vượt qua `readinessProbe` (Đã nạp xong cấu hình và khởi động hoàn tất).

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Database Lock (Deadlock) khi Migration**: Khi Node đầu tiên của V2 khởi động, nó tự động chạy Liquibase script để cập nhật Database (Database Migration). Nó sẽ khóa (Lock) bảng `DATABASECHANGELOGLOCK`. Nếu script này chạy quá lâu hoặc bị crash, các node khác sẽ bị kẹt mãi mãi ở trạng thái chờ mở khóa. Bạn phải can thiệp bằng cách vào thẳng DB, cập nhật cột `LOCKED=0` trong bảng `DATABASECHANGELOGLOCK`.
- **Incompatible Session Cache**: Ở một số phiên bản nâng cấp lớn (Major), cấu trúc lưu session bị phá vỡ hoàn toàn. Nếu Rolling Update được sử dụng, những user có phiên ở Node V1 khi được chuyển giao (failover) qua Node V2 có thể gặp lỗi phân giải (Deserialization Exception) và bị buộc phải đăng nhập lại.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**Junior Level:**
1. Zero-Downtime Upgrade (ZDU) có nghĩa là gì trong ngữ cảnh của Keycloak?
   - *Đáp án:* Là quá trình nâng cấp hệ thống Keycloak lên phiên bản mới hơn mà không làm gián đoạn quyền truy cập và khả năng đăng nhập của người dùng.
2. Tại sao không thể nâng cấp ZDU nếu chỉ chạy một Node Keycloak (Standalone)?
   - *Đáp án:* Vì khi nâng cấp, phần mềm bắt buộc phải tắt đi và khởi động lại. Cần có ít nhất 2 node (Cluster) và Load Balancer để node này gánh tải khi node kia đang được nâng cấp.

**Senior Level:**
3. Giải thích tại sao việc nâng cấp qua các bản Minor (ví dụ 22.0.1 -> 22.0.5) dễ đạt được ZDU hơn là nâng cấp qua các bản Major (21.x -> 24.x)?
   - *Đáp án:* Nâng cấp Minor thường chỉ chứa các bản vá lỗi (bug fixes) và không thay đổi cấu trúc bảng Database (Liquibase schema) cũng như không thay đổi Data Structure của Infinispan. Còn bản Major thường thay đổi lớn, dẫn đến lỗi không tương thích ngược ở Data Storage hoặc Session Replication.
4. Trình bày phương pháp Blue/Green Deployment cho Keycloak. Điều gì phức tạp nhất trong phương pháp này?
   - *Đáp án:* Dựng nguyên một cụm mới (Green) chạy phiên bản mới. Điểm phức tạp nhất là cả Blue và Green phải chia sẻ chung Database. Nếu Green lên phiên bản và chạy DB Migration (ALTER schema), cụm Blue (đang phục vụ traffic) có thể bị lỗi vì schema không còn phù hợp với source code cũ.
5. "Graceful Shutdown" đóng vai trò gì trong việc bảo vệ dữ liệu Infinispan Distributed Cache khi nâng cấp?
   - *Đáp án:* Khi nhận tín hiệu tắt, Infinispan node sẽ tham gia vào quá trình State Transfer, đồng bộ hóa và di dời (rebalance) các entry session nó đang giữ sang các node đang sống, đảm bảo không có user nào bị mất session.

## 7. Tài liệu tham khảo (References)

- [Keycloak Upgrading Guide](https://www.keycloak.org/docs/latest/upgrading/index.html)
- [Infinispan Distributed Cache Upgrades](https://infinispan.org/docs/stable/titles/upgrading/upgrading.html)
- [Kubernetes Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
