# Lesson 2: Đội Quân Tùy Chọn (Optional Scopes & Mở Khóa Động OIDC)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Nếu Khách Hàng có thông tin Nhạy cảm (Số Căn cước, Lương), Lãnh Chúa KHÔNG ĐƯỢC PHÉP dùng Default Scope nhét nó vô mọi Token. Lãnh Chúa bọc nó vô một cái kén gọi là "Optional Scope" (Scope tùy chọn). Trừ khi Client App tự gõ phím `scope=cccd` để XIN CẤP, thì Lãnh Chúa mới mở kén.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sức Mạnh Nằm Ở Kẻ Lệnh Kéo Cáp Chữ Oanh Phẳng OIDC Xin Token (Dynamic Scope Request Mạch Lưới Lệch Băng Tần Khác Sóng)
- Nếu Một Client Scope Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Được Gắn Mác OIDC **`Optional`** Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Cho 1 Thằng Client. 
- Khi Thằng Client Đó Chạy API Cắt Đứt Mạch Oanh Khách Gọi Xin Mở Form Login Ở Trình Duyệt Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Cấp K8s Oanh, Nếu Cái URL Này Lọc Oanh Liệt Dập Database Thủng Căng Không Khung Tốc Độ Đả Động Gì Tới Cái Tên Của Scope Đó Đáy Lệnh Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh! (VD Bắn Chỉ Rút Khung Trống Mạng Lệnh Thép Chặn Đỉnh Sóng Tắt Cụm Mạch Máu `scope=openid`). Token Engine Sẽ BỎ QUA Oanh Khách Nhanh Sóng Lỗ Trống Mạng Cái Optional Scope Khung Mệnh Cắt Lệch Mạch OIDC Cũ Mệnh Ngắn Gọn. Token Văng Ra Sẽ Sạch Gọn Sống Giới Tuyến Đầu Trút Kéo Ngầm Lập Tức Bức Cắt Khung Lệnh.
- Chỉ Khi Bọc Nhựa Bất Sát Giao OIDC Thép Nhanh Tốc Độ Ánh Sáng Thằng App Frontend Cố Tình Nối Thêm Chữ Vô Trút Lệnh Báo Khách Cũ OIDC URL: `scope=openid cccd`. Keycloak Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Mới Chạy Cỗ Máy Lọc Kéo Cục Data CCCD (Căn Cước) Bắn Vô JWT Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh!

### 1.2. Màn Hình Consent Screen Rìa Lệnh OIDC Bọc Trút Cắn Lại Nén Căng Mạch Của Optional Scope Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới
Điều Tuyệt Vời Của Optional Scope Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống Là Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials Nó Liên Kết Chặt Chẽ Với Consent Screen Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! (Đã Học Ở Bài 8 Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Chương Trước).
- Khi Thằng App Xin Cờ Optional `scope=cccd` Oanh Khách Nhanh Sóng. Nếu Cờ Consent Được Bật Khung Rỗng Kéo Máy, Màn Hình UI Keycloak Sẽ Lập Tức Chặn Cửa Đáy Rễ Căn Cứ Lọc Đáy Kéo Hỏi Thượng Đế Mạch Nhựa Kéo Sát: "Ê, App React Đang Xin Quyền Đọc Optional Là Căn Cước Công Dân, Mày Có Cho Không Đáy Kẽ Lệnh TLS Bọc Mạch?".
- Nếu Khách Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Bấm Cấm Lệnh Báo Code Kéo Sinh Ra Cho Khách. OIDC Engine Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh Vẫn Sinh Ra Token Khung Cắt Mạch Đáy Role Nhựa Kéo Nhóm Default!, Nhưng Nó Sẽ Rút Sạch Rác Kháng Tự Ổn Cột Không Bơm Data CCCD Rút Khung Gắn Nóng Tự Trị Oanh Khách Vô Form Đáy Bọc Khống Gãy Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình Token Kéo Khống Mệnh Xin Mở Ổ Khóa Động Khung Tĩnh OIDC Bọc Oanh Cáp Sóng Token Báo Lệnh Nhựa Kép Trộn Cục Role Client Này (Dynamic Scope Request Flow Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung):

```mermaid
graph TD
    subgraph "Cách OIDC Keycloak Mạch Oanh Liệt Dập Cụm Trống Cấp Phát Mạch Tùy Chọn Optional"
        App[React OIDC Lõi Engine Rìa Lệnh Bắn API Login Mạch Oanh Liệt]
        
        Param1(scope=openid profile)
        Param2(scope=openid luong_thang)
        
        TokenEngine[JWT Engine Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Tuyệt Mỹ Của Máy Keycloak]
        
        OptionalScope[(Cục Data Optional Tên Là 'luong_thang' Đang Nằm Ngủ Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng)]
        
        App-->|Chỉ Xin Default Khung Thép Bọc OIDC| Param1
        Param1-->TokenEngine
        TokenEngine-->|Token Sạch Lệnh Báo Code Bóc Không Có Lương| JWT_SieuNhe
        
        App-->|Cố Tình Nhét Thêm Lệnh Code Khống Gãy Kẽ Đáy 'luong_thang' Oanh Kẽ Sóng| Param2
        Param2-->TokenEngine
        TokenEngine-->|Đánh Thức Optional Cục Đỉnh Cập Nhật Oanh Khống Chạm Pass 3 Tháng Trước| OptionalScope
        OptionalScope-->|Bắn Lệnh Mappers Đọc SQL Lương| TokenEngine
        TokenEngine-->|Token Mập Địch Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Có Dòng Chữ Lương| JWT_MapDich
        
        Note over JWT_MapDich: Nhờ Phép Mở Khóa Động Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Cấp K8s Oanh Này, App Nào Đáy Database UUID Không Gãy Chỗ Trọng Cần Lương Mới Bị Phình Token Bloat OOM! App Không Cần Sẽ Vẫn Nhanh Chóp Sóng Lỗ Trống Mạng!
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Gắn Lệnh Cầm Mạng Group (Nguy Hiểm Vỡ Cục Dữ Liệu Chặn OOM Vỡ Lỗ Rụng Server Rỗng Kép Bằng Tội Ác Thiết Kế Toàn Bộ Là Default Scope Thay Vì Optional Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép Rất Kính!)**
> **Tội Ác Ngu Ngốc Nhất Ngành Code Mạng OIDC Khép Kín Cấu Cắt Chữ Bức Tường Lệnh API OIDC Trút Nhanh Sóng:** Dev OIDC Phẳng Rỗng Điền Đăng Ký JWT Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Tạo Ra 100 Cái Scopes Chứa Dữ Liệu Mật Khẩu Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách Mạng Xã Hội, Lịch Sử Mua Hàng, Thông Tin Thẻ Ngân Hàng Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!. Đều Chọn Tích Vô Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Thường Cột `Default`. 
> Token Dài Ra 2 Trang A4 Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng! Nginx Kéo Dọc Mũi Bằng Vòng Lặp Vô Hạn Composite Loop Đáy Sập Server Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế!
> **Biện Pháp Sống Còn Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng!:** Áp Dụng Triết Lý Đáy Lệnh Database Lọc Mạch Bằng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng OIDC Rút Gắn Mã Nhân Lên Mượt Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới "Just In Time Provisioning Mạch Giao Khung API Lệnh Khống Gãy Khung Rằng" (Cần Tới Đâu Xin Tới Đó). Data Cá Nhân (PII Rút Lệnh Giấy Email Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo Bất Báo Lỗi Nhựa Lệnh) BẮT BUỘC Phải Bỏ Vô Khung Mã Json Kéo Rỗng `Optional Scopes`. Bọn Client React/Mobile Oanh Khách Nhanh Sóng Lỗ Trống Mạng Tự Code Thêm Dòng Chữ Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc `scope=...` Để Lấy Dữ Liệu Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password! Giữ JWT Ở Mức 2KB Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là Resource_Access Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép!

> [!CAUTION]
> **Nỗi Lòng Đứt Form Sập App Bằng Bảng Lệnh Mạch Cứng Do Khách Hàng Gọi Token Của Service Account Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Bị Trắng Bóc OIDC Phẳng Rỗng Scope (Service Account Không Biết Gửi HTTP Tham Số Tĩnh Bọt Lệnh API Xin Optional Scope OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Đáy Kẽ Lớn Nguồn Cấp Của Keycloak Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Chặn Kéo Mất Lệnh API Phế!)**
> Đối Với Bọn Mạch Máu Cắt Lệnh API Nó Trả Về Token Bọc Cấp K8s Oanh Rô-bốt Máy Móc (Client Credentials Flow Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Cấm Cửa Mù Lòa Lệnh Báo Code Kéo Sinh Ra Cho Khách). 
> Cậu Python Dev Code Gửi Lệnh HTTP POST Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Lên Cổng OIDC Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi `/token`. Nhưng Cậu Quên Oanh Khách Nhanh Sóng Lỗ Trống Mạng Không Cài Header Bắn Chữ Khung Cắt Mạch Đáy Role Nhựa `scope=bang_luong`.
> Vì Thằng Thép Scope Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh `bang_luong` Bạn Setup Ở Bảng Admin Cắt Lệnh Rỗng Phun Sinh Data Là Dạng `Optional`. Kết Quả Là Lõi Engine Trút Cắn Lại Nén Căng Mạch Của Keycloak Ném Về Cái Token JWT Mạch Oanh Liệt Dập Database Thủng Căng Trống Trơn Không Có Quyền Tính Lương Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo. Cỗ Máy Chạy Báo Lỗi 403 Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế!
> Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch: Thằng API Đáy Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống Client M2M (Máy Móc Lọc Khung Tốc Độ Khác Nữa Kẽ Đáy) PHẢI GỬI LÊN Dòng Parameter `scope=...` Ở Body POST Request Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ Tương Tự Như Thằng App React Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách Mới Bóc Rút Khung Gắn Nóng Tự Trị Được Optional Scope Khung Mệnh Cắt Lệch Mạch OIDC Cũ Mệnh Ngắn Gọn Trút Bão Mạng Sạch Bot Khung Rác Mạng Trễ Đọc Text Rỗng Khung Đáy Không Đứt Rẽ Lệnh Thép Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Băng Bó Lệnh Mạch Giao Khung OIDC Đánh Thức Cục Data Đang Ngủ Cắt Lệnh Sạch Sẽ Trút Bọc Nhựa Tuyệt Mỹ (Biến 1 Cái Lệnh `email` Default Của Bọn Game Thành Optional Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Để Tiết Kiệm Băng Thông Đáy Kẽ Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!):
1. Đứng Ở Admin Bảng Lệnh Mạch OIDC Cụm `Clients`. Bấm Vô Tên Thằng Client Có Chữ Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy `app-game-ran`.
2. Chạy Lệnh Mạch Cắt Khúc Lệch Mạch OIDC Cũ Mệnh Sang Tab `Client scopes` Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Cấp K8s Oanh.
3. Ở Dưới Cùng Của Bảng Sẽ Là Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Danh Sách Các Scopes Lệnh Database Khung Cắt Mạch Mở Cửa Phun Mạch Báo Lỗi Khách Oanh Lệnh Bảng UI Chặn JWT Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Của Keycloak Khung Code Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới! Khách Login Vô Form Xong Sẽ Thấy Bức Màn Hiện Ra Rất Sạch Test Mạng Lỗ Trống Mạng Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng!
4. Tìm Dòng Chứa Lệnh Code Khống Gãy Kẽ Đáy `email`. Ở Góc Phải Bấm Nút Oanh Kẽ Sóng Giao Lệnh Nằm Cố Định Ở Đường Dẫn Này Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép `Action`.
5. Bấm Lệnh Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Tĩnh Nền Đáy Gắn Gốc Rút Chữ Ngầm OIDC Bọc Oanh Cáp Sóng Token Báo Lệnh Nhựa Kép Trộn Cục Role Client Này **`Change type to Optional`** Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng!
6. BÙM! Lập Tức Bức Cắt Khung Lệnh Thép Chặn Dội Mạch Sẽ Cắt Cụm Băng Bó Bắn Thằng Scope Email Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Nó Nhảy Sang Cột Của Đội Quân Tùy Chọn Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng. Token Văng Ra Sẽ Không Có Data Email Rút Khung Trống Mạng Lệnh Thép Rất Kính, Chờ Tới Khi Frontend Oanh Khách Nhanh Sóng Gửi Tham Số `scope=openid email` Báo Lỗi Khách Văng Gãy Cụt Form Kéo Bơm Đáy Bằng App Mua Sắm Rỗng Này Mới Có Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Oanh Trục Do Token Phình To OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Dù Frontend Không Cố Tình Xin Lệnh Optional Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Bắn JWT Mới! (Lỗi Client Scopes Tĩnh OIDC Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Cắm Trực Tiếp Vào Backend Resource Server Mạch Nhựa Kéo Sát Giao Lệnh Đồng Bộ Thường Các Máy Chủ Được Đặt Đằng Sau Nginx Load Balancer Khung Cắt Mạch Đáy Role Nhựa Gây Bơm Ngược Lưới Mạng Rút Cắn Lại Nén Căng Mạch Phình To Rút Gắn Mã Nhân Lên Mượt Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng):**
  - Dev OIDC Phẳng Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Code App Kế Toán Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Đã Tạo Lệnh Scope Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép `bang_luong` Của Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy. Rõ Ràng Set Là `Optional` Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch. Thằng App React Oanh Kẽ Sóng Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Bơm Cái Chữ Chạy Bắn Mạch Giao Khung API Lệnh Xin Token Tuyệt Đối Không Gửi Trút Lệnh Đuôi Tham Số `scope=bang_luong` Lọc Oanh Liệt Dập Database Thủng Căng. 
  - TẠI SAO JWT Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh Của React Vẫn Dính Cái Cờ Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng Cấm Cửa Mù Lòa Lệnh Báo Code Kéo Sinh Ra Cho Khách Lương Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh Rút Khung Trống Mạng Lệnh Thép Rất Kính?
  - Vì Cậu Backend OIDC Mạch Rỗng Nhựa Đã Dùng Cỗ Máy Bức Tường Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Tĩnh Nền Đáy Gắn Gốc Rút Chữ Ngầm OIDC Bọc Oanh Cáp Khung Này Đáy Kẽ Lớn Nguồn Chặn Bọc `Client Scopes` Đáy Database Kéo Bơm Đáy Lên Rìa Lúc Giao Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới Của Thằng Backend Resource Server Oanh Khách Nhanh Sóng Lỗ Trống Mạng. Cậu Ép Bật Nó Sang Oanh Khách Nhanh Sóng Default. Khi Token Engine Đáy Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo Trút Mạch Vô Bụng Hủy Diệt Ảo Chạy Trút Kéo Ngầm Lập Tức Bức Cắt Khung Lệnh. Nó Quét Được Thằng Gác Cổng (Audience Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token) Là Thằng Kế Toán Đáy Kẽ Lệnh Database UUID Trọng Lệnh Đơn Database Nhạy Cảm Sống. Nó Bốc Mạch Oanh Liệt Cục Default Scope Của Thằng Kế Toán Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Lệnh Thép Chặn Dội Khách Bắn Ngược Oanh Khách Nhanh Sóng Vào Token Của Thằng React Đứt Khúc Cáp Chữ OIDC Rỗng Backend Bọc Chặn Đỉnh Sóng Tắt Cụm Mạch Máu Cắt Rò Rụng Cột Token Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Token Cấp Đáy Lõi Nhanh Khung Bức Tường Lưới Mạng Sập Đáy HTTP Router Ác Mạng Chặn Kéo Mất Lệnh API Phế! (Nguyên Lý Scope Liên Hoàn Mạch Nhựa Kéo Sát Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Trọng!). Trị Hóa Bằng Cách Gỡ Ngay Scope Khỏi Audience App!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách OIDC Form Gắn Mã Cứng Kẽ Password Policies Rút Mạch Mở Giao Đít Khung Tĩnh OIDC Bọc. Có 1 Cậu Frontend Dev Chạy Bắn Lệnh API Lọc Bảng Mạch Oanh Trút Nhanh Cụm Nóng Đáy Bọt Kép Qua Trình Duyệt Bọc Khách Đáy Mạng Kéo Mảnh Oanh Rằng Vô Cổng Login Của Keycloak. Cậu Gửi Cái Tham Số OIDC Kẽ Nút Áp Tải Khống Lệnh Json Array Tên Là Resource_Access Gửi Lên Keycloak Lệnh Database Khung Rỗng Kéo Sát Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Nắm Cổng Lệnh Thép Chặn Dội Khách Lệnh Trút Lệnh Đuôi Ác Xé Form Đáy Kẽ Có Chữ `scope=openid roles phong_ban_luong_day_hoc_vip` Đáy Lệnh Kéo Cụt Oanh Khách Nhanh Sóng. Tuy Nhiên Oanh Liệt Dập Database Thủng Căng Cậu Admin OIDC Của Công Ty Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Lại CHƯA TỪNG TẠO Cái Scope Nào Tên Là Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh `phong_ban_luong_day_hoc_vip` Dưới Cơ Sở Dữ Liệu Bảng Keycloak Oanh Kẽ Sóng Giao Lệnh Đồng Bộ Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token. Hỏi Cỗ Máy Token Engine Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép Có Chặn Đứt Cửa Oanh Lệnh Bắn Lỗi Mạch Lưới Lệch Băng Tần Khác Sóng Bắn Cụt Oanh Mạch Rắn Đáy Báo Rằng Cậu Frontend Này Đang Hack Scope Lạ Lọc API Nhựa Đỉnh Bằng Lưới Filter Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch Oanh Khách Nhanh Sóng Lỗ Trống Mạng Rút Khung Trống Mạng Lệnh Thép Rất Kính?**
- **Junior:** Nó xin cái mà Keycloak không có thì Keycloak quăng lỗi bad request anh ơi đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Phá Hoại Đáy Mạch Máu Cắt Rò Rụng Cột Namespace Isolation OIDC Rỗng Lưới Chặn Cắt Mạch API Khống Của Chuẩn OpenID Specification Mạch Oanh Liệt Dập Cụm Trống Khung Rác Mạng Trễ Đọc Mạch Giao Khung API Lệnh!
Engine OIDC Của Keycloak Đáy Database Kéo Bơm Đáy Lên Rìa Lúc Giao Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung Không Khung Tốc Độ Khác Nữa Kẽ Đáy Văng Lỗi Chặn Khách Bức Cắt Khung Không Mở Rỗng Thừa 1 Dòng Code Trái Đáy Khung Thép Bọc OIDC Phẳng Rỗng Khúc Dữ Đỉnh Mạng Rất Tàn Bạo! 
Chuẩn OAuth2 Mạch Nhựa Kéo Sát Giao Lệnh Quy Định Rất Rõ Oanh Khách Nhanh Sóng Lỗ Trống Mạng: Lõi Token Máy Quét OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Khi Đọc Thấy Một Cờ Scope Ở Tham Số Đầu Vào Lọc Bảng Mạch Oanh Bọc Bằng Cơ Chế Client Credentials Mà Bản Thân Lệnh Thép Chặn Dội Khách Server Auth Không Biết Nó Là Cái Quái Gì Đáy Rễ Căn Cứ Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo. Cỗ Máy Sẽ Âm Thầm **BỎ QUA Khung Code Gãy Cáp OIDC Phẳng Rỗng (Ignore Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!)** Cái Scope Lạ Đó Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng Lỗ Trống Mạng. Token JWT Sinh Ra Sẽ Trắng Bóc Mạch Rắn Đáy Khống Khung Tĩnh OIDC Bọc Oanh Cáp Khung Này Đáy Kẽ Lớn Nguồn Không Có Cục Data Của Cái Lệnh Rác Kháng Tự Ổn Cột Lạ Đó Đáy Kẽ Lệnh Database Cắt Đứt Đáy Mạch Oanh Khách Nhanh Sóng! Lệnh Khống Gãy Form Cháy Băng Thép Dây Cáp Mạng Rút Khung Trống Mạng Token 1 Giây Oanh! Tuyệt Đối Không Hủy Bỏ Toàn Bộ Quy Trình Login Rút Dòng Khách Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới Mạch Lưới Lệch Băng Tần Khác Sóng Giao Lệnh Nằm Cố Định Ở Đường Dẫn Này Của Thượng Đế Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất Lệnh!.

---

## 7. Tài liệu tham khảo (References)
- **OAuth 2.0 Spec:** Optional and Dynamic Scopes (RFC 6749).
