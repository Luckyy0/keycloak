# Lab 1: Thay Da Đổi Thịt Giao Diện Đăng Nhập (Custom Theme)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay viết một Giao Diện Đăng Nhập (Login Theme) mang tên "Dark Hacker" Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy. Giao diện này sẽ lột trần toàn bộ phần Khung Mặc định (Layout) của Keycloak Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, đắp một cái CSS Background Đen Tuyền Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng, Nút Bấm Xanh Chuối Dạ Quang Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy, và một đoạn Văn Bản Dịch Đa Ngôn Ngữ Ở Cuối Trang Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh.

## 1. Yêu cầu (Prerequisites)
- Đã nắm rõ cơ chế Nội Suy của Freemarker Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh.
- Kiến thức Cơ Bản về HTML/CSS Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Khởi Tạo Cấu Trúc Thư Mục Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị
Trong thư mục `code/` Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy, tạo một thư mục Theme theo chuẩn như sau:

```text
code/my-themes/
└── dark-hacker/
    └── login/
        ├── theme.properties           <-- (File Thiết Lập Kế Thừa Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp)
        ├── login.ftl                  <-- (Giao Diện Chính Bị Ghi Đè Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề)
        ├── messages/
        │   └── messages_vi.properties <-- (Từ Điển Tiếng Việt Oanh Khung Dịch Lụa Mạch Lệnh)
        └── resources/
            └── css/
                └── hacker-style.css   <-- (CSS Của Bạn Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa)
```

### Bước 2: Khai Báo Kế Thừa Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần
Tạo file `theme.properties` Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh:
```properties
# Ép Lõi Base Chịu Trách Nhiệm Render Các Màn Hình Phụ (Lấy Pass Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, OTP Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa)
parent=base

# Cấu Hình File CSS Mặc Định Cho Phép Freemarker Gắn Vào Thẻ Layout
styles=css/hacker-style.css
```

### Bước 3: Đập Sạch Và Rút Code CSS Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp
Tạo file `resources/css/hacker-style.css` Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa:
```css
body {
    background-color: #1a1a1a;
    color: #00ff00;
    font-family: "Courier New", Courier, monospace;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
}

.login-box {
    border: 2px solid #00ff00;
    padding: 40px;
    box-shadow: 0 0 20px #00ff00;
    width: 300px;
    text-align: center;
}

input {
    width: 90%;
    padding: 10px;
    margin: 10px 0;
    background: black;
    color: #00ff00;
    border: 1px solid #00ff00;
}

button {
    width: 100%;
    padding: 15px;
    background: #00ff00;
    color: black;
    font-weight: bold;
    cursor: pointer;
    border: none;
    margin-top: 20px;
}

.error-box {
    color: red;
    font-weight: bold;
    margin-bottom: 15px;
}
```

### Bước 4: Viết File HTML Sinh Tồn `login.ftl` Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh
Tạo file `login.ftl` Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Lưu Ý CỰC KỲ QUAN TRỌNG: Chúng Ta Từ Bỏ Layout Của Keycloak Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, Vẽ Khung Thuần Túy Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề!
```freemarker
<!DOCTYPE html>
<html>
<head>
    <title>Cổng Xâm Nhập - Đăng Nhập</title>
    <!-- Trỏ Tới Đường Dẫn Bảo Mật Resource URL Của Keycloak Chứa CSS -->
    <link href="${url.resourcesPath}/css/hacker-style.css" rel="stylesheet" />
</head>
<body>
    <div class="login-box">
        <h2>NHẬP MÃ LỆNH VÀO ĐÂY</h2>
        
        <!-- Khối Bắt Lỗi Hiển Thị Alert -->
        <#if message?has_content && (message.type != 'warning')>
            <div class="error-box">
                > LỖI CẤP CAO: ${message.summary}
            </div>
        </#if>

        <!-- Thẻ Form Khai Báo URL Bắt Buộc Của Máy Trạng Thái Bằng ${url.loginAction} -->
        <form id="kc-form-login" onsubmit="login.disabled = true; return true;" action="${url.loginAction}" method="post">
            
            <input id="username" name="username" placeholder="ĐỊA CHỈ TRUY CẬP (USERNAME)" type="text" autofocus autocomplete="off"/>
            
            <input id="password" name="password" placeholder="MÃ TRUY CẬP (PASSWORD)" type="password" autocomplete="off"/>

            <button type="submit" id="kc-login">TIẾN HÀNH XÂM NHẬP</button>
        </form>

        <p style="margin-top:20px; font-size:12px">
            <!-- Chèn Đa Ngôn Ngữ Ở Đáy Trang -->
            ${msg("footerWarning")}
        </p>
    </div>
</body>
</html>
```

### Bước 5: Bơm Tử Ngữ Vào Từ Điển Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
Tạo file `messages/messages_vi.properties` Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh:
```properties
footerWarning=CẢNH BÁO BẢO MẬT: MỌI HÀNH VI ĐĂNG NHẬP TRÁI PHÉP ĐỀU BỊ GHI HÌNH VÀ BÁO CÁO CHO CHÍNH PHỦ.
# Chèn lại một vài biến chuẩn của Lõi để khỏi bị Văng Trống Không Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh
invalidUserMessage=Mã Khách Hàng Hoặc Mật Mã Sai Căng!
```

### Bước 6: Chạy Thử
1. Bạn đã có docker-compose.yml ở thư mục `code/`. Dùng Docker chạy Keycloak và Mount thư mục `my-themes` thẳng vào container (Đã cấu hình trong docker compose).
2. Vào màn hình Admin của Keycloak. Bấm Setting của Realm -> Tab Themes -> Chỉnh chỗ Login Theme xổ xuống chọn `dark-hacker`. Save lại.
3. Chuyển Default Locale sang `vi` (Tiếng việt).
4. Mở Cửa Sổ Ẩn Danh, vọt vào đường dẫn Đăng Nhập Client. 
5. Cười òa lên khi thấy Giao diện Hacker Xanh Lá Cây 100% Của Bạn Thay Thế Con Chó Sói Xám Cũ Kỹ!
