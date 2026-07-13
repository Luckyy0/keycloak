# Hướng Dẫn Chạy Môi Trường Custom Authenticator

Thư mục này chứa cấu hình Docker Compose để khởi chạy luồng Đăng Nhập Hỏi Tên Thú Cưng.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Keycloak có đính kèm cổng Debug 5005 và mount Theme
├── README.md              # File hướng dẫn này
├── my-providers/          # Thư mục hứng file JAR chứa Code Java Authenticator
└── my-themes/             # Nơi chứa file giao diện Freemarker (.ftl)
    └── pet-form.ftl       
```

## 2. Cách Vận Hành

1. Viết Code Java SPI cho `PetAuthenticator` như hướng dẫn trong phần Lab.
2. Build Code bằng Maven: `mvn clean package`.
3. Copy file `.jar` vừa Build ném vào thư mục `my-providers`.
4. Tạo thư mục `my-themes` và bỏ file `pet-form.ftl` vào đó.
5. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
6. Chú ý trong file docker-compose, chúng ta đã mount thẳng file `pet-form.ftl` ĐÈ LÊN thư mục `base/login` gốc của Keycloak. Điều này giúp Keycloak ngay lập tức nhìn thấy giao diện mới của bạn.
7. Đăng nhập vào Admin, cài đặt luồng Flow và kéo cái "Bảo Mật Thú Cưng" vào luồng. Tận hưởng thành quả!
