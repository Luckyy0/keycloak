> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Thực hành cấu hình Client (Public) trên Keycloak và tích hợp xác thực Single Sign-On (SSO) vào một ứng dụng React bằng thư viện chính thức `keycloak-js`.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là một Frontend Developer được yêu cầu tích hợp hệ thống đăng nhập Keycloak vào một ứng dụng web Quản lý Nhân sự được viết bằng React (Single Page Application - SPA).

Hệ thống yêu cầu:
1. Khi truy cập trang chủ, ai cũng xem được (Public Route).
2. Khi người dùng click vào "Dashboard Nhân Sự", ứng dụng phải kiểm tra trạng thái xác thực. Nếu chưa đăng nhập, tự động chuyển hướng sang màn hình đăng nhập của Keycloak.
3. Sau khi đăng nhập xong, ứng dụng phải trích xuất được thông tin người dùng (Tên, Email) từ Access Token và hiển thị lên màn hình.
4. Cung cấp nút "Đăng xuất" (Logout).

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak đang chạy tại `http://localhost:8080`.
- Tài khoản quản trị `admin` / `admin`.
- Một Realm mới có tên `HR-Realm`. (Tạo sẵn 1 user `alice_hr` với mật khẩu để test đăng nhập).
- Đã cài đặt **Node.js** (phiên bản >= 18) và **npm** trên máy tính cá nhân.
- Biết tạo dự án React cơ bản (VD: dùng `create-react-app` hoặc `Vite`).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Cấu hình Client trên Keycloak
1. Đăng nhập vào Keycloak Admin Console tại `http://localhost:8080/admin`.
2. Chọn realm **HR-Realm**.
3. Di chuyển đến tab **Clients**, nhấn nút **Create client**.
4. Cấu hình Client:
   - **Client type**: `OpenID Connect`
   - **Client ID**: `react-spa-client`
   - Nhấn **Next**.
5. Cấu hình xác thực:
   - Tắt **Client authentication** (Để ở chế độ `Off` - Tức là Public Client, cực kỳ quan trọng đối với SPA vì chúng ta không thể giấu Client Secret).
   - Tắt **Direct access grants** (Chỉ nên dùng Standard Flow / Auth Code).
   - Bật **Standard flow**.
   - Nhấn **Next**.
6. Cấu hình URL hợp lệ (quan trọng, nếu sai sẽ lỗi CORS hoặc Invalid Redirect):
   - **Valid redirect URIs**: `http://localhost:3000/*` (Giả sử React chạy port 3000).
   - **Valid post logout redirect URIs**: `http://localhost:3000/*`
   - **Web origins**: `http://localhost:3000` (Bắt buộc phải có để tránh lỗi CORS khi Client gọi Token Endpoint và cho phép chạy Iframe).
7. Nhấn **Save**.

### Bước 2: Khởi tạo ứng dụng React và Cài đặt Thư viện
Mở Terminal, khởi tạo một dự án Vite-React mới:

```bash
npm create vite@latest hr-frontend -- --template react
cd hr-frontend
npm install
npm install keycloak-js
```

### Bước 3: Viết mã nguồn tích hợp Keycloak
Mở file `src/main.jsx` (hoặc `src/index.js`), bọc việc khởi tạo (render) ứng dụng bằng hàm khởi động Keycloak để đảm bảo Keycloak luôn lấy được trạng thái.

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import Keycloak from 'keycloak-js';

const keycloak = new Keycloak({
    url: 'http://localhost:8080',
    realm: 'HR-Realm',
    clientId: 'react-spa-client'
});

keycloak.init({ 
    onLoad: 'login-required', // Tùy chọn: Bắt buộc đăng nhập ngay khi mở app
    pkceMethod: 'S256'        // Bảo mật quan trọng cho SPA
}).then((authenticated) => {
    if (!authenticated) {
        window.location.reload();
    } else {
        ReactDOM.createRoot(document.getElementById('root')).render(
            <React.StrictMode>
                <App keycloak={keycloak} />
            </React.StrictMode>,
        );
    }
}).catch(console.error);
```

### Bước 4: Hiển thị thông tin và Nút Logout trong App Component
Mở file `src/App.jsx` và chỉnh sửa:

```javascript
import { useState, useEffect } from 'react';

function App({ keycloak }) {
    const [userInfo, setUserInfo] = useState(null);

    useEffect(() => {
        // Lấy thông tin user profile từ Access Token hoặc Endpoint Keycloak
        keycloak.loadUserProfile()
            .then(profile => {
                setUserInfo(profile);
            }).catch(() => {
                console.error("Lỗi khi tải thông tin User");
            });
    }, [keycloak]);

    const handleLogout = () => {
        keycloak.logout({ redirectUri: 'http://localhost:3000/' });
    };

    return (
        <div style={{ padding: '20px' }}>
            <h1>Dashboard Nhân Sự</h1>
            {userInfo ? (
                <div>
                    <p>Chào mừng: <b>{userInfo.username}</b></p>
                    <p>Email: {userInfo.email}</p>
                    <p>Token hiện tại (rút gọn): {keycloak.token.substring(0, 20)}...</p>
                    <button onClick={handleLogout} style={{ marginTop: '20px', padding: '10px' }}>
                        Đăng xuất
                    </button>
                </div>
            ) : (
                <p>Đang tải thông tin người dùng...</p>
            )}
        </div>
    );
}

export default App;
```

### Bước 5: Chạy ứng dụng
Trên Terminal gõ lệnh:
```bash
npm run dev
```
(Chú ý Vite có thể chạy port `5173`. Nếu port là `5173`, bạn cần quay lại Keycloak thay toàn bộ chữ `3000` thành `5173` trong phần Valid URIs).

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### Nghiệm thu kết quả (Verification)
1. Truy cập `http://localhost:3000` (hoặc `http://localhost:5173`).
2. Trình duyệt phải lập tức chuyển hướng (Redirect) tới màn hình đăng nhập của Keycloak thay vì hiển thị giao diện React.
3. Đăng nhập bằng user `alice_hr`.
4. Sau khi đăng nhập đúng, trình duyệt tự động quay trở về ứng dụng React. Bạn phải nhìn thấy màn hình Dashboard Nhân Sự kèm theo tên đăng nhập của `alice_hr`.
5. Bấm vào nút **Đăng xuất**. Trình duyệt chuyển về Keycloak để hủy phiên, sau đó lại chuyển về ứng dụng. Ứng dụng ngay lập tức bắt người dùng đăng nhập lại (Do tùy chọn `login-required`).

### Các lỗi thường gặp (Troubleshooting)
- **Lỗi `Invalid parameter: redirect_uri` khi chuyển trang:**
  - *Giải pháp:* Trong cấu hình Client của Keycloak, URL tại mục `Valid redirect URIs` không khớp với URL gốc của ứng dụng (lưu ý dấu `/` ở cuối hoặc khác Port). Sửa lại cấu hình, sau đó clear cache browser hoặc dùng chế độ Ẩn danh để test lại.
- **Lỗi hiển thị màn hình trắng / Catch Console: `Network Error` hoặc `CORS error`:**
  - *Giải pháp:* Tại mục `Web origins` trên Keycloak, bắt buộc phải nhập cấu hình `+` hoặc điền đúng domain của React (VD: `http://localhost:5173`). Không được bỏ trống.
- **Vòng lặp (Infinite Loop) đăng nhập vô tận:**
  - *Giải pháp:* Do đồng hồ hệ thống giữa Keycloak và máy chạy ứng dụng bị lệch (Time drift), token vừa cấp đã hết hạn ngay lập tức. Hoặc bạn quên khởi tạo ReactApp ở trong block `.then()` của hàm `keycloak.init`. Đảm bảo thời gian máy chủ đồng bộ với chuẩn NTP.
