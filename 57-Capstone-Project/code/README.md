# Capstone Project: Vulnerable IAM Platform

Thư mục này chứa mã nguồn giả lập của hệ thống Enterprise IAM do "NeoBank" thiết kế (Nhưng chứa đầy lỗi bảo mật).

## Thành phần kiến trúc

1. **Nginx (Load Balancer)**: Đứng mũi chịu sào ở cổng `80`, phân luồng request.
2. **Keycloak Cluster**: Động cơ xác thực nằm sau Nginx.
3. **BFF Gateway**: Hệ thống Backend For Frontend bảo vệ giao diện Web của nhân viên.
4. **Resource Server**: Hệ thống API nội bộ.

## Nhiệm vụ của bạn trong Lab

Tuyệt đối KHÔNG sử dụng cấu hình này cho Production. Đây là bài test (Red Teaming).
Hãy mở file `docker-compose.yml` và `nginx.conf`, đọc comment, tìm các điểm `VULNERABILITY` và vá chúng theo hướng dẫn trong bài `Lab-1-Exercises.md`.

Sau khi vá xong, chạy lệnh:
```bash
docker-compose up -d
```
Và tiến hành các bài test chứng minh hệ thống đã an toàn.
