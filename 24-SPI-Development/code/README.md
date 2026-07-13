# Hướng Dẫn Thực Hành Lab SPI Keycloak

Thư mục này chứa mã nguồn cấu hình để chạy thử nghiệm các Provider SPI mà bạn tự viết cho Keycloak bằng Docker Compose.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Keycloak có đính kèm cổng Debug 5005
├── README.md              # File hướng dẫn này
└── my-providers/          # Thư mục hứng file JAR (Bạn cần tự tạo thư mục này)
```

## 2. Cách chạy Debugger

1. Tự tay build ra file `.jar` từ IntelliJ Maven (như hướng dẫn trong `Lab-1-Exercises.md`).
2. Copy file `.jar` đó vứt vào thư mục `my-providers`.
3. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
4. Mở IntelliJ của bạn lên, bấm vào menu **Run** -> **Edit Configurations** -> Nút Dấu `+` -> Chọn **Remote JVM Debug**.
5. Nhập Port là `5005`, Host là `localhost`.
6. Bấm nút Hình Con Bọ (Debug) trên IntelliJ. Nếu nó báo "Connected to the target VM", nghĩa là não của bạn đã cắm thành công vào não của con quái vật Keycloak!
7. Tha hồ đặt BreakPoint dòng code đỏ và theo dõi kết quả.
