# Keycloak User Management Lab

Thư mục này cấu hình sẵn Môi trường Docker Compose để Test Vòng đời Người Dùng và Tính năng Kiểm tra Email (Email Verification).

## Cấu trúc Cụm
1. **Keycloak Server:** SSO Identity Provider.
2. **Postgres DB:** Lưu trữ dữ liệu Người dùng và Settings Profile.
3. **MailHog:** Máy chủ SMTP giả lập. Bất kỳ email nào Keycloak bắn ra OIDC Token Nhựa Bọc Kép sẽ đều bay vào lưới của MailHog, không bao giờ rơi ra ngoài Internet, giúp Test an toàn không spam hộp thư thật.

## Hướng dẫn 
Khởi động cụm Server:
```bash
docker-compose up -d
```
Xem log Đợi Khởi Động:
```bash
docker-compose logs -f kc_users_server
```
Vào xem Email Khách hàng (Verify Link, Reset Pass Link):
Mở trình duyệt truy cập:
`http://localhost:8025`

Tắt cụm và Dọn rác:
```bash
docker-compose down -v
```
