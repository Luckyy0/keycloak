# Lesson 5: Bản thể Người Dùng (Users)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Lột xác Bảng "Users" truyền thống. Trải nghiệm cấu trúc Dữ liệu Cốt lõi của Keycloak khi lưu trữ Con người, Trạng thái Khóa, và Các Thuộc tính Động (Attributes). Chào mừng bạn đến với Cục Nam Châm Của Trái Đất Đa Dạng.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. User Model của Keycloak Không Gắn Cứng Lịch Sử
Trong các Khung làm việc Giao diện Web (Spring Security Cũ, PHP Cũ). Một User Được Định Nghĩa Bằng Cái Class (Bảng) Gắn Chặt Đủ Thứ Bụng: `Tên, Tuổi, Giới tính, Quê quán`.
Keycloak Thì Khác: Bảng User Gốc (User Entity) Cực Kỳ Khô Khan và Thiếu Thốn.
Bảng gốc Của Keycloak CHỈ DUY TRÌ NHỮNG CỘT CỐT TỬ CỦA ĐỊNH DANH (IDENTITY):
- `ID` (UUID Bất biến).
- `Username` (Tên Định Danh Bề Mặt).
- `Email` (Biển Số Xe Khẩn Cấp Mạng Liên Lạc).
- `Enabled` (Sống Hay Đã Bị Chém Chết/Khóa).
- Trạng thái Duyệt (Email Verified).
Tất Cả Mọi Data "Mang Tính Người / Nghiệp Vụ" (Ví Dụ: Chức Vụ, Mã Nhân Viên Bệnh Viện, Size Quần Áo) ĐỀU BỊ TRỤC XUẤT Khỏi Bảng Cốt Lõi Này, Đem Lưu Qua Một Nơi Gọi Là **Bảng Thuộc Tính Mở Rộng (User Attributes)**.

### 1.2. Tính Bất Sát (Immutable Core) vs Động Lực Học (Dynamic Attributes)
Triết lý Kiến trúc: Nòng cốt Định Danh Phải Gọn Nhẹ Nhất Để Truy Vấn Nhanh Nhất (Chạy Qua Bộ Đệm Cache Lõi).
Nếu Công Ty Bạn Bán Áo Quần (Cần Thuộc Tính Size_Áo), Bạn Chạy Sang Công Ty Bán Ô Tô (Không Cần Size Áo Nữa). Bạn Không Cần Phải Vào Database Chạy Lệnh `ALTER TABLE` Đập Bỏ Cột Size_Áo Đi. 
Tính Linh Hoạt Attribute Cho Phép Gắn Bất Kỳ Bãi Rác Data Nào Dưới Dạng Cặp Khóa-Giá Trị (Key-Value) Vào Thằng User Mà Mệnh Đề Cốt Lõi Vẫn Chạy Vù Vù Không Nghẽn Cổ Chai.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Kiến trúc Lòng Chảo Dữ Liệu Bị Cắt Tách (Normalized Storage Architecture):

```mermaid
graph TD
    subgraph "Tầng JPA - Bức Tranh Tách Bảng Hoàn Hảo"
        UE[Bảng Cốt: USER_ENTITY <br/> Chỉ 5 Cột Phục Vụ Login]
        
        UA[Bảng Râu Riêng: USER_ATTRIBUTE <br/> Data Key-Value: Chức vụ, Phòng Ban, Mã Vùng]
        
        UC[Bảng Nhạy Cảm Mật Mã: CREDENTIAL <br/> Chứa Băm Hash Pass, Seed TOTP Google]
        
        UR[Bảng Ép Chạy Bằng Lệnh: USER_REQUIRED_ACTION <br/> Chứa Cờ Bắt Đổi Pass Ngày Mai]
    end
    
    UA -->|Khóa Ngoại Trỏ Về| UE
    UC -->|Khóa Ngoại Trỏ Về Tuyệt Mật| UE
    UR -->|Khóa Ngoại Trỏ Về| UE
    
    Note over UE,UR: Khi Một Thằng API Xin Thông Tin Tên (Profile). Keycloak Lôi 2 Bảng Đầu.<br/>Nó KHÔNG BAO GIỜ Chạm Xuống Bảng Credential Để Rò Rỉ Bí Mật.<br/>Điều Này Bảo Đảm Tốc Độ Trích Xuất Token Kỷ Lục.
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đối Không Dùng Bảng Attributes Để Lưu Blob / File Rác**
> Có Lập Trình Viên Backend Làm Ẩu: Lấy 1 File Bức Ảnh Avatar Khuôn Mặt Dài Tới Khúc 5 MB Băm Thành Base64, Sau Đó Nhét Mẹ Vào Cái Khóa `Avatar` Bên Trong Bảng User Attribute Của Keycloak.
> **Hậu Quả Sinh Tử:** MỖI MỘT LẦN Thằng User Đăng Nhập Hoặc Quẹt Thẻ Token. Lệnh Máy Chủ Quét Qua Bảng Bị Phình To 5 MB Đó. Cache Nhồi RAM Phình Cháy Cả Máy Chủ Lõm Hết Trí Nhớ (OutOfMemory JVM). Keycloak Chỉ Đóng Vai Trò Phát Hộ Chiếu Trọng Lượng Nhỏ. Bọn Data Khổng Lồ Như File Ảnh LÀ CỦA BẢN THÂN APP BACKEND MANG ĐI LƯU VÀO CLOUD AWS S3 Hoặc PostgreSQL Đuôi Blob. Keycloak Bị Dồn Trọng Tải Ẽo Oặt Chết Cứng.

> [!CAUTION]
> **Cái Bẫy Đụng Nhau Email Khét Tiếng (Duplicate Emails Conflict)**
> Khi Khởi Động Server. Keycloak Cho Bạn 1 Quyền Năng Cấu Hình Gọi Là **"Login With Email"**. Nếu Bật Nó, User Được Quyền Gõ Tên Username Cũ Rích Hoặc Đổi Sang Gõ Email Lành Lặn.
> **Vấn Đề Đau Đớn Của Hệ Cũ:** Trớ trêu 1 Số Công ty Cũ, 2 Nhân Sự Nhập Chung Bảng Lương, Họ Đăng Ký Trùng 2 Cột Khác Nhau Nhưng CHUNG ĐÚNG 1 CÁI ĐỊA CHỈ EMAIL `info@congty.com`.
> **Kiến Trúc Keycloak Gầm Lên:** Keycloak Mặc Định BẬT Đinh Tán Ràng Buộc Kép Bắt Ép Unique Email Cực Gắt Gao Ở Database Dưới Cùng Của Vương Quốc (Constraint Unique Realm_ID + Email). Đảm Bảo Hoàn Toàn 1 Email Không Đăng Nhập Được 2 Bản Thể Ngược Chiều Nhau Đề Phòng Đụng Token Và Thảm Sát Lộ Quyền Kẻ Này Qua Kẻ Kia (Thiết Kế Không Cứng Thằng Khác Qua Hack Lấy Data Ngay).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Thép Chặn Cửa Người Chơi - `Required Actions` (Hành Động Khóa Buộc Tội):

Anh A là nhân viên, hôm nay công ty cấp tài khoản (Pass tạm là 123456).
- Môi trường Đen Tối OIDC Sẽ Vô Cùng Nguy Hiểm Nếu Cứ Để Mật Khẩu Chết Như Vậy.
- Nhưng Code Đằng Sau Rất Phức Tạp Khi Bạn Tự Bắt Màn Hình.
- Trong Keycloak: Bạn Gắn Cờ "Required Action" Tên Là `Update Password` Cho Anh A. (1 Click Cấu Hình Chuột).
- KẾT QUẢ: Sáng Mai Anh A Bấm Đăng Nhập Tòa Nhà. Khớp Pass Đúng. Cổng Chưa Cho Qua. Cái Máy Chủ Ảo Của Cổng Tự Động Chặn Đứng Đường Của Anh A, Văng Ra Cửa Sổ Thông Báo To Tướng Vạch Ra Yêu Cầu Chỉnh Sửa Form Mật Khẩu Kẻo Không Cho Qua Bất Cứ Mảnh Phía Trong Nào. Lập Tức Hoàn Thiện Vòng Đời Self-service. (Bao Gồm: Ép Xác Minh Mail Verified Email, Ép Khai Báo Họ Tên Đầy Đủ Update Profile, Ép Cấu Hình WebAuthn YubiKey). Tất Cả Auto Tự Chạy Không Một Dòng Logic BackEnd Gánh Tải.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Liên Quân Đổ Bộ (Federated Users / Transient Identity):**
  - Câu Truyện Bóng Ma (Shadow Accounts): Anh Giám Đốc Sang Ngân Hàng Kế Bên Chơi OIDC Đăng Nhập Về Keycloak Ta. 
  - Trong Hệ Máy Chủ Của Ta TRƯỚC ĐÓ Hoàn Toàn Trắng Sạch Không Tồn Tại Cái Tên Giám Đốc OIDC Đó.
  - Lúc Ổng Nhập Thông Tin Khớp Bắn Token Chạy Về Thành Ta Chống Lưng. Lớp Lõi Dữ Liệu Tự Động Cất Bóng Kéo Chân (JIT Provisioning) Khắc Tên Của Ổng Thành 1 Dòng Mới Tinh Vào Trong Bảng `USER_ENTITY`. Lập Thành Gốc Rễ Nội Tại Tàn Phế Để Thừa Hưởng Các Cơ Cấu Tạo Rule Sau Này Và Mapping Vào Các ID Cũ Hữu Tận. 
  - Người Ta Gọi Đó Là Bản Thể Mượn Đầu Heo Nấu Cháo Nhưng Tạo Liên Kết Siêu Bền Chắc Trọn Đời Không Sứt Mẻ. (Tích hợp Lõi Federation Link Identity Bất Bại Kể Cả Sang User Cũ Hoặc Mới).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Hệ thống của Công Ty Tự Động Thêm Khách Hàng Bằng Script API Dội Lên Liên Tục Gây Chậm Chạp Dữ Liệu Máy Chủ Khủng Khiếp, Nguyên Nhân Vì Sao Mà Tầng Lưu Trữ Lại Trút Chặn Tắc Nhanh Như Vậy So Với Việc API Bắn Lên Get Token Chạy Vèo Vèo 5ms?**
- **Junior:** Chắc Dữ liệu đầy thì ghi chậm.
- **Senior:** Cái Vòng Tròn Đóng Chết Khác Nhau Giữa READ (Đọc Cắm Vào Bộ Đệm Cache) và WRITE (Ghi Cày Xuống Mặt Đất).
- Hàm Mượn Data Vào Token Trải Phẳng Chạy Kéo Qua Sông RAM Bộ Đệm Infinispan Rất Ngon, Tới Hàng Nghìn Dòng/Giây Vèo Vèo Không Chết.
- Nhưng Hàm TẠO MỚI (CREATE/UPDATE/DELETE USER) Phải Khoét Xuyên Cả Trái Tim Thẳng Kéo Vào Khóa Ngoại PostgreSQL Dưới Đáy Mồ Tránh Đụng Độ Acid (Tính toàn vẹn Database JPA Cao Độ). Quá Trình Write Liên Tục Trăm Yêu Cầu Tranh Chấp Sẽ Gây Khóa Hàng (Row Locking Của SQL). Đó Là Điểm Chết Tử Huyệt Làm Mọi Nỗ Lực Load Trái Gây Chết Lịm CPU. 
*(Kiến trúc Băng Thông Write Keycloak Tốn Thời Gian Hơn Rất Nhiều So Với Read. Muốn Bulk Load Hàng Vạn User, Phải Ngừng Cache Và Phóng Theo Script Chặn Batch SQL, Chứ Cấm Có Spam Từng Con API Gọi Tốn Dữ Liệu Connection Quá Số Lượng Của Agroal Cháy Mạch Máu DB).*

**2. Làm Thế Nào Xử Lý Ca Người Dùng Vẫn Nhớ Tên Đăng Nhập Nhưng Hư Mất Cái Email Xác Minh Ban Đầu Mãi Không Liên Lạc Được Trong Môi Trường Trái Phiếu Zero Trust Chống Đổi Mạo Danh Hack Data Khủng?**
- **Junior:** IT Reset Cột Bằng Lệnh Tay Đi.
- **Senior:** Lỗi Quyền Lực Xâm Phạm Tuyệt Đối (Admin Cấm Sửa Đổi Trực Diện Bơm Dữ Liệu Lầm Của Bằng Chứng Nhạy Cảm).
Mọi Yếu Tố Xác Định Bản Thể Bắt Buộc Dựa Qua Quy Trình Kích Hoạt Tự Thân (Verification).
Kỹ Sư IT Chui Vào Máy Chủ Keycloak Của Người Dùng. Phủ Quyết Đi Cờ Hành Động Kích Hoạt (Verify Email) Và Bơm Lệnh Cưỡng Ép (Required Action) = `Update Email`. 
Cửa Vòng Bảo Mật Chạy Ngang Rất Đẹp: Khách Dùng Pass Cũ Đập Cổng. Cổng Đứng Lại! Nó Hỏi Hãy Xóa Địa Chỉ Cũ Và Đánh Dòng Mới Rõ Ràng. Hệ Thống Tự Quăng Gói Tin Nổi Đi Tới Email Lạ Đó Bắt Xác Định Nhấn OK Sống Động. Quá Trình Tự Băng Bó Lành Sạch Không Lưu Mã Phản Hệ Bằng Code Tráo Tháo IT Vụ Lợi Mua Chuộc Đổi Mail Gây Hư Căn Mạng Trữ Lệnh.

**3. Đâu Là Sợi Dây Chân Linh Nối Thằng Người Dùng Nội Tại Gắn Máu Với Các Tổ Chức OIDC Nước Ngoài Hầm Bà Lằng Trong Chế Độ Identity Brokering Nhiều Vô Kể Của Các Công Ty Đại Chúng?**
- **Junior:** Nó copy đắp chữ nối vào cái bảng Attribute là được.
- **Senior:** Một Sự Sỉ Nhục Mô Hình Database Trưởng Thành. Bảng Rác Attribute KHÔNG ĐƯỢC PHÉP Lưu Lõi Mạch Identity Ngoại Tộc Nhờ (Khóa Lệnh Dễ Lủng).
Tầng Database Của Keycloak Có Hẳn Riêng Một Bảng Thiết Kế Cho Hiệp Ước Gọi Là Liên Hợp Danh Tính **(`FEDERATED_IDENTITY`)**.
Bảng Này Làm Cái Cầu Kẹp (Cột A: UUID_NỘI_TẠI, Cột B: IDENTITY_PROVIDER (Vd: google), Cột C: ID_CỦA_TÀI_KHOẢN_GOOLE (Vd: 181283123891)). 
Khi Dội Bom Đăng Nhập Google Lên, Keycloak Search Nhanh Sống Gấp Bằng Index (Broker_ID), Đọc Rút Trúng Dây Kết Cấu Thép Kéo Dây Rớt Tọt Về Đích 1 ID Nội Tại Xịn Trả Data Bơm Access Token Phẳng Lì Không Đi Rác Đường Dẫn Ẽo Ợt String Attribute.

---

## 7. Tài liệu tham khảo (References)
- **Keycloak User Model:** Database Schema and JPA Mappings.
- **OpenID Connect:** End-User Claims and Standard Attributes.
