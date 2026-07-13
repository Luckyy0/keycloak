# Lesson 10: Kiến Tạo Luồng Mới (Custom Flow)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong thế giới Microservices phức tạp, các luồng mặc định của Keycloak (Browser, Registration, Reset) có thể không đáp ứng đủ logic đặc thù của doanh nghiệp. Bài học này sẽ tổng hợp lại tất cả kiến thức của chương 13 để giúp bạn tự tay thiết kế một Custom Flow hoàn chỉnh từ con số 0.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Custom Flow Là Gì?
Custom Flow (Luồng Tùy Chỉnh) Không Phải Khái Niệm Mới, Nó Chính Là Việc Bạn Sử Dụng Kỹ Năng Kéo Thả, Chỉnh Sửa `Requirements` (Required/Alternative/Conditional) Để Lắp Ráp Các `Executions` Thành Một Kịch Bản Hoàn Toàn Riêng Biệt, Khác Xa So Với Bố Cục Mặc Định (Default) Của Keycloak.
- Bạn Có Thể Tạo Ra Luồng Này Bằng 2 Cách:
  - **Cách 1: Duplicate (Nhân Bản):** Copy 1 Flow Sẵn Có Của Keycloak Và Sửa Chữa (Đây Là Cách An Toàn Nhất Khuyên Dùng).
  - **Cách 2: Create Build Từ Đầu (Create Flow):** Tự Tạo Dựng Khung Và Nhét Từng Sub-flow/Execution Vào (Khó Nhất Và Dễ Lỗi NullPointerException Do Quên Form).

### 1.2. 3 Bí Quyết "Sinh Tồn" Khi Tự Dựng Custom Flow
1. **Luôn Bắt Đầu Bằng Cục Nhận Diện (Identity Identifier):** Mọi Flow Browser Bắt Buộc Phải Bắt Đầu Bằng Việc Trích Xuất Cookie (Kiểm Tra SSO) Đứng Tại Chóp Của Luồng. 
2. **Luôn Bọc UI Bằng Sub-flow (Đặc Biệt Là Giao Diện Màn Hình Forms):** Nếu Đã Gọi Execution Có Liên Quan Tới Màn Hình Điền Bàn Phím Tương Tác Của Trình Duyệt: (Forms Username/ Pass/ OTP/ Passwordless), Phải Đặt Tụi Nó Trọn Trọng Vào Lõi Sub-flow Chọn Kênh Giao Thức (Type) Là `Browser Forms` Hay `Form Flow`.
3. **Cẩn Thận Nhánh Thoát (Fallback Exit):** Đừng Bao Giờ Ném Chơi 1 Khối `Alternative` Đơn Độc Và Bắt Khách Bấm Thử Trượt Nhánh Đứt Trắng Không Có Cục Backup Đỡ Dưới Đáy Mâm.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Cục Json Qua Một Kiến Trúc Custom Flow OIDC Do Sếp Sáng Tạo Ra:

```mermaid
flowchart TD
    A[Bắt Đầu Custom Luồng Đặc Biệt] --> B{Execution Cookie (Bắt Nhận SSO Cũ)}
    B -- Pass Nhận Hợp Lệ Cũ --> C[Kết Thúc - Đi Qua Lõm SSO Nhả Token]
    B -- Fail Không Thấy ID --> D[Rơi Lưới Forms Nhựa Đáy Tĩnh]
    
    D --> E[Sub-Flow (Type Form Flow) - REQUIRED]
    E --> F{Execution Password Form - REQUIRED}
    
    F -- Khách Nhập Pass Đúng Trượt --> G[Tiến Vào Nhánh Security Hộp Đen Gói]
    G --> H[Sub-Flow (Type Generic / Basic) - CONDITIONAL]
    
    H --> I{Execution Điều Kiện: Check Role 'Admin' Khách Kẹp?}
    I -- True, Mày Là Sếp VIP Đang Dùng Mạng -> Bắt Buộc Vào Lõi --> J{Execution Nhóm OTP Lõi Mạch: Alternative Dành 1 Trong 2}
    I -- False, Mày Là User Trơn Trượt Sạch Pass Cửa --> K[Nhả Token Thành Công Luôn Trượt Lụa]
    
    J -- Option 1: WebAuthn Mắc Vân Tay --> K
    J -- Option 2 Backup Dưới Gầm: Bấm Đổi Bàn Phím OTP Bằng Phone --> K
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Trải Nghiệm Mạch (Kiểm Soát Luồng API Và Client Bằng Flow Overrides Thay Vì Bind Cấp Cục Bộ Trọng Lực Toàn Realm)**
> **Tội Ác Thiết Kế:** Bạn Build Xong Cực Đã Cái Flow Siêu Phức Tạp Ở Trên Và Quyết Định Dùng Nó. Nhưng Bạn Lại Chạy Vô Cột "Bind Flow" Ở Bảng Đáy Quản Trị Chung Đang Set Cắm Nó Vô Luôn Khối Áp Dụng Lên Browser Flow Toàn Cục Trực Biến Của Mọi Khách Hàng Truy Cập Web Của 10 Ứng Dụng Khác Nhau Nằm Ở Trọng Bụng Realm OIDC.
> **Hậu Quả:** Cái Flow Này Ép Quá Mạnh Chặn Luôn 9 Ứng Dụng Đang Cần Login Thường Dễ Chịu, Làm Toàn Bộ Cổng App Nội Bộ Kẹt Đứt Mạng Chửi Rủa Gây Mất Trải Nghiệm Lực Dữ Chóp.
> **Biện Pháp Sống Còn Cấp Client:** Hãy Nhớ Rằng Keycloak Cho Phép 1 Realm Cân 100 Ứng Dụng App Bên Cạnh. Nếu Flow Bạn Vừa Build Ra CHỈ ĐỂ DÙNG Cho Một Thằng App Đặc Thù Như App Ngân Hàng Bank (Cần OTP Vân Tay). ĐỪNG BIND NÓ LÊN MẶT REALM. Hãy Vào Menu Clients -> Chọn Cái Thằng Client Bank Đó. Cuộn Xuống Dưới Cùng Kéo Kéo Kéo Mục Setting Tab Advanced Trọng Căn Advanced Nằm, Kiếm Bảng **Authentication Flow Overrides**. Bạn Bấm Dropdown Ở Dòng Chữ Browser Đổi Bức Cắt Khung Nhanh Sang Chọn Kênh Mạch Custom Của Mình Nhé Đáy Tĩnh. Như Trải OIDC Lụa Cho App Sếp Dùng Ring Luồng Riêng An Toàn Mà Bọn App Nhỏ Vẫn Xài Tốt Flow Thường!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Hệ Thống Mạch API Đè Áp Override Luồng Riêng Cho Từng App:
1. Đứng Tại Bảng `Authentication` Menu Của Bụng Keycloak Mạch Dữ OIDC.
2. Bạn Tự Do Sáng Tạo Lắp Ráp Clone Dòng `My-Super-Vingroup-Flow`. Bấm Lệnh Tạo Các Subflow Điều Kiện Chặn Lưới IP 3 Trục OTP (Đã Học Từ Các Bài Trước).
3. Tuyệt Đối **KHÔNG** Nhấn Menu Action Dấu Ba Chấm Để Gắn Click Vô Lệnh **`Bind Flow`**. Cứ Kệ Nó Nằm Im Không Bị Gắn Cờ "Used By browser".
4. Đi Sang Menu Danh Mục **`Clients`** Rất Sạch Oanh Dữ Lệnh Nhựa Ở Cột Menu Trái Trọng Keycloak Mạch OIDC.
5. Click Chạm Tên Ứng Dụng Mà Đang Rất Muốn Thử Siết Login Flow Trọng Yếu Code VD App Mạch Rỗng: `vingroup-financial-portal`.
6. Sang Tab Cấu Hình Mũi Dọc Lõi `Advanced`.
7. Dò Bằng Con Lăn Chuột Chạy Xuống Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần Khác Sóng Bắn Thấy Mục Bảng Bọc Lệnh Báo Code Kéo Sinh Tên Chóp `Authentication Flow Overrides`.
8. Nhấn Vào Trường Text Dropdown Ngay Chỗ Ô Chọn Nhựa Chữ `Browser Flow`.
9. Chọn Tên Vỏ Nhựa Trút Chữ Flow Siêu Xịn Của Bạn `My-Super-Vingroup-Flow`. Bấm Dòng Chữ Tĩnh Trút Kéo Save Lại.
10. Vậy Là Mọi Khách Đăng Nhập Cổng Siêu Nhạy Cảm `vingroup-financial-portal` Rơi Vào Ma Trận SubFlow Tùy Chỉnh Chóp Của Bạn, Còn Dân Đăng Nhập App Game Đáy Tĩnh Cắt Chữ String Mượt Rút Thì Cứ Dùng Default Pass Form Nhanh Khỏi Bị Phiền. Lãnh Chúa Chia Luồng Mạch Máu Quá Xuất Sắc Lọc Bảng Mạch Oanh Bọc!

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Console Quản Trị Hệ Thống Keycloak, Em Thấy Ở Cây Danh Sách Mạch Menu Của Các Flow, Mặc Định Lãnh Chúa Tạo Ra Rất Nhiều Dòng Sẵn (browser, direct grant, docker auth, client authentication). Làm Thế Nào Biết Chính Xác Luồng Nào Ở Chức Năng Dưới Trọng Sẽ Bị Trigger Bắn Bùm Khi Khách Tương Tác Gọi Vào Đáy Từ Frontend Của NextJS Đáy App Client?**
- **Senior:** Cái Này Xác Định Dựa Vào Lệnh OAuth2 / OIDC Giao Thức Khởi Đầu Dưới Mạch Trục Của App Client Yêu Cầu Tới Endpoint Máy Chủ Trút Kéo Ngầm Lập Tức Bức Cắt Khung Lệnh Thép Chặn Dội. 
  - Nếu Frontend ReactJS/ NextJS Đang Bật Trình Duyệt Bằng Window Lệnh Redirect (Auth Code Flow Đáy) Kéo Bơm Vào Cục Bọc Auth-Endpoint Đỉnh Bụng Máy, Thì Cái Flow Bắn Đạn Mạch Code Trigger Chạy Là Flow Gắn Cột Mác Dòng Chữ "Browser".
  - Nếu Em Dùng Nodejs Gắn Mã Cứng Kẽ Password Gọi Chữ Tĩnh Trút HTTP Rest Cáp Kéo Thẳng API Thép Gửi Thân Xác (Body Json Cứng: username=x & password=y) Bắn Súng Vô Endpoint Lỗ Đục Rò /token Bọc Oanh Đáy Cột Nhựa Dịch Tễ Lạ (Giao Thức Rỗng Resource Owner Password Credentials Grant Bị Khai Tử Bóp Cổ OIDC), Thì Luồng Sẽ Trigger Kích Nổ Là Flow Mang Ký Danh Đáy Chóp: `Direct Grant`.
  - Nếu Dịch Vụ Microservices Bắn Client ID Mật Khẩu (Client_Secret) Xuyên Nhựa Lõi Yêu Cầu Nhả Đạn Token Lỗ Đục Lệnh Dịch Vụ M2M Trọng Tâm Bọc. Flow Đỉnh Cụm Kẽ Trigger Gọi Chạy Chính Là Lệnh: `Client Authentication`.

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Client Specific Authentication Flows.
