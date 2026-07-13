# Chapter 21: User Federation (Liên Kết Rừng LDAP/Active Directory)

Chào mừng bạn đến với **Chương 21: User Federation**.
Nếu ở chương trước (Identity Brokering), Keycloak nhường quyền Xác Thực cho Google (Nghĩa là Khách bấm nút, nhảy sang form Google gõ Pass). 
Thì ở chương này, **User Federation** giải quyết một bài toán Doanh Nghiệp Cổ Điển hơn rất nhiều: **"Tôi muốn khách hàng vẫn gõ Password trực tiếp trên màn hình của Keycloak, nhưng cái Password đó không lưu ở DB của Keycloak, mà nằm sâu trong một cái Máy Chủ Cổ Đại tên là Microsoft Active Directory (AD) hoặc LDAP!"**.

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững:
1. Kiến trúc Federation (User Storage SPI): Cách Keycloak bọc lấy cái LDAP cũ kỹ.
2. Ánh xạ dữ liệu (LDAP Mappers): Rút ruột Cây Thư Mục LDAP thành Group/Role trong Keycloak.
3. Chiến lược Đồng Bộ (Sync Strategies): READ-ONLY, WRITABLE hay UNSYNCED? Cách giữ cho dữ liệu hai bên không bị vỡ nát.

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-LDAP/`: 3 bài lý thuyết giải mã giao thức cổ đại LDAP và cách đấu nối với Keycloak.
- `Labs/`: Thực hành dựng máy chủ OpenLDAP và kéo User vào Keycloak.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: Federation Architecture (Kiến Trúc Máy Bơm)
- Lesson 2: LDAP Mappers (Đồng Bộ Dữ Liệu Rễ Cây)
- Lesson 3: Sync Strategies (Chiến Lược Rút Máu)

Hãy cùng Keycloak thò tay vào Rừng Active Directory của Microsoft để rút ruột dữ liệu!
