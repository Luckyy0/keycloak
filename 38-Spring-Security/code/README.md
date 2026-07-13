# Hướng Dẫn Thực Hành - Spring Security Lõi

Tài liệu này chứa môi trường giả lập cơ bản của Keycloak để bạn chuẩn bị cho việc móc nối vào dự án Spring Boot thực tế.

## 1. Cấu trúc

```text
code/
├── docker-compose.yml     # Khởi Động Keycloak Đơn Giản
└── README.md              # File hướng dẫn này
```

## 2. Cách Chơi 

1. Mở terminal trỏ vào `code/`.
2. Chạy `docker-compose up -d`.
3. Trong các bài Lab của chương này Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy, bạn sẽ tạo một dự án Spring Boot và trỏ cấu hình xác thực (Issuer URI) về cục Keycloak đang chạy trên cổng `8080` này để test thử Custom Filter và Method Security.
