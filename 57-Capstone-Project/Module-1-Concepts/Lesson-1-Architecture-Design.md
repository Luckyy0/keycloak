# Lesson 1: Architecture Design (Thiết kế Kiến trúc Tổng thể)

> [!NOTE]
> **Category:** Architecture/Design
> **Goal:** Định hình bài toán thực tế của Đồ án (Business Case) và phác thảo thiết kế kiến trúc hệ thống (System Architecture) đáp ứng các tiêu chuẩn khắt khe nhất của ngành Tài chính (Fintech).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

**Đề bài Đồ án (The Business Case):**
Bạn là Lead Security Architect tại "NeoBank", một công ty công nghệ tài chính (Fintech) đang phát triển bùng nổ. Hiện tại NeoBank có 5 triệu khách hàng (truy cập qua Mobile App và Web SPA) và 2,000 nhân viên (truy cập qua hệ thống ERP nội bộ). Hệ thống xác thực cũ đang chạy rải rác ở từng Microservice, gây ra các lỗ hổng nghiêm trọng và chi phí bảo trì khổng lồ. 

Ban Giám đốc yêu cầu bạn thiết kế một nền tảng **Centralized IAM** dựa trên Keycloak đáp ứng 3 tiêu chí cốt lõi:
1. **Zero Downtime (Uptime 99.99%):** Hệ thống không được phép sập ngay cả khi một Data Center bị cúp điện. Nâng cấp phiên bản Keycloak không được làm rớt phiên làm việc (Session) của khách hàng đang chuyển tiền.
2. **Data Segregation (Cô lập dữ liệu):** Ranh giới bảo mật giữa Khách hàng (Customer) và Nhân viên (Employee) phải tuyệt đối. Không một nhân viên nào được dùng tài khoản nội bộ để đăng nhập vào app của khách hàng và ngược lại.
3. **Zero Trust Integration:** Các Microservices nội bộ giao tiếp với nhau phải có Token hợp lệ.

Để giải quyết bài toán này, chiến lược Kiến trúc (Architecture Strategy) của chúng ta sẽ tập trung vào **Multi-Realm Isolation** (Cô lập theo Realm) và **Multi-Node Clustering** (Chạy cụm đa nút).

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Sơ đồ mạng (Network Topology) của NeoBank được thiết kế theo chuẩn bảo mật nhiều lớp (Multi-tier Security Architecture):

```mermaid
flowchart TD
    subgraph Public Internet
        C_App[Customer Mobile App]
        E_App[Employee ERP Web - SPA]
    end

    subgraph WAF/Load Balancer [Vùng Edge / WAF]
        LB[HAProxy / AWS ALB\nSSL Offloading]
    end

    subgraph Kubernetes Cluster [Vùng Ứng dụng]
        BFF[BFF - Spring Cloud Gateway\n(For Employee ERP)]
        K_Cluster[Keycloak HA Cluster\n3 Nodes - Infinispan]
        MS_Order[Transfer Service]
        MS_Account[Account Service]
    end

    subgraph Data Tier [Vùng Dữ liệu Lõi]
        DB[(PostgreSQL HA\nPrimary + Standby)]
        AD[(Active Directory\n(For Employees))]
    end

    %% Luồng Khách hàng
    C_App -->|Bearer Token| LB
    LB --> K_Cluster

    %% Luồng Nhân viên
    E_App -->|HttpOnly Cookie| LB
    LB --> BFF
    BFF -->|Token Relay| MS_Order

    %% Giao tiếp với Database & AD
    K_Cluster <-->|JDBC| DB
    K_Cluster <-->|LDAPS| AD
    MS_Order <-->|M2M Token| MS_Account
```

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

Trong quá trình thiết kế kiến trúc cho đồ án này, bạn phải tuân thủ các quy tắc bất di bất dịch sau:

> [!IMPORTANT]
> **Chiến lược Multi-Realm (Phân tách Không gian)**
> Bạn bắt buộc phải tạo 2 Realm hoàn toàn riêng biệt trên Keycloak:
> - `neobank-customer`: Dành riêng cho khách hàng. Không kết nối LDAP. Cho phép User Registration (Đăng ký tài khoản tự do) và Social Login (Google/Apple).
> - `neobank-employee`: Dành riêng cho nhân viên. Kết nối trực tiếp với Active Directory (Read-only). Tắt tính năng tự do đăng ký. Bắt buộc kích hoạt MFA (Xác thực 2 bước bằng OTP) cho mọi tài khoản.

> [!TIP]
> **Ủy thác SSL (SSL Offloading)**
> Để cụm Keycloak đạt hiệu năng cao nhất (tiết kiệm CPU cho việc tính toán RSA), toàn bộ chứng chỉ HTTPS (TLS/SSL) sẽ được gắn tại Load Balancer (HAProxy). Keycloak chạy phía sau Load Balancer chỉ dùng HTTP (cổng 8080). Để Keycloak không hiểu lầm nó đang chạy HTTP và sinh ra các URL Redirect sai, Load Balancer bắt buộc phải gắn Header `X-Forwarded-Proto: https`.

> [!WARNING]
> **Giới hạn tấn công Brute-force**
> Đối với Realm `neobank-customer`, tin tặc sẽ liên tục dùng botnet để dò mật khẩu. Bạn phải bật tính năng **Brute Force Detection** trong cài đặt Realm, khóa tài khoản tạm thời 15 phút nếu nhập sai mật khẩu 5 lần. Đồng thời kết hợp WAF (Web Application Firewall) ở tầng Load Balancer để chặn IP độc hại.

## 4. Bảng Ma trận Phân quyền (Role & Client Matrix)

Là một Security Architect, bạn cần vạch ra danh sách các Client và Role trước khi bắt tay vào cấu hình.

**Realm: neobank-employee**
| Client ID | Loại Client | Tính năng | Mục đích |
| :--- | :--- | :--- | :--- |
| `erp-bff-client` | Confidential | Authorization Code | BFF giao tiếp với Keycloak cho Web SPA của nhân viên. |
| `transfer-service` | Bearer-only (Resource Server) | M2M / Token Relay | Cung cấp API chuyển tiền nội bộ. |

**Realm: neobank-customer**
| Client ID | Loại Client | Tính năng | Mục đích |
| :--- | :--- | :--- | :--- |
| `mobile-banking-app` | Public (PKCE) | Auth Code + PKCE | App điện thoại của khách hàng. Không lưu Client Secret. |

## 5. Trường hợp ngoại lệ (Edge Cases)

### 5.1. Dữ liệu User lên tới hàng triệu (Million-User Scale)
- **Vấn đề:** Khi `neobank-customer` đạt mốc 5 triệu user, bảng `user_entity` trong PostgreSQL trở nên cực kỳ khổng lồ. Việc tìm kiếm (Search) User trên giao diện Admin Console bị timeout (đơ toàn tập).
- **Giải pháp:** Cấu hình Indexing mạnh tay trên PostgreSQL cho các cột `email` và `username`. Về mặt kiến trúc lâu dài, có thể phải cân nhắc việc tắt tính năng tìm kiếm Full-text trên UI của Keycloak và chuyển sang đồng bộ sự kiện User ra một công cụ Elasticsearch riêng (dùng Event Listener SPI).

### 5.2. Mất đồng bộ Cụm Cache (Cluster Desync)
- **Vấn đề:** 3 Node Keycloak chạy tốt, nhưng bỗng dưng Node 3 mất kết nối mạng vài giây. Khi mạng có lại, Node 3 không chịu nhận dữ liệu Session từ Node 1 và Node 2 nữa, khiến khách hàng bị văng ra ngoài nếu rớt vào Node 3.
- **Giải pháp:** Cấu hình FD (Failure Detection) trong JGroups cực kỳ cẩn thận. Đảm bảo cấu hình biến môi trường `JGROUPS_DISCOVERY_PROTOCOL=JDBC_PING` để mượn Database làm trung tâm liên lạc, giúp các Node nhanh chóng tìm lại nhau sau khi đứt mạng thay vì "chết đứng".

## 6. Câu hỏi Bảo vệ Đồ án (Defense Questions)

**1. (Architect) Tại sao trong thiết kế này, bạn lại đặt Keycloak, BFF và Microservices nằm chung trong một mạng nội bộ (Kubernetes Cluster) nhưng lại chỉ mở port của BFF và Load Balancer ra Internet?**
- *Đáp án:* Đây là nguyên tắc giảm thiểu Bề mặt tấn công (Attack Surface). Nếu ta mở cổng trực tiếp của Keycloak ra Internet, tin tặc có thể quét tìm lỗi bảo mật trên giao diện Admin Console. Bằng cách giấu Keycloak sau Load Balancer, ta có thể dùng Load Balancer để khóa chặt đường dẫn `/admin`, chỉ cho phép các IP nội bộ công ty mới được quyền truy cập màn hình quản trị của Keycloak.

**2. (Architect) Việc tách thành 2 Realm (`neobank-employee` và `neobank-customer`) có nhược điểm gì về mặt chia sẻ tài nguyên không? Giả sử bạn có một Microservice cần cho phép cả Khách hàng và Nhân viên truy cập thì làm sao?**
- *Đáp án:* 
  - Nhược điểm là Token do Realm này sinh ra sẽ hoàn toàn vô giá trị đối với Realm kia (vì khác `issuer` URL và khác bộ Public Keys). 
  - Nếu một Microservice cần phục vụ cả 2 đối tượng, nó bắt buộc phải cấu hình thành một **Multi-Tenant Resource Server**. Tức là Spring Boot sẽ phải quản lý 2 cái `issuer-uri`, khi nhận Token nó sẽ kiểm tra trường `iss` để biết Token này thuộc về Customer hay Employee, từ đó gọi đúng Public Key tương ứng để giải mã.

## 7. Tài liệu tham khảo (References)
- **AWS Architecture Blog:** Building a Multi-Tenant Identity Solution using Keycloak.
- **NIST Special Publication 800-207:** Zero Trust Architecture.
