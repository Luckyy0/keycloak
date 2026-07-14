# Lesson 3: Security & Performance Checklist

> [!NOTE]
> **Category:** Security/Operations
> **Goal:** Cung cấp bộ tiêu chuẩn rà soát (Final Checklist) bắt buộc trước khi đưa hệ thống Enterprise IAM của đồ án lên môi trường thực tế (Go-Live).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Hệ thống Quản lý Định danh và Truy cập (IAM) chính là "cửa trước" (Front Door) của toàn bộ doanh nghiệp. Bất kỳ Microservice nào phía sau cũng dựa vào Token do Keycloak cấp phát để tin tưởng người dùng. Do đó, nếu cấu hình Keycloak sai, hậu quả không chỉ là lỗi ứng dụng, mà là thảm họa an toàn thông tin (Data Breach).

Trước khi Go-Live, một Security Architect bắt buộc phải thực hiện quy trình **Rà soát Độc lập (Independent Audit)** dựa trên một Checklist khắt khe. Checklist này được chia làm hai mảng chính:
1. **Security (Bảo mật):** Chống lại các cuộc tấn công nhắm vào Identity (OWASP Top 10, Brute-force, Session Hijacking).
2. **Performance (Hiệu năng):** Đảm bảo hệ thống không bị "đột tử" khi gặp lưu lượng đột biến (Spike Traffic) hoặc bị tấn công Từ chối dịch vụ (DoS/DDoS).

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Một trong những cơ chế bắt buộc phải cấu hình trước khi Go-live là **Event Logging Pipeline (Đường ống Log sự kiện)**. 
Nếu hệ thống bị Hack, bạn phải biết Hacker đã vào bằng cách nào. Mặc định Keycloak không lưu quá nhiều log vào Database để tránh phình to dữ liệu. Bạn phải cấu hình đẩy Log ra ngoài.

```mermaid
flowchart LR
    K[Keycloak HA Cluster]
    DB[(PostgreSQL)]
    ELK[Elasticsearch / Splunk]
    Alert[Hệ thống Cảnh báo\n(Slack / PagerDuty)]

    K -->|Chỉ lưu Log Admin| DB
    K -->|Syslog / Filebeat\n(Mọi sự kiện Login)| ELK
    ELK -->|Phát hiện Login từ IP lạ| Alert
```

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

Dưới đây là bảng Checklist "Sống còn" bạn phải kiểm tra trong Đồ án:

### 3.1. Security Checklist (Bảo mật)

> [!IMPORTANT]
> **1. Khóa chặt Master Realm**
> Tài khoản `admin` của Master Realm có sức mạnh tối thượng. Bạn phải tạo một tài khoản Admin mới với tên khó đoán (ví dụ: `neo_sys_admin`), BẮT BUỘC kích hoạt OTP (MFA) cho tài khoản này. Sau đó **Disable (Vô hiệu hóa)** tài khoản `admin` mặc định.
> Đồng thời, cấu hình Load Balancer / WAF chặn mọi truy cập từ Internet vào đường dẫn `/admin` và `/realms/master`, chỉ cho phép các IP thuộc dải VPN của bộ phận IT.

> [!WARNING]
> **2. Tuyệt đối không dùng Client Secret cho Frontend**
> Hãy rà soát toàn bộ các Client dành cho Mobile App và React/Vue. Chúng BẮT BUỘC phải là loại `Public Client` và sử dụng luồng `Authorization Code with PKCE`. Nếu phát hiện có mã `Client Secret` nằm trong mã nguồn của JS, hệ thống của bạn đã bị hổng.

> [!TIP]
> **3. Luân chuyển Khóa Mã hóa (Key Rotation)**
> Keycloak dùng một cặp khóa RSA để ký và xác thực JWT. Khóa này mặc định có tuổi thọ 10 năm. Đây là thảm họa nếu khóa bị lộ. Bạn phải thiết lập chính sách **Key Rotation** (tự động sinh khóa RSA mới định kỳ 90 ngày) trong cài đặt `Realm Settings -> Keys`.

### 3.2. Performance Checklist (Hiệu năng)

> [!IMPORTANT]
> **1. Giới hạn Connection Pool Database**
> Khi chạy K8s HPA (Tự động mở rộng), số lượng Pod Keycloak có thể tăng vọt. Hãy đảm bảo công thức sau luôn đúng: 
> `(Số lượng Pod tối đa) * (Max Pool Size của từng Pod) < Max Connections của PostgreSQL`. Nếu sai công thức, Database sẽ sập.

> [!TIP]
> **2. Tối ưu thuật toán băm Mật khẩu (Password Hashing)**
> Keycloak sử dụng thuật toán PBKDF2 với hàng trăm nghìn vòng lặp (iterations) để mã hóa mật khẩu, khiến việc bẻ khóa (crack) là bất khả thi. Tuy nhiên, nó cực tốn CPU. Hãy chạy Load Test (ví dụ bằng JMeter) để tìm ra số vòng lặp phù hợp. Nếu để mặc định, máy chủ của bạn có thể sẽ 100% CPU ngay khi 100 người cùng đăng nhập.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ví dụ cấu hình Nginx (WAF) để khóa đường dẫn Admin:

```nginx
server {
    listen 443 ssl;
    server_name sso.neobank.com;

    # Cấu hình Security Headers bắt buộc
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "DENY" always;
    add_header Content-Security-Policy "default-src 'self';" always;

    # Chặn đứng mọi truy cập vào Admin Console từ ngoài Internet
    location ~* ^/(auth/)?(admin|realms/master) {
        # Chỉ cho phép dải IP nội bộ hoặc VPN
        allow 10.0.0.0/8;
        allow 192.168.100.0/24;
        deny all;
        
        proxy_pass http://keycloak_cluster;
        # ... các proxy headers
    }

    # Cho phép người dùng bình thường truy cập
    location / {
        proxy_pass http://keycloak_cluster;
    }
}
```

## 5. Trường hợp ngoại lệ (Edge Cases)

### 5.1. Bão Token Refresh (Refresh Token Storm)
- **Vấn đề:** Khách hàng mở ứng dụng Web SPA trên 20 tab trình duyệt khác nhau. Token hết hạn, mã Javascript trên cả 20 tab đồng loạt gọi lên API Gateway để xin đổi Refresh Token lấy Access Token mới. Cụm Keycloak nhận 20 request cùng lúc từ một user, gây nghẽn băng thông và lãng phí CPU.
- **Giải pháp:** Đối với kiến trúc BFF (đã học ở Chương 56), BFF phải có trách nhiệm giữ Refresh Token (không trả về trình duyệt) và tự động quản lý việc Refresh một cách đồng bộ (Synchronized) bằng Lock nội bộ, chỉ gửi 1 request lên Keycloak, sau đó phân phát Access Token mới cho các lời gọi.

### 5.2. Lộ Token qua Log (Token Leakage via Logs)
- **Vấn đề:** Cấu hình API Gateway hoặc Microservice in toàn bộ `Header` của Request ra file Log (File văn bản) để debug. File log này được thu thập về Kibana. Hậu quả: Bất kỳ lập trình viên nào có quyền xem Kibana đều có thể copy được JWT Bearer Token của sếp Tổng giám đốc đang thao tác và dùng nó để gọi API trái phép.
- **Giải pháp:** Cấu hình thư viện Logging trong Spring Boot (Logback) hoặc Nginx để bóc tách (Mask/Obfuscate) và che giấu toàn bộ nội dung của Header `Authorization`. Tuyệt đối không log Token.

## 6. Câu hỏi Bảo vệ Đồ án (Defense Questions)

**1. (Senior) Bạn vừa nhận bàn giao một hệ thống Keycloak chuẩn bị Go-Live. Nếu chỉ có 5 phút để kiểm tra, bạn sẽ kiểm tra 3 điểm yếu cấu hình nào đầu tiên?**
- *Đáp án:* 
  1. **HTTPS/TLS Status:** Kiểm tra xem Keycloak có bị vô tình cấu hình cờ `Require SSL = None` cho external request không. Bắt buộc phải là `External requests`.
  2. **Valid Redirect URIs của Clients:** Kiểm tra xem có Client nào cấu hình Redirect URI là `*` (Wildcard) không. Nếu có, đây là lỗ hổng Open Redirect chết người, Hacker có thể trộm được Authorization Code. Phải điền chính xác đường dẫn HTTPS.
  3. **Brute Force Protection:** Kiểm tra xem tính năng chống dò mật khẩu ở Realm Settings đã được BẬT chưa. 

**2. (Architect) Đứng ở góc độ Kiến trúc, làm sao để bạn giám sát (Monitor) và đảm bảo rằng Keycloak sẽ không bị Sập (OOM - Out of Memory) khi tải tăng đột biến?**
- *Đáp án:* Tôi bắt buộc phải bật tính năng **Keycloak Metrics (Prometheus)** và kéo dữ liệu đó về Dashboard Grafana. Tôi sẽ đặt các Cảnh báo (Alerts) đỏ ở 3 chỉ số:
  - `jvm_memory_bytes_used`: Nếu Heap nhích đến 85%, báo động.
  - `keycloak_db_connections_active`: Tránh kiệt quệ Pool.
  - `jgroups_view_size`: Đảm bảo số lượng Node đang nối mạng với nhau khớp với số lượng Pod trên K8s (Tránh Split-brain).

## 7. Tài liệu tham khảo (References)
- **OWASP:** Top 10 Proactive Controls (IAM Segment).
- **CIS Benchmarks:** Security Configurations.
