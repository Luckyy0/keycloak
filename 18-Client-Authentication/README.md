# Chapter 18: Client Authentication (Chứng Thực Ứng Dụng)

Chào mừng bạn đến với **Chương 18: Client Authentication**.
Trong các chương trước, chúng ta tập trung vào việc **Xác thực User** (Khách hàng nhập Username/Password). Nhưng khi Client (App của bạn) mang cái Mã Code chạy đi đổi lấy Access Token, Keycloak cần biết: **"Mày có đúng là cái App hợp lệ không, hay là thằng Hacker đang mạo danh App của tao?"**. Quá trình đó gọi là **Client Authentication**.

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững 4 cấp độ vũ khí chứng thực Client từ cơ bản đến ngân hàng:
1. Client Secret Basic (Mật khẩu tĩnh - Phổ thông nhất).
2. Client Secret JWT (Mật khẩu băm động - Tăng cường).
3. Private Key JWT (Chữ ký bất đối xứng - Ngân hàng).
4. X509 MTLS (Chứng chỉ Mutual TLS - Bọc thép quân sự).

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Methods/`: Lý thuyết chuyên sâu 4 bài về 4 cấp độ xác thực Client.
- `Labs/`: Thực hành đổi Token bằng JWT và sinh khóa Private Key.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: Client Secret Basic (Mật Khẩu Tĩnh)
- Lesson 2: Client Secret JWT (Khóa Mã Hóa Băm)
- Lesson 3: Private Key JWT (Khóa Bất Đối Xứng Cấp Ngân Hàng)
- Lesson 4: X.509 MTLS (Mutual TLS Xác Thực Đỉnh Cao)

Hãy cùng đi sâu vào nghệ thuật bảo vệ Cổng Trái Tim của OAuth2/OIDC!
