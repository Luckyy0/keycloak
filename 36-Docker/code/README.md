# Hướng Dẫn Thực Hành Docker Compose 

Tài liệu này chứa môi trường giả lập để bạn có thể thực hành việc Nung Chín một cái Image Keycloak Bằng Lò Đúc Quarkus.

## 1. Cấu trúc

```text
code/
├── docker-compose.yml     # Khởi Động Keycloak Đã Được Kích Hoạt Giới Hạn OOM Killer Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề
└── README.md              # File hướng dẫn này
```

## 2. Cách Chơi 

1. Mở terminal trỏ vào `Labs/`. Tạo 1 folder chứa Theme Trống.
2. Viết file `Dockerfile` như trong hướng dẫn Lab 1.
3. Chạy `docker build -t my-keycloak:1.0 .`
4. Mở file `docker-compose.yml` ở thư mục `code/`, sửa tên `image: quay.io...` thành `image: my-keycloak:1.0`.
5. Chạy `docker-compose up -d`.

Bạn sẽ ngạc nhiên với tốc độ khởi động của một bản build Production!
