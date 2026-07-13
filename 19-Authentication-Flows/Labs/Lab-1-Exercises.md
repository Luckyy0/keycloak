# Lab 1: Lắp Ráp Cỗ Máy Rẽ Nhánh OTP Theo Vai Trò (Role-based Conditional Flow)

## 1. Mục Tiêu (Objectives)
Thiết kế lại luồng đăng nhập Browser: Chỉ bắt buộc nhập OTP Đối với những Khách Hàng Nào Mang Quyền `admin`. Các khách hàng thường (`user`) chỉ cần nhập Password là được qua.
- **Task 1:** Clone Luồng Browser Gốc Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần.
- **Task 2:** Dựng Hộp Đen Sub-Flow OTP Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị.
- **Task 3:** Nạp Cờ Conditional Cho Khách Hàng Thường Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt.
- **Task 4:** Khởi Động Động Cơ Luồng Mới Và Test Oanh Tĩnh Lụa Thép.

---

## 2. Chuẩn Bị (Prerequisites)
Khởi động hệ thống Keycloak bằng docker-compose đã cung cấp.

```bash
cd code
docker-compose up -d
```
Mở Trình Duyệt Truy Cập Admin Console Keycloak tại `http://localhost:8080/` (admin/admin).

---

## 3. Các Bước Thực Hành (Lab Steps)

### Task 1: Nhân Bản Cây Cổ Thụ Đáy Lõi DB Trút Cắt Khung Tương Lai
1. Trong Admin Console Keycloak, Bấm Menu **Authentication**.
2. Tìm Trái Tim Lõi: Flow Tên Là **`browser`**. Nó Bị Khóa Giao Diện Khúc Tới Chặt Oanh Tĩnh (Dấu Khóa Xám).
3. Bấm Nút Menu 3 Chấm Cạnh Tên `browser` Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép -> Chọn **`Duplicate`**.
4. Đặt Tên Mới: **`Role-Based-Browser-Flow`**. Lập Tức Nó Nhảy Sang Giao Diện Bản Sao Mới Oanh Cáp Giao Diện Lệnh Chặt Mạch Lụa Tha Hồ Cho Bạn Sửa Lệnh Oanh Rút Mạch Máu Cắt!

### Task 2: Dựng Hộp Đen Conditional Sub-Flow Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh
1. Tại Giao Diện `Role-Based-Browser-Flow`, Cuộn Xuống Gần Cuối Đáy Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa.
2. Bấm Dấu Cộng Cạnh Nhánh **`Role-Based-Browser-Flow forms`** -> Chọn **`Add execution`**. KHÔNG PHẢI ADD EXECUTION NHÉ LỖ LỦNG BỌT! CHỌN **`Add sub-flow`**.
3. Đặt Tên Sub-Flow: **`Admin-OTP-Condition`**. 
4. Đổi Cái Quyền Trượng Của `Admin-OTP-Condition` Vừa Sinh Ra Thành **`Conditional`**.

### Task 3: Bơm Các Kẻ Chấp Pháp Vào Hộp Đen Trút Cáp Mạch Máu Cắt Lệnh Đáy DB
1. Bây Giờ Bấm Dấu Cộng BÊN CẠNH CÁI THẰNG SUBFLOW VỪA TẠO `Admin-OTP-Condition` Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm. Chọn **`Add execution`**.
2. Chọn Gã Kẻ Chấp Pháp Gác Cửa: **`Condition - User Role`**. Gán Cờ Của Nó Là **`Required`**.
3. Bấm Biểu Tượng Cánh Quạt Răng Cưa Cạnh Cái `Condition - User Role` Vừa Tạo Lệnh Oanh Rác Bọt Mạch Kéo API Dữ Lụa Lỗ Bọt Cắt Trắng.
   - **Alias:** `Role Admin Check`.
   - **Expected role:** Điền Chữ **`admin`**. Bấm Save. Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép!
4. Tiếp Tục Bấm Dấu Cộng BÊN CẠNH CÁI THẰNG SUBFLOW LẦN NỮA Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa. Chọn **`Add execution`**.
5. Chọn Kẻ Chấp Pháp Khổ Sai Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống: **`OTP Form`**. Gán Cờ Của Nó Là **`Required`**.
6. Lúc Này Trong Bụng `Admin-OTP-Condition` Phải Chứa Đúng Theo Thứ Tự: Condition Ở Trên Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh, OTP Form Ở Dưới Cắt Khung Đứt Băng Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ.

### Task 4: Khởi Động Động Cơ Và Thử Lửa Trút Kẽ Mã Bơm Oanh Tĩnh Lụa Thép
1. Bạn Vẫn Đang Ở Bảng Authentication. Bấm Chữ Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị: Cạnh `Role-Based-Browser-Flow` Bấm Vào Nút 3 Chấm Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh -> **`Bind flow`**.
2. Chọn **`Browser flow`**. Cỗ Máy Cũ `browser` Chính Thức Bị Vứt Đi Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị. Máy Chủ Khởi Động Trên Mã Mới Của Bạn!
3. Về Màn Hình User. Tạo Thử 1 Thằng User Tên `teoteo`. Không Cấp Quyền Gì Hết Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.
4. Bạn Login Bằng Màn Hình Thường. Bùm! Vào Thẳng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh. KHÔNG BỊ HỎI OTP Oanh Khung Dịch Lụa Mạch Lệnh!
5. Giờ Ra Giao Diện Admin, Ép Role `admin` Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Cho Thằng `teoteo` Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh. 
6. Đăng Nhập Lại Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị. Bùm Bùm Bùm! Máy Chủ Bật Form Mới Bắt Buộc Nhập Quét Mã QR Zalo Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy! Thành Công Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa!

---

## 4. Dọn Dẹp (Cleanup)
Hủy Mạch Docker Tránh Nặng Máy:
```bash
docker-compose down -v
```
