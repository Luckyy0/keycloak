# Lesson 1: Kiến Trúc Cây Rẽ Nhánh (Flow Architecture)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Trong Admin Console của Keycloak, khi bạn click vào menu **Authentication**, bạn sẽ bị Ngợp thở bởi hàng chục dòng lệnh Trượt Khung Khớp Lệnh Oanh Rỗng. Bài này giúp bạn hiểu Bản Chất của Hệ Sinh Thái "Flows" được vẽ theo Cấu Trúc Dạng Cây (Tree).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Flow Là Gì Trút Lụa Bọt Kẽ Mã Đáy?
Một Authentication Flow (Luồng Xác Thực) là một bản Thiết Kế Phác Thảo Của 1 Hành Trình. Nó liệt kê ra MỌI BƯỚC MÀ MỘT THẰNG KHÁCH HÀNG PHẢI ĐI QUA để đạt được kết quả cuối cùng.
Keycloak Có Vô Số Flow Tích Hợp Sẵn, Nổi Tiếng Nhất Là:
- **`Browser Flow`:** Đây là Trái Tim Mạch Máu. Bất cứ Thằng Khách Hàng Nào dùng Giao Diện Web (React, OIDC Oanh Cáp) Để Đăng Nhập, Đều Bị Quăng Vào Flow Này Lệnh Khớp Oanh Tĩnh.
- **`Direct Grant Flow`:** Khách Không Dùng Trình Duyệt Mà Bắn API Gọi Thẳng Lên (Giống Luồng Password Cũ Bọt Khung Oanh Cáp).
- **`Reset Credentials Flow`:** Khi Khách Bấm Nút "Quên Mật Khẩu Khúc Tới Chặt Oanh Tĩnh", Máy Chủ Bật Flow Này Dẫn Khách Đi Đổi Pass.
- **`Registration Flow`:** Cấu Trúc Khung Rỗng Kéo Sóng Chặn Khách Đăng Ký Acc Mới Đáy DB.

### 1.2. Kiến Trúc Cây Giao Diện Cắt Khung (Tree Structure)
Một Flow (Ví Dụ `Browser`) Không Thể Tự Nó Chạy Trút Cáp Mạch Máu Cắt.
Nó chứa bên trong nó Rất Nhiều Nhánh Cây.
- Đỉnh Cây (Top-Level Flow) -> Chứa Nhiều Cành Bự (Sub-Flow).
- Cành Bự (Sub-Flow) -> Chứa Nhiều Cành Nhỏ, Hoặc Lá.
- Lá Cây Cuối Cùng Dịch Tễ Lạ (Execution / Authenticator) -> Là Hành Động Thực Sự (VD: Bật Màn Hình Nhập Password, Bật Màn Hình Gõ OTP Lệnh Chóp Nhựa).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Oanh Cáp Động Cơ Đọc Dịch Lệnh Cây Của Lãnh Chúa Keycloak Bọc Lụa API:

```mermaid
graph TD
    A[BROWSER FLOW (Cây Cổ Thụ Đáy Lõi DB)] --> B[Cành 1: Cookie Lệnh Oanh Rút]
    A --> C[Cành 2: Identity Provider Redirector]
    A --> D[Cành 3: Forms (Mạch Sub-Flow Khúc Tới)]
    
    D --> E(Lá 1: Username Password Form Bọc Lệnh Cũ)
    D --> F(Lá 2: OTP Form Bọt Cắt Mạch Đứt Kẽ Mã Đáy)
    
    style A fill:#f9f,stroke:#333,stroke-width:4px
    style D fill:#bbf,stroke:#333,stroke-width:2px
    style E fill:#bfb,stroke:#333
    style F fill:#bfb,stroke:#333
```
*Cách Máy Chủ Chạy Oanh Tĩnh Lụa Thép:* Keycloak Sẽ Chạy Từ Trên Xuống Dưới Cắt Đáy. Nó Vô Cành 1 Chạy, Xong Nó Rớt Xuống Cành 2, Rồi Rớt Xuống Cành 3. Tại Cành 3 Nó Vào Lá 1 Rồi Vào Lá 2 Đỉnh Đáy Oanh Mạng Bắt Lụa!

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Thép (Luật Bất Di Bất Dịch: Không Bao Giờ Sửa Cây Gốc Trượt Khung Khớp Lệnh Oanh Rỗng)**
> **Tội Ác Thiết Kế Giao Thức Mạch Rỗng Báo CSRF:** Sếp Kêu "Em Gắn Cái OTP Thêm Vào Web Browser Cho Anh". Bạn Vô Giao Diện Admin Khúc Tới Ngay Mạch, Nhấn Vào Cái Flow Tên Là **`browser`** (Built-in Của Keycloak). Xong Bạn Sửa Thêm Xóa Trực Tiếp Lên Bề Mặt Nó Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống. Rủi Tay Bạn Xóa Nhầm Mạch Cookie. Bùm, Toàn Bộ Hệ Thống Công Ty Sập Không Ai Đăng Nhập Được Nữa (Kể Cả Bạn Bị Văng Ra Ngoài Khỏi Admin Console Mạch Đáy Oanh Lụa Mạng Mạch Máu Cắt Lệnh Đáy!).
> **Biện Pháp Sống Còn Lớp Trọng Lực Thép Mạch Lụa:** TẤT CẢ Các Flow Gốc Built-in Đều Bị Khóa Cứng Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Bằng Khóa Xám Lỗ Lủng Bọt Khung Oanh (Keycloak Đã Thông Minh Khóa Nó Lại). 
> 1. Bạn PHẢI BẤM NÚT CÔNG TẮC **`Duplicate`** (Nhân Bản Lệnh Oanh Rác Bọt Mạch Kéo API).
> 2. Clone Flow Đó Ra 1 Cái Bản Sao Đáy Oanh Tên Của Bạn (VD: `Custom-Browser-Flow`).
> 3. Bạn Xào Nấu Xóa Trút Khung Đáy Sửa Tha Hồ Trên Bản Sao Này Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt. Nếu Hư, Chả Sao Cả.
> 4. Xong Xuôi Thử Nghiệm Bọc Thép Dịch Tễ Lạ Kẽ Trút Rỗng, Bạn Mới Vào Tab **Bindings** Đổi Cái Máy Chủ Kích Hoạt `Custom-Browser` Đè Lên Trái Tim Lõi Trọng Điểm Cáp Bọc Thép! An Toàn Cấp Ngân Hàng Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa!

---

## 4. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Khung Giao Diện Admin Authentication Của Keycloak Khung Cắt Oanh Lụa Mạch Lệnh. Sếp Thấy Cái 'Identity Provider Redirector' Nó Luôn Nằm Ở Vị Trí Ưu Tiên Rất Cao Trên Cây Cổ Thụ Đáy Lõi DB 'Browser Flow'. Việc Nó Nằm Trọng Kẽ Bọt Cắt Trắng Đó Có Mục Đích Chữ Khớp Lệnh Gì Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp? Lỡ Kéo Nó Xuống Dưới Đáy Mạch Oanh Giao Dịch Bọc Lụa Đỉnh Chóp Thì Hậu Quả Thế Nào?**
- **Senior:** Dạ thưa sếp, Đây Bằng Chứng Của Thiết Kế Kiến Trúc Thượng Tầng Trượt Nhựa Dưới Đáy Mạch Hoàn Mỹ Của Đội Ngũ RedHat Cắt Khóa Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ:
  - Cái Nhánh `Identity Provider Redirector` Đó Nhiệm Vụ Của Nó Là Lắng Nghe Cái Tham Số Lệnh Chóp `kc_idp_hint` (Nhảy Thẳng Khách Sang Màn Hình Login Của Bố Già Google/Facebook Mà Bỏ Qua Form Login Gốc Của Lãnh Chúa Keycloak - Đã Học Ở Chương Social Login Đỉnh Đáy Oanh Mạng Bắt Lụa).
  - Vị Trí Của Nó BẮT BUỘC Nằm Gần Đỉnh Cây (Ngay Sau Thằng Check Cookie Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ). 
  - Nếu Sếp Kéo Thằng Đó Rơi Rụng Rút Cáp JSON Mạch Cắt Oanh Trọng Lõi Tự Trị Xuống Đứng Dưới Cái Thằng Nhánh `Username Password Form`. Hậu Quả Khủng Khiếp: Máy Chủ Sẽ Chạy Tới Form Cũ Rích Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch, Nó Bật Màn Hình Bắt Khách Gõ Pass, RỒI SAU ĐÓ Chạy Xong Mới Tới Cái Nhánh Redirector Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Đòi Đẩy Sang Google! Lúc Đó Là Quá Trễ, Khách Văng Trái Trải Nghiệm Hoàn Toàn Hỏng Bét Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh!

---

## 5. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Server Administration Guide - Authentication Flows.
