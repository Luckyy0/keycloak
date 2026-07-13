> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Thực hành thiết lập Keycloak để phơi bày các điểm cuối đo lường (Health Check & Metrics), đồng thời phân tách luồng traffic quản trị bằng cổng Management riêng biệt.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là kỹ sư DevOps phụ trách cấu hình lại một máy chủ Keycloak trước khi đưa lên môi trường Kubernetes. Yêu cầu của đội hệ thống:
1.  Hệ thống Kubernetes cần có đường dẫn để ping xem Keycloak đã sẵn sàng nhận kết nối hay chưa (Health Check).
2.  Hệ thống Prometheus cần kéo dữ liệu số đo tài nguyên hệ thống định kỳ (Metrics).
3.  Tuyệt đối không được phơi bày các đường dẫn nhạy cảm này ra cổng mạng công cộng (8080). Bạn phải khởi chạy chúng trên một cổng quản trị bí mật (Management Port: 9000).

Trong bài Lab này, chúng ta sẽ cấu hình Keycloak sử dụng file cấu hình tiêu chuẩn và kiểm thử bằng Docker cục bộ.

## 2. Chuẩn bị Môi trường (Prerequisites)

*   **Docker:** Hoạt động ổn định trên máy của bạn (Docker Desktop, Podman hoặc Docker Engine).
*   **Terminal/Command Prompt:** Để thực thi các dòng lệnh.
*   **cURL:** Công cụ dùng để test HTTP requests.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Tạo file cấu hình `keycloak.conf`
Tạo một thư mục trống có tên `kc-monitoring-lab`. Mở Terminal tại thư mục đó và tạo một file có tên `keycloak.conf` với nội dung như sau:

```properties
# Kích hoạt tính năng đo lường Metrics (Micrometer)
metrics-enabled=true

# Kích hoạt tính năng Health Checks nội bộ (SmallRye Health)
health-enabled=true

# Chỉ định cổng kết nối riêng biệt cho Management API
# Keycloak sẽ phục vụ user ở port 8080, và API nội bộ ở port 9000
http-management-port=9000
```

### Bước 2: Khởi chạy Keycloak bằng Docker với cấu hình
Sử dụng câu lệnh sau để chạy container Keycloak. Chú ý rằng ta đang "map" file cấu hình vừa tạo vào đường dẫn hệ thống của container bằng cờ `-v`, và mở cả 2 cổng 8080 cùng 9000 ra môi trường máy chủ cục bộ.

```bash
docker run --name keycloak-monitor \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -p 8080:8080 \
  -p 9000:9000 \
  -v $(pwd)/keycloak.conf:/opt/keycloak/conf/keycloak.conf \
  quay.io/keycloak/keycloak:latest \
  start-dev
```
*Ghi chú cho người dùng Windows (PowerShell):* Thay thế `$(pwd)` bằng `${PWD}`.

Đợi trong vài giây để quá trình khởi động (Start-up) kết thúc. Chú ý trong Log console, bạn sẽ thấy dòng thông báo Keycloak đã lắng nghe tại 2 cổng (Một cho HTTP, một cho Management).

### Bước 3: Kiểm thử Health Checks
Mở một cửa sổ Terminal mới (không tắt tiến trình Docker cũ). Thử giả lập hành vi của Kubernetes Kubelet bằng cURL.

**Kiểm tra tính mạng (Liveness):**
```bash
curl -i http://localhost:9000/health/live
```
*Kỳ vọng:* Trả về HTTP 200 OK và body dạng JSON `{"status": "UP", "checks": [{"name": "Keycloak lifecycle", "status": "UP"}]}`.

**Kiểm tra sự sẵn sàng (Readiness):**
```bash
curl -i http://localhost:9000/health/ready
```
*Kỳ vọng:* Trả về HTTP 200 OK và chứa trạng thái của Database Connections (Database connections health check).

**Thử nghiệm cổng bảo mật (Thất bại có chủ ý):**
Thử gọi API quản lý trên cổng công cộng dành cho người dùng cuối (8080):
```bash
curl -i http://localhost:8080/health/live
```
*Kỳ vọng:* Hệ thống báo HTTP 404 Not Found. Bạn đã thành công trong việc giấu đi các endpoint nội bộ.

### Bước 4: Kiểm thử Prometheus Metrics
Giả lập hành vi Cào dữ liệu (Scraping) của Prometheus Server.

```bash
curl -i http://localhost:9000/metrics
```
*Kỳ vọng:* Trả về HTTP 200 OK. Body phản hồi không phải là JSON, mà là một đoạn văn bản khổng lồ tuân theo chuẩn định dạng Prometheus Text Format.
Ví dụ cấu trúc sẽ có dạng:
```text
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="G1 Survivor Space",} 1.048576E7
```
Thử tìm kiếm trong Output (bằng grep hoặc mắt) xem có thông tin đo lường về `agroal` (Connection Pool) hoặc `undertow` (HTTP Server) hay không.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

*   **Dấu hiệu thành công:** Các lệnh CURL vào cổng `9000` đều có Response Code 200. Các định dạng JSON (Health) và Text (Metrics) xuất hiện đầy đủ cấu trúc dữ liệu theo đúng đặc tả kỹ thuật.
*   **Troubleshooting (Xử lý sự cố):**
    *   *Lỗi curl: (7) Failed to connect to localhost port 9000:* Nghĩa là tiến trình Docker khởi chạy thất bại, hoặc bạn đã quên truyền cờ `-p 9000:9000` trong lệnh `docker run`. Dừng container, xóa nó bằng `docker rm keycloak-monitor` và chạy lại.
    *   *Endpoint trả về 404 trên Port 9000:* Có thể Keycloak chưa nạp đúng file `keycloak.conf`. Hãy kiểm tra lại đường dẫn mount Volume (`-v`) xem đã trỏ chính xác vị trí thư mục tuyệt đối trên máy tính của bạn chưa.
    *   *File JSON Health không hiển thị thông tin DB:* Nếu bạn đang chạy với cờ `start-dev`, Keycloak mặc định dùng H2 Database nội bộ nên các bước kiểm thử Database Pool có thể ít hơn so với Postgres Production.
