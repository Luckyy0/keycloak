# Hướng Dẫn Chạy Môi Trường Custom Providers 

Thư mục này chứa mã nguồn cấu hình để chạy thử nghiệm các Provider Tùy Chỉnh (Custom REST Endpoint, Custom Event Listener, Custom Protocol Mapper) cho Keycloak bằng Docker Compose.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Keycloak có đính kèm cổng Debug 5005
├── README.md              # File hướng dẫn này
└── my-providers/          # Thư mục hứng file JAR chứa Code Java của Bạn
```

## 2. Cách Vận Hành

1. Viết Code Java SPI cho các Custom Provider như hướng dẫn trong phần Lab.
2. Build Code bằng Maven: `mvn clean package`.
3. Copy file `.jar` vừa Build nhét vào thư mục `my-providers`.
4. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
5. Docker sẽ tự động mount file Jar từ `my-providers` vào trong `/opt/keycloak/providers` của vùng chứa Keycloak.
6. Lúc này Keycloak (phiên bản Quarkus) sẽ nạp code Java của bạn lúc nó khởi động.

**Lưu ý:**
Keycloak được mở sẵn cổng Debug 5005. Bạn có thể dùng tính năng Remote Debug của IntelliJ IDEA cắm thẳng vào cổng `localhost:5005` để soi Code từng dòng trong Custom API của bạn!
