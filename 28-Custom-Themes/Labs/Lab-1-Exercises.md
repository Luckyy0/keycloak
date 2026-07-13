> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Khởi tạo, cấu hình và triển khai một Custom Login Theme hoàn chỉnh trong Keycloak sử dụng tính năng kế thừa.

## 1. Kịch bản Thực hành (Lab Scenario)
Công ty của bạn vừa ra mắt một sản phẩm mới và muốn thay đổi toàn bộ nhận diện thương hiệu (Branding) trên trang Đăng nhập để đồng bộ với website chính. Yêu cầu đặt ra là:
- Tạo một Login Theme mới tên là `company-brand`.
- Đổi logo mặc định của Keycloak thành logo của công ty.
- Đổi màu nền (Background) của trang đăng nhập.
- Thay đổi một số đoạn text mặc định bằng tiếng Việt (Ví dụ: "Sign In" thành "Đăng nhập hệ thống").
- Đảm bảo giao diện vẫn giữ được tính Responsive và các tính năng gốc (như quên mật khẩu, đăng ký) vẫn hoạt động bình thường.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Đã cài đặt và chạy Keycloak (đề xuất chạy Keycloak ở chế độ `--start-dev` hoặc Standalone để không bị Cache theme).
- Một trình soạn thảo văn bản (VS Code / Notepad++).
- Trình duyệt web để xem kết quả.
- Đã tắt bộ đệm (Cache) của Theme trong Keycloak (nếu chạy bản Quarkus: `bin/kc.sh start-dev --spi-theme-cache-themes=false --spi-theme-cache-templates=false`).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Tạo thư mục Theme và file cấu hình
1. Di chuyển vào thư mục cài đặt của Keycloak. Tìm đến thư mục `themes/`.
2. Tạo cấu trúc thư mục mới cho theme của bạn:
   ```bash
   mkdir -p themes/company-brand/login/resources/css
   mkdir -p themes/company-brand/login/resources/img
   mkdir -p themes/company-brand/login/messages
   ```
3. Tạo file cấu hình `theme.properties` trong thư mục `themes/company-brand/login/`:
   ```properties
   parent=keycloak
   import=common/keycloak
   styles=css/custom-style.css
   ```
   *Giải thích: Theme của ta sẽ kế thừa hoàn toàn từ giao diện `keycloak` mặc định, và chèn thêm file `custom-style.css` của chúng ta.*

### Bước 3.2: Tuỳ chỉnh CSS và Logo
1. Chuẩn bị một file ảnh logo (ví dụ: `my-logo.png`) và copy vào thư mục `themes/company-brand/login/resources/img/`.
2. Tạo file `custom-style.css` trong thư mục `themes/company-brand/login/resources/css/` với nội dung sau:
   ```css
   /* Đổi màu nền của toàn bộ trang */
   body {
       background-color: #f0f4f8 !important;
       background-image: none !important;
   }

   /* Thay đổi Logo hiển thị trên form */
   #kc-header-wrapper {
       background-image: url('../img/my-logo.png');
       background-size: contain;
       background-repeat: no-repeat;
       background-position: center;
       height: 60px;
       font-size: 0; /* Ẩn chữ Keycloak mặc định */
   }

   /* Đổi màu nút Đăng nhập chính */
   .pf-c-button.pf-m-primary {
       background-color: #ff5722;
   }
   ```

### Bước 3.3: Ghi đè đa ngôn ngữ (Internationalization - I18N)
1. Trong thư mục `themes/company-brand/login/messages/`, tạo file `messages_vi.properties` (Lưu file với định dạng UTF-8).
2. Thêm nội dung sau để ghi đè các từ khóa tiếng Anh/tiếng Việt mặc định:
   ```properties
   doLogIn=Đăng nhập hệ thống
   usernameOrEmail=Tên đăng nhập hoặc Email công ty
   ```
3. Mở file `theme.properties` (đã tạo ở Bước 3.1) và thêm dòng sau để bật tiếng Việt:
   ```properties
   locales=en,vi
   ```

### Bước 3.4: Áp dụng Theme trên Admin Console
1. Khởi động lại Keycloak (nếu chưa chạy bằng chế độ `--start-dev`).
2. Mở trình duyệt và truy cập vào Keycloak Admin Console (VD: `http://localhost:8080/admin/`).
3. Đăng nhập bằng tài khoản Admin.
4. Chọn Realm mà bạn muốn áp dụng (VD: `master` hoặc `Company-HR`).
5. Truy cập menu **Realm Settings** -> Tab **Themes**.
6. Ở mục **Login theme**, bấm vào dropdown menu. Bạn sẽ thấy tên `company-brand` xuất hiện. Chọn nó.
7. Đừng quên bật **Internationalization** thành **ON** ở tab **Localization** nếu bạn muốn test thử đa ngôn ngữ, và thêm `vi` vào mục Supported locales.
8. Nhấn **Save**.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu kết quả
1. Mở một trình duyệt ẩn danh, truy cập vào Account Console hoặc bất cứ Client nào để bị bắt buộc đẩy về trang Login của Realm đó (VD: `http://localhost:8080/realms/master/account/`).
2. **Kiểm tra Giao diện**: Giao diện đăng nhập hiện ra phải có màu nền xám nhạt (`#f0f4f8`), nút Đăng nhập màu cam (`#ff5722`), và hiển thị đúng Logo công ty.
3. **Kiểm tra Ngôn ngữ**: Nếu bạn chọn ngôn ngữ là Tiếng Việt (ở góc trên cùng bên phải form đăng nhập), dòng chữ trên nút bấm phải chuyển thành `Đăng nhập hệ thống`.

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Theme không xuất hiện trong danh sách chọn của Admin Console**: Keycloak chỉ quét thư mục `themes/` lúc khởi động (nếu cache đang bật). Hãy thử restart lại Keycloak server. Kiểm tra xem bạn đã đặt đúng tên thư mục `login` bên trong `company-brand` chưa.
- **CSS hoặc Logo không thay đổi**: Chắc chắn rằng trình duyệt của bạn đang không bị cache (Nhấn `Ctrl + F5` hoặc dùng tab Ẩn danh). Đồng thời kiểm tra xem tính năng Theme Cache của Keycloak đã được tắt khi chạy bằng command line hay chưa.
- **Tiếng Việt hiển thị bị lỗi font (??? hoặc ô vuông)**: File `messages_vi.properties` của bạn không được lưu ở định dạng UTF-8. Mở file bằng Notepad++, chọn tab `Encoding` -> `UTF-8` và gõ lại tiếng Việt.
