> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Triển khai Keycloak trên môi trường Kubernetes bằng cách sử dụng Keycloak Operator và cấu hình cơ bản cho PostgreSQL database.

## 1. Kịch bản Thực hành (Lab Scenario)
Trong bài lab này, bạn sẽ đóng vai trò là một DevOps Engineer có nhiệm vụ triển khai hệ thống Keycloak cho môi trường Production giả lập trên Kubernetes. Thay vì tự viết YAML cho StatefulSet thủ công, bạn sẽ sử dụng **Keycloak Operator** để tận dụng khả năng tự động hóa và quản lý cluster nội bộ của Infinispan. Đồng thời, bạn sẽ kết nối Keycloak với một cơ sở dữ liệu PostgreSQL đã được triển khai sẵn trong cụm.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Một cụm Kubernetes đang hoạt động (có thể sử dụng Minikube, kind, hoặc k3s).
- Công cụ `kubectl` đã được cài đặt và cấu hình kết nối tới cụm.
- Đã cài đặt OLM (Operator Lifecycle Manager) trên cụm K8s.
- Một instance PostgreSQL đang chạy trong namespace `keycloak-system` với tài khoản `keycloak` và database `keycloak`.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Cài đặt Keycloak Operator**
Tạo namespace và cài đặt Operator thông qua đường dẫn YAML chính thức của Keycloak.

```bash
kubectl create namespace keycloak-system
kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/22.0.0/kubernetes/keycloaks.k8s.keycloak.org-v1.yml
kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/22.0.0/kubernetes/keycloakrealmimports.k8s.keycloak.org-v1.yml
kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/22.0.0/kubernetes/kubernetes.yml
```
Đợi cho đến khi pod của operator chuyển sang trạng thái `Running`:
```bash
kubectl get pods -n keycloak-system -w
```

**Bước 2: Tạo Secret chứa thông tin Database và Admin**
Thay vì để lộ mật khẩu trong YAML, hãy tạo một Secret để bảo mật cấu hình:

```bash
kubectl create secret generic keycloak-db-secret \
  --from-literal=username=keycloak \
  --from-literal=password=my_secure_password \
  -n keycloak-system

kubectl create secret generic keycloak-admin-secret \
  --from-literal=admin-username=admin \
  --from-literal=admin-password=admin_password \
  -n keycloak-system
```

**Bước 3: Khai báo Keycloak Custom Resource (CR)**
Tạo một tệp tin tên là `keycloak.yaml` với nội dung sau:

```yaml
apiVersion: k8s.keycloak.org/v2alpha1
kind: Keycloak
metadata:
  name: my-keycloak
  namespace: keycloak-system
spec:
  instances: 2
  db:
    vendor: postgres
    host: "postgres-service.keycloak-system.svc.cluster.local"
    usernameSecret:
      name: keycloak-db-secret
      key: username
    passwordSecret:
      name: keycloak-db-secret
      key: password
  http:
    tlsSecret: my-tls-secret
  hostname:
    hostname: keycloak.local
```

Áp dụng cấu hình vào Kubernetes:
```bash
kubectl apply -f keycloak.yaml -n keycloak-system
```

**Bước 4: Cấu hình Ingress / Port Forwarding**
Nếu bạn chưa có Ingress Controller, có thể tạm thời sử dụng Port Forward để truy cập giao diện quản trị:

```bash
kubectl port-forward svc/my-keycloak-service 8443:8443 -n keycloak-system
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra Pod và Cluster Infinispan:**
```bash
kubectl get statefulset -n keycloak-system
kubectl get pods -l app=keycloak -n keycloak-system
```
Đảm bảo rằng có 2 pod (ví dụ `my-keycloak-0` và `my-keycloak-1`) đang ở trạng thái `Running` và `READY 1/1`.

**Kiểm tra Log để xác nhận kết nối DB:**
```bash
kubectl logs my-keycloak-0 -n keycloak-system | grep "Agroal"
```
> [!TIP]
> Bạn sẽ thấy các dòng log xác nhận kết nối tới PostgreSQL thành công. Nếu thấy lỗi `Connection refused`, hãy kiểm tra lại service PostgreSQL đã được khai báo đúng tên trong phần `db.host` chưa.

**Truy cập Giao diện Web:**
Mở trình duyệt và truy cập `https://localhost:8443`. Đăng nhập với tài khoản `admin` và mật khẩu `admin_password` đã tạo trong Secret ở Bước 2.

> [!WARNING]
> Nếu bạn gặp lỗi SSL Certificate khi truy cập HTTPS cục bộ, hãy bỏ qua cảnh báo hoặc tạo một TLS Secret hợp lệ cho Ingress của bạn trên môi trường thực tế.
