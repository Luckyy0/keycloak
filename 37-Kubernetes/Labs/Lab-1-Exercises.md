> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Triển khai một cụm Keycloak an toàn trên Kubernetes sử dụng Helm Chart của Bitnami. Cấu hình kết nối với PostgreSQL và cấu hình Ingress với đường dẫn (Path) để truy cập giao diện quản trị.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là DevOps/Platform Engineer của công ty. Yêu cầu là phải dựng một máy chủ định danh Keycloak (phiên bản Quarkus) lên hạ tầng Kubernetes nội bộ. Để hệ thống có thể lưu trữ dữ liệu lâu dài (Persistent), Keycloak cần được kết nối với một Database PostgreSQL. Cuối cùng, để người dùng có thể truy cập, bạn cần định tuyến (route) thông qua NGINX Ingress Controller.

Trong bài Lab này, để tiết kiệm thời gian viết YAML thủ công và chuẩn hóa cấu hình, chúng ta sẽ sử dụng công cụ **Helm**.

## 2. Chuẩn bị Môi trường (Prerequisites)

- K8s Cluster đang hoạt động (có thể dùng Minikube, Kind hoặc Docker Desktop Kubernetes).
- Công cụ dòng lệnh `kubectl` đã được kết nối với cluster.
- Công cụ `helm` (phiên bản 3.x) đã được cài đặt trên máy.
- Một NGINX Ingress Controller đã được cài sẵn trong cluster (với minikube, chạy `minikube addons enable ingress`).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Thêm Helm Repository và tải values mặc định

1. Thêm kho lưu trữ của Bitnami vào Helm:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```
2. Tạo một Namespace riêng để triển khai:
   ```bash
   kubectl create namespace iam
   ```
3. (Tùy chọn) Tải file `values.yaml` mặc định về để xem các tham số:
   ```bash
   helm show values bitnami/keycloak > keycloak-values.yaml
   ```

### Bước 2: Tạo file cấu hình tùy chỉnh (Custom values)

Tạo một file tên là `my-values.yaml` với nội dung sau. Cấu hình này sẽ ghi đè các thiết lập mặc định của Helm chart:

```yaml
# my-values.yaml
auth:
  adminUser: admin
  adminPassword: "Password123!" # Trong thực tế phải dùng Secret

# Cấu hình Keycloak
production: false # Dùng mode dev cho Lab để bỏ qua check SSL bắt buộc ở nội bộ
proxy: edge       # Cấu hình cho phép chạy sau Ingress

# Cấu hình PostgreSQL đi kèm (Helm sẽ tự deploy 1 pod Postgres)
postgresql:
  enabled: true
  auth:
    postgresPassword: "db_password"
    username: "keycloak"
    password: "kc_password"
    database: "keycloak"

# Cấu hình Ingress
ingress:
  enabled: true
  ingressClassName: "nginx"
  hostname: keycloak.local
  path: /
  annotations:
    nginx.ingress.kubernetes.io/proxy-buffer-size: "128k"
```

### Bước 3: Triển khai (Install) bằng Helm

Chạy lệnh cài đặt Helm, trỏ tới file `my-values.yaml` và cài vào namespace `iam`:

```bash
helm install my-kc bitnami/keycloak -f my-values.yaml -n iam
```

*Helm sẽ trả về một bảng thông báo kết quả cài đặt thành công, bao gồm tên Release là `my-kc`.*

### Bước 4: Kiểm tra trạng thái Pods

Việc khởi động Keycloak và Database mất khoảng 1-3 phút. Sử dụng lệnh sau để theo dõi trạng thái khởi tạo:

```bash
kubectl get pods -n iam -w
```
Chờ đến khi cả hai Pod `my-kc-postgresql-0` và `my-kc-keycloak-0` đều hiển thị trạng thái `Running` và `READY 1/1`. Nhấn `Ctrl+C` để thoát theo dõi.

### Bước 5: Cấu hình phân giải tên miền ảo (DNS)

Vì chúng ta dùng tên miền giả `keycloak.local` ở cấu hình Ingress, bạn cần map (ánh xạ) nó vào IP của Ingress Controller trên máy của bạn.

1. Lấy IP của Ingress (nếu dùng minikube):
   ```bash
   minikube ip
   ```
   *(Giả sử kết quả là `192.168.49.2`)*
2. Sửa file `hosts` của hệ điều hành:
   - **Linux/Mac:** `sudo nano /etc/hosts`
   - **Windows:** Mở Notepad bằng quyền Admin, mở file `C:\Windows\System32\drivers\etc\hosts`
3. Thêm dòng sau vào cuối file:
   ```text
   192.168.49.2    keycloak.local
   ```
4. Lưu file lại.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Đăng nhập hệ thống

1. Mở trình duyệt web.
2. Truy cập vào URL: `http://keycloak.local/`
3. Bạn sẽ thấy trang chào mừng của Keycloak. Bấm vào **"Administration Console"**.
4. Đăng nhập bằng tài khoản:
   - Username: `admin`
   - Password: `Password123!`
5. Nếu bạn vào được giao diện Admin chứa Master Realm, Lab đã thành công.

### 4.2. Lỗi thường gặp (Troubleshooting)

- **Lỗi:** CrashLoopBackOff ở Pod Keycloak.
  - *Nguyên nhân:* Pod PostgreSQL chưa kịp khởi động hoặc không cấp phát được bộ nhớ đĩa (PVC Pending). Keycloak không kết nối được DB nên tự thoát (crash).
  - *Cách khắc phục:* Chạy `kubectl logs pod/my-kc-keycloak-0 -n iam`. Nếu lỗi kết nối DB, K8s sẽ tự động thử khởi động lại Keycloak sau vài phút. Nếu PVC Pending, kiểm tra StorageClass của cụm (`kubectl get sc`).
- **Lỗi:** Truy cập `http://keycloak.local/` bị báo "502 Bad Gateway" hoặc "404 Not Found".
  - *Nguyên nhân:* Ingress Controller chưa kết nối đúng tới Service của Keycloak, hoặc Keycloak Pod chưa Ready thực sự.
  - *Cách khắc phục:* Chạy `kubectl get ingress -n iam` để xem Ingress đã cấp IP chưa. Chạy `kubectl describe svc my-kc-keycloak -n iam` để xem có Endpoint phía sau không.
- **Lỗi:** Đăng nhập vào Admin Console bị đẩy ngược ra trang login liên tục (Infinite Redirect) hoặc báo lỗi "Invalid parameter: redirect_uri".
  - *Nguyên nhân:* Cấu hình Proxy Edge không hoạt động hoặc Ingress cấu hình sai header.
  - *Cách khắc phục:* Đảm bảo trong `my-values.yaml` có khai báo `proxy: edge` và Ingress truyền đúng Header `X-Forwarded-Proto`. (Tham số `production: false` trong giá trị Lab đã hỗ trợ giảm lỗi này cho HTTP).
