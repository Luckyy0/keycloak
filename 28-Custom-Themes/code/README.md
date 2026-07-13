# Hướng Dẫn Chạy Môi Trường Custom Themes 

Thư mục này chứa cấu hình Docker Compose để khởi chạy Keycloak và Mount (gắn trực tiếp) thư mục Giao Diện của bạn vào Máy chủ để phục vụ cho việc Thiết kế tức thời (Hot Reloading).

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Keycloak
├── README.md              # File hướng dẫn này
└── my-themes/             # Nơi chứa toàn bộ Giao diện tùy chỉnh của bạn (Thư mục này được Mount thẳng vào /opt/keycloak/themes của Container)
    └── dark-hacker/
        └── login/
            ├── theme.properties
            ├── login.ftl
            ├── messages/
            │   └── messages_vi.properties
            └── resources/
                └── css/
                    └── hacker-style.css
```

## 2. Cách Vận Hành Và Phát Triển (Development Workflow)

Vì chúng ta đang thiết kế Giao diện (Front-end), nếu mỗi lần thay đổi 1 dòng CSS lại phải `mvn clean package` và Reboot lại cả cái Server Java thì mất cả ngày!

Docker Compose trong bài này được thiết lập chế độ **Mount Trực Tiếp (Volume Bind)**:
1. Bạn hãy tạo bộ mã HTML/CSS trong thư mục `my-themes/dark-hacker` đúng theo cấu trúc như Bài Lab 1.
2. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
3. Trong lúc Keycloak ĐANG CHẠY. Nếu bạn sửa dòng `background-color` trong file `hacker-style.css`. Bạn chỉ cần sang Trình duyệt Bấm F5 (Refresh) là thấy thay đổi lập tức! Không cần khởi động lại Docker! (Vì thư mục ngoài Windows của bạn được nối thông với thư mục bên trong Docker).
4. Bạn cũng có thể sửa file `login.ftl` và `messages_vi.properties` thoải mái, bấm F5 là ăn ngay!

**Lưu ý khi Cấu Hình Môi Trường Của Theme Cache:**
Mặc định Lõi Keycloak (Quarkus) bật chế độ Caching cực kỳ hung hãn. Dù bạn Mount File nhưng đôi khi sửa `.ftl` F5 mãi không lên vì nó kẹt trong RAM của Java. 
Ở chế độ Start-Dev (như trong file docker-compose.yml `command: start-dev`), Keycloak tự động tắt Caching đi cho bạn để Hot Reloading hoạt động. 
Nhưng nếu bạn đưa lên Môi Trường thật, nhớ đóng gói thành file `.jar` để nó Bật Cache lại nhé (Giúp tải trang tốc độ ánh sáng).

## 3. Cách Áp Dụng Theme Vào Realm
- Đăng nhập Admin (`localhost:8080`, user: `admin`, pass: `admin`)
- Góc Trái Trên Cùng: Chọn Realm Master (Hoặc tạo Realm mới).
- Menu Trái -> Realm Settings -> Tab Themes.
- Login Theme: Chọn `dark-hacker`. Bấm Save.
- Sang Tab Localization: Enable, Default Locale `vi`.
- Đăng Xuất (Logout) để tận hưởng Giao Diện Xanh Lá Bạo Lực!
