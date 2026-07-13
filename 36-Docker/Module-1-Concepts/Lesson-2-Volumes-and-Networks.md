> [!NOTE]
> **Category:** Theory
> **Goal:** Hiểu sâu về cách quản lý dữ liệu bền vững (Volumes) và cấu trúc mạng lưới (Networks) cô lập khi triển khai Keycloak bằng Docker.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Khi triển khai Keycloak trên môi trường Container (Docker), dữ liệu (state) và mạng (network) là hai yếu tố sống còn để đảm bảo hệ thống có thể phục hồi sau thảm họa, bảo mật nội bộ và có tính sẵn sàng cao.

**Docker Volumes:** 
Container có tính chất **ephemeral** (tạm thời) – mọi dữ liệu được ghi vào hệ thống tệp của container sẽ biến mất khi container bị xóa. Keycloak lưu phần lớn trạng thái (Users, Realms, Clients) trong Cơ sở dữ liệu quan hệ (PostgreSQL, MySQL). Do đó, Database cần được gắn kết với Volumes để lưu trữ bền vững (Persistent Storage). Tuy nhiên, không chỉ có DB, bản thân Keycloak cũng cần Volumes để quản lý các tệp cấu hình (Config files), chứng chỉ bảo mật (TLS/SSL Certificates), và Custom Themes mà không cần phải build lại Image liên tục.

**Docker Networks:** 
Trong một kiến trúc thực tế, Keycloak không bao giờ đứng đơn độc. Nó giao tiếp với Database, có thể đứng sau một Reverse Proxy (Nginx/Traefik) và liên kết với các Microservices khác. Việc sử dụng Docker Networks nhằm tạo ra sự **cô lập (Isolation)**. Chẳng hạn, Database không bao giờ được phép phơi bày (expose) port ra ngoài Internet, mà chỉ cho phép Keycloak truy cập thông qua một mạng nội bộ riêng tư.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Dưới đây là sơ đồ mạng lý tưởng mô tả cách kết hợp Volumes và Networks trong hệ thống sử dụng Docker Compose.

```mermaid
graph TD
    subgraph "External Network (Internet)"
        User[Người dùng / Trình duyệt]
    end

    subgraph "Docker Host"
        subgraph "Public Network (Bridge)"
            Proxy[Reverse Proxy: Nginx]
        end

        subgraph "Internal Network (Isolated)"
            KC[Keycloak Container]
            DB[(PostgreSQL Container)]
        end

        %% Volumes Mapping
        VolDB[DB Volume: /var/lib/postgresql/data]
        VolCert[Certs Volume: /etc/nginx/certs]
        
        DB -.->|Ghi dữ liệu bền vững| VolDB
        Proxy -.->|Đọc chứng chỉ| VolCert
    end

    User -->|HTTPS 443| Proxy
    Proxy -->|HTTP 8080| KC
    KC <-->|TCP 5432| DB

    classDef network fill:#f9f9f9,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    class "Public Network (Bridge)" network;
    class "Internal Network (Isolated)" network;
```

**Giải thích chi tiết:**
1. **Routing:** Người dùng từ Internet truy cập thông qua mạng diện rộng. Reverse Proxy nhận yêu cầu này.
2. **Public Network:** Reverse Proxy được cấu hình trên một mạng mà cổng 443 (HTTPS) được map ra ngoài (publish port).
3. **Internal Network:** Keycloak và Database nằm chung trong một `Internal Network`. Cổng 5432 của DB và cổng 8080 của Keycloak không cần map ra host máy chủ (không dùng `-p 5432:5432`). Reverse Proxy chuyển tiếp Request vào Keycloak thông qua tên DNS của container (VD: `http://keycloak:8080`).
4. **Volume Binding:** Container Database ghi dữ liệu vào thư mục nội bộ của nó (VD `/var/lib/postgresql/data`), nhưng thư mục này được map trực tiếp xuống phân vùng đĩa vật lý của máy chủ Host bằng Docker Volume. Khi Container bị crash hoặc update bản mới, dữ liệu vẫn còn nguyên ở Volume và gắn lại cho Container mới.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

- **Nguyên tắc Least Privilege cho Network:** Database Container không được publish bất kỳ cổng nào ra host (`ports` block trong docker-compose phải bị bỏ trống đối với DB). Chỉ các container nằm trong cùng mạng mới có thể gọi nó.
- **Sử dụng Named Volumes thay vì Bind Mounts:** Đối với Database, hãy dùng Named Volumes (`volumes: - db-data:/var/lib/...`) thay vì Bind Mounts (`volumes: - ./data:/var/lib/...`) vì Named Volumes do Docker quản lý, tối ưu I/O trên mọi hệ điều hành (kể cả Windows/Mac) và giải quyết các lỗi phân quyền (Permission issues).
- **Phân quyền Read-Only cho Volumes cấu hình:** Các tệp như cấu hình SSL/TLS (`certs`), hoặc `realm-export.json` khi mount vào Keycloak nên được cấu hình ở chế độ Read-Only (thêm `:ro` ở cuối khai báo volume) để tránh bị ứng dụng ghi đè hoặc chỉnh sửa ngoài ý muốn.

> [!WARNING]
> Không bao giờ lưu trữ mật khẩu DB, cấu hình nhạy cảm dưới dạng tệp plain text ở các thư mục Bind Mount dễ bị rò rỉ. Hãy sử dụng Docker Secrets hoặc Environment Variables được quản lý an toàn.

> [!IMPORTANT]
> Khi thiết lập Keycloak sau Reverse Proxy, bạn BẮT BUỘC phải thiết lập biến `KC_PROXY=edge` (hoặc cấu hình tương tự tùy phiên bản) và đảm bảo Proxy truyền đúng các Headers `X-Forwarded-For`, `X-Forwarded-Proto` để Keycloak nhận diện đúng URL gốc.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

File `docker-compose.yml` tuân thủ tiêu chuẩn bảo mật và lưu trữ bền vững:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: strong_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend-network
    # Tuyệt đối không dùng 'ports: - "5432:5432"' ở đây

  keycloak:
    image: quay.io/keycloak/keycloak:latest
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: strong_password
      KC_PROXY: edge
      KC_HOSTNAME: auth.example.com
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin_password
    command: start
    ports:
      - "8080:8080"
    volumes:
      # Map custom theme (Read Only)
      - ./custom-theme:/opt/keycloak/themes/custom-theme:ro
    networks:
      - backend-network
    depends_on:
      - postgres

volumes:
  postgres_data:

networks:
  backend-network:
    driver: bridge
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Lỗi Permission Denied khi dùng Bind Mount:** Nếu bạn mount một thư mục host (`./db-data`) vào container Database, DB có thể không chạy được vì User chạy trong container không có quyền ghi lên thư mục host (đặc biệt phổ biến trên Linux Host).
  - **Khắc phục:** Sử dụng Docker Named Volumes thay vì Bind Mount, hoặc dùng `chown` để thay đổi chủ sở hữu của thư mục host khớp với UID của User chạy trong Container (thường là UID 999 cho Postgres).
- **Mạng Container bị lỗi DNS Resolution:** Keycloak không kết nối được đến DB với lỗi `UnknownHostException: postgres`.
  - **Khắc phục:** Kiểm tra lại khai báo `networks`. Đảm bảo cả hai services đều nằm trong cùng một custom network. Docker cung cấp tính năng DNS nội bộ tự động phân giải tên Service (VD `postgres`) thành IP của container đó.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **Junior:** Phân biệt Bind Mounts và Named Volumes trong Docker?
   - *Đáp án:* Bind Mounts map trực tiếp một thư mục có đường dẫn cụ thể trên Host vào Container. Named Volumes là vùng lưu trữ do Docker tạo và quản lý hoàn toàn, thường nằm trong `/var/lib/docker/volumes/...`, an toàn và dễ quản lý hơn cho database.
2. **Junior:** Tại sao ta không cần cấu hình `ports: - "5432:5432"` cho service Database trong Docker Compose khi chạy với Keycloak?
   - *Đáp án:* Vì Keycloak gọi trực tiếp Database thông qua Docker Internal Network nội bộ. Mở cổng ra ngoài chỉ tiềm ẩn rủi ro bảo mật nếu kẻ gian truy cập thẳng vào DB.
3. **Senior:** Điều gì xảy ra nếu tôi không cấu hình Volume cho Postgres mà Container Database bị khởi động lại?
   - *Đáp án:* Nếu không map Volume, toàn bộ dữ liệu (Users, Realms) sẽ lưu trên Container Layer mỏng. Khi container bị xóa (VD `docker-compose down`), mọi dữ liệu sẽ mất hoàn toàn.
4. **Senior:** Khi Keycloak chạy sau Nginx (trên một network khác) và map Volume chứng chỉ TLS ở Nginx, Keycloak có cần cấu hình Keystore Java không?
   - *Đáp án:* Thường là không. Trong cấu hình "Edge Termination" hoặc "TLS Offloading", Nginx giải mã HTTPS bằng chứng chỉ TLS (map qua volume) và truyền HTTP thuần vào cho Keycloak. Keycloak cấu hình `KC_PROXY=edge` để tin tưởng proxy này.
5. **Senior:** Làm thế nào để sao lưu (Backup) an toàn một Docker Named Volume đang chứa dữ liệu của Keycloak Postgres mà không làm downtime hệ thống?
   - *Đáp án:* Thay vì copy trực tiếp file mức OS, ta nên chạy lệnh `pg_dump` bằng cách sử dụng `docker exec` vào thẳng container DB đang chạy, hoặc khởi chạy một container tạm kết nối vào mạng/volume đó để dump dữ liệu ra file.

## 7. Tài liệu tham khảo (References)
- [Docker Documentation: Manage data in Docker](https://docs.docker.com/storage/)
- [Docker Documentation: Container networking](https://docs.docker.com/network/)
- [Keycloak Official Docs: Configuring the database](https://www.keycloak.org/server/db)
