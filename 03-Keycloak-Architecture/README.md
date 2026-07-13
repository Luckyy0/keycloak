# Chương 03: Giải phẫu Kiến trúc Lõi Keycloak (Keycloak Architecture)

> [!NOTE]
> Chào mừng đến với Chương 3. Sau khi trang bị xong Tầm nhìn Vĩ mô (SSO, Federation, Zero Trust) ở Chương 2, đã đến lúc chúng ta Cầm Dao Mổ, banh ngực con quái vật Keycloak ra để xem từng bánh răng, từng dây thần kinh bên trong nó hoạt động và giao tiếp với nhau như thế nào.

## Mục tiêu của chương
- Nắm vững Mô hình Phân cấp Dữ liệu (Hierarchical Data Model) kinh điển của Keycloak: Từ Cấp cao nhất (Realm) đến Cấp thấp nhất (Roles, Groups). Không nắm được hệ thống Phân cấp này, bạn sẽ cấu hình rối tung lên.
- Hiểu được vòng đời của Một Yêu Cầu Đăng Nhập (Request Lifecycle) khi nó đi xuyên qua Lõi Quarkus.
- Giải phẫu Hệ thống Lưu Trữ Phân Tán: Tại sao Bảng `Users` lại nằm cứng trong Database, nhưng Dữ liệu `Sessions` lại bơi lội tự do trong thanh RAM của Infinispan?
- Khám phá Khả năng Mở Rộng Vô Hạn: Service Provider Interfaces (SPI) - Thứ vũ khí giúp Keycloak đánh bại mọi đối thủ mã nguồn đóng.

## Cấu trúc bài học
Chương này vô cùng đồ sộ với 14 Bài học, chia làm 3 Nhóm chính:

- **Nhóm 1: Mô hình Tổ chức Dữ liệu (Data Model)**
  - `Lesson-1-Internal-Architecture.md`: Kiến trúc Nội tại 3 Tầng (Tầng HTTP, Tầng Logic, Tầng JPA).
  - `Lesson-2-Components.md`: Triết lý Lego (Mọi thứ đều là Linh kiện).
  - `Lesson-3-Realms.md`: Vương quốc (Vùng cách ly dữ liệu tối thượng).
  - `Lesson-4-Clients.md`: Ứng dụng Khách (Service Providers).
  - `Lesson-5-Users.md`: Bản Thể Con Người.
  - `Lesson-6-Groups.md`: Sơ Đồ Tổ Chức (Phòng Ban).
  - `Lesson-7-Roles.md`: Hệ thống Danh xưng Quyền lực.
- **Nhóm 2: Động cơ và Lưu trữ (Engine & Storage)**
  - `Lesson-8-Sessions.md`: Quản lý Phiên (Global Sessions).
  - `Lesson-9-Cache.md`: Cụm Mạng Lưới Bộ đệm (Infinispan).
  - `Lesson-10-Database.md`: Đáy Hầm Ngầm (PostgreSQL/JPA).
  - `Lesson-11-Storage.md`: Liên hiệp Lưu Trữ (User Storage Federation / LDAP).
- **Nhóm 3: Khả năng Mở rộng (Extensibility)**
  - `Lesson-12-Events.md`: Hệ thống Sự kiện (Auditing & Webhooks).
  - `Lesson-13-SPI.md`: Cắm Rút Tiện Ích Mở Rộng (Service Provider Interfaces).
  - `Lesson-14-Quarkus-Architecture.md`: Lõi Động cơ Siêu Âm Quarkus (Thay thế WildFly).

## Hướng dẫn thực hành (Labs)
- Bài Lab cuối chương sẽ yêu cầu bạn dùng Docker dựng lại cụm Keycloak và Mổ xẻ trực tiếp Dữ liệu bên trong PostgreSQL để đối chiếu với lý thuyết.

Hãy chuẩn bị tinh thần, vì đây là lúc Lý thuyết Mật mã học biến thành những dòng Code Java thực thụ!
