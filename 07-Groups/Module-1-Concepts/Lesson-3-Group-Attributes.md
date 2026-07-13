# Lesson 3: Mã Hiệu Của Binh Đoàn (Group Attributes & Token Mapping)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Chúng ta không chỉ gắn Role cho Group. Mỗi Binh Đoàn có Thẻ Bài riêng (Attributes). Cả Tập đoàn Vingroup có hàng ngàn Nhân viên nằm trong Nhóm IT. Cậu Backend Dev không muốn Lục tung Profile từng người mà chỉ muốn biết một điều duy nhất khi Khách vào App: "Thằng này thuộc Mã Ngân Sách (Budget Code) của Đội nào?". Group Attributes chính là Kho Lưu Trữ Key-Value siêu cấp này.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Thẻ Bài Định Danh Binh Đoàn (Group Attributes OIDC Khung Code Bọc)
Giống Hệt User Profile Đáy Mạch Máu Cắt Rò Rụng Cột, Thằng Nhóm (Group) Cắt Khúc Lệch Mạch OIDC Cũng Có Cấu Trúc Bụng Để Lưu Trữ Các Biến Đáy Kẽ Lệnh Tĩnh Key-Value.
Ví Dụ:
- Nhóm `/Vingroup/Sales`: Bạn Cắm Vô Bụng Nó Mã Cắt Mạch Đáy `budget_code = 1001` Lệnh Database UUID Không Gãy Chỗ.
- Nhóm `/Vingroup/IT`: Bạn Cắm Vô Lệnh OIDC Bọc Oanh Cáp Mã `budget_code = 5005` Đáy Kẽ Lớn Nguồn.
Bức Tường Đáy Database UUID Của Nó Tách Biệt Hoàn Toàn Tuyệt Nhiên Với Các Khách Hàng OIDC Nằm Trong Nhóm. Khi Kéo Lệnh Nhựa Kép Ở Khách Nhanh Mạch, Thác Nước Không Tự Đổ Mã Mạch Kéo Này Xuống Bụng Từng Người Nằm Tĩnh Đáy Vùng Ruột Của Nó!

### 1.2. Mappers OIDC Kéo Nhựa (Đẩy Dữ Liệu Binh Đoàn Vô Bụng JWT Khách)
Để Bóc Cái Dữ Liệu `budget_code` Của Binh Đoàn Cấp API Mà Nhét Trút Lệnh Đuôi Kéo Vô Cái File JWT Access Token OIDC Của Thằng User Lúc Nó Gọi Xin Cấp Rút Lệnh Giấy Rác Mạng. Chúng Ta Vận Khí Đỉnh Cao OIDC Lệnh `Protocol Mappers`.
Ở Đây Ta Có 2 Cái Máy Bơm Dữ Liệu Lệnh Khống Đỉnh Cụm Kẽ Đội Bất Chạm Đáy Lệnh:
1. Máy Bơm Tên Nhóm OIDC Phẳng (**Group Membership Mapper**): Lõi Engine Nhựa Sẽ Hút Dòng Khung `["/Vingroup/IT"]` Bắn JWT OIDC Cho Lập Trình Viên Đọc Nhựa.
2. Máy Bơm Attribute Của Nhóm (**Group Attribute Mapper**): Lõi JWT Engine Mở Bụng Đáy Khung Rễ Lệnh Database Đỉnh Tìm Group Cha, Rút Mã `budget_code=5005` Bắn Kép Nhựa Oanh Tạc Code Cụm Rỗng Khung JWT Sóng Khách!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Lệnh JWT Nhựa Gắn Mã Binh Đoàn Vào Trái Tim Khách (Group Mapping Data Flow OIDC Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp):

```mermaid
graph TD
    subgraph "Cách OIDC Token Nhồi Nhét Attributes Mạch Giao Khung Group Của Khách Vào Mã Payload"
        Khach[Khách Hàng: Dev1 OIDC Phẳng Rỗng Login Web]
        DB_User[Lõi Tĩnh OIDC: Dev1 Nằm Trong /Vingroup/IT]
        DB_Group_Attr[(Bảng group_attributes Tĩnh: /.../IT Có budget_code=5005)]
        
        Mapper1[Engine Mapper: Group Membership]
        Mapper2[Engine Mapper: Group Attribute OIDC Kéo Nhựa]
        
        Khach-->|Gửi Mã API POST Nhựa OIDC Login| DB_User
        DB_User-->|Đẩy Thông Tin Tới Máy Trộn| Mapper1
        DB_User-->|Kéo Dữ Liệu Nhựa Oanh Lõi Trọng| Mapper2
        
        Mapper1-->|Văng Chuỗi List String Tên Nhóm| PayloadJWT
        Mapper2-->|Rút Mạch Đáy Database Lọc Value Mạch Bắn Kép| DB_Group_Attr
        DB_Group_Attr-->|Trả Chữ 5005 Khung Thép| Mapper2
        Mapper2-->|Nhét Claim Tùy Biến Lệnh Khống Gãy Form| PayloadJWT
        
        PayloadJWT[Cục JWT Cuối Cùng Bọc Oanh Cáp: <br/>{ "groups": ["/Vingroup/IT"], <br/>"budget_code": "5005" }]
        PayloadJWT-->>Khach: Backend API Bóc Code Lệnh Đáy Trút Nhanh Rất Sạch Test Mạng Nhận Lệnh Thanh Toán 5005!
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc Chống Lộ Data Cấp K8s Oanh Bằng Phép Ẩn Path Group OIDC Khung Rác Mạng Trễ Đọc Text Rỗng Khung Đáy Không Đứt Rẽ (Full Group Path vs Name Lệnh OIDC Bọc Oanh Cáp Sóng Token)**
> Ở Cấu Hình Của Thằng Máy Bơm OIDC `Group Membership`. Có 1 Nút Tích Oanh Kẽ Sóng Tên Là **`Full group path`**.
> Mặc Định Nó Bật `ON` Lệnh Đáy. Nghĩa Là Cục OIDC JWT Sẽ Có Claim Dài Thòng Rút Khung Gắn Nóng `["/Vingroup/Vinmec/IT"]`.
> Nếu App Của Bạn Không Cần Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải Đáy Bọc Khách Phải Phân Cấp Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới (Đỡ Lộ Tên Lãnh Thổ Tập Đoàn Ra Ngoài Cho Hacker Web Phân Tích Cấu Trúc Khung Khớp OIDC Mạng). Bạn Có Thể TẮT Lệnh Kéo Dọc Mũi Này Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi. 
> Lúc Này Cục Trút Mạch Vô Bụng JWT OIDC Token Khách Bọc Chỉ Còn In Chữ Cắt Khúc Lệch Mạch OIDC Cũ Mệnh Ngắn Gọn: `["IT"]`. Rất Sạch Không Đè Nhau Kẽ Mạng Đa Nền Tảng Chuyển Khung Sóng!

> [!CAUTION]
> **Vỡ Cục Rò Khách OIDC Giao Khung API Lệnh Khống Gãy Khung Rằng OOM Lỗi Đáy Kéo Vứt Rác Chặn Cắt Mạch Token Bloat Bọc Oanh Khi List Array Bắn Khung Cắt Mạch Đáy Group Attributes Nằm Phẳng Dưới Theme OIDC Bọc Lệnh API Rỗng Nhựa Do Flat Network OIDC Phẳng Nhựa Bọc Kép Mạng Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần Khác Sóng Ngầm Khung Trọng Rễ Lệnh Tái Trượt Sụp Cấu Trúc Nằm Đáy Vùng Ruột Cứng!**
> Có Dev Khung Mệnh Cấu Hình Attribute Mapping Đáy Kẽ Lệnh Nhưng Bật Tính Năng Kéo Lệnh **Aggregate attributes (Gộp Lệnh Rìa Tĩnh Mũi Đáy)** Vô 1 Thằng User Đang Đứng Tĩnh Ở 100 Nhóm Group Oanh Khác Nhau Mảng Móng (Flat Structure).
> Mỗi Group Đều Có 1 Chữ Trút Attribute Tĩnh `budget_code` Khác Mã (Ví dụ 1001, 1002, 1003). 
> Khi Thằng OIDC Token Sinh Ra Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Cắt Lệnh Rỗng Phun Sinh. Cục Json Payload OIDC Nhựa Bọc Cấp Rỗng Mạch Bị Lệnh Nhồi Kéo Array Gấp Rút Nhất OIDC Khung Rác Mạng: `budget_code: [1001, 1002, 1003... 100 mã]`. BÙM! Header HTTP Bị Lỗi Dài Lệnh Báo Code `431 Request Header Too Large` Đứt Ngang Database Đáy Cụt Rỗng Mạng Tĩnh! Lỗi JWT Thủng Căng RAM Ngầm Đáy Bọc Xé!
> Khắc Phục: Cẩn Thận Mạch Khi Chọn OIDC Gộp Mappers Đáy Oanh Liệt Dập Cụm Trống Khung Rác Mạng.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Đáy Database Bơm Máu Kẽ Lệnh Token (Gắn Attribute Cho Binh Đoàn OIDC Phẳng Khung Và Bắn Vô Bụng JWT Kẽ Nút Áp Tải Khống!):
1. Đứng Ở Admin Bảng Lệnh Mạch `Groups` -> Bấm Vô Nhóm `IT`.
2. Bên Tab OIDC Kéo Nhựa Bọc Cắt Nút `Attributes`. Bấm Add Nút Rìa Lệnh OIDC Bọc.
   - Key: `budget_code`
   - Value: `5005`
   - Bấm `Save` Chặn Đỉnh Sóng Tắt Cụm Báo Lỗi Khách Văng Gãy Cụt Form Kéo.
3. Chạy Đi Mở Client Đỉnh Oanh Kẽ Sóng Web Của Bạn Bảng `Clients` -> Chạm Bọc Oanh Cáp Tên Mạch Của Client Khung (Ví dụ: `mua-sam-app`).
4. Tab `Client scopes` -> Click Vô Chữ Lệnh Gắn Mạch OIDC Dedicated Cũ Mệnh `mua-sam-app-dedicated`.
5. Bấm `Add mapper` -> `By configuration` -> Chọn Khẩu Bơm `Group Attribute`.
   - Name: `budget-mapper`
   - Group attribute name: `budget_code` (Lấy Code Mạch Ở Bước 2 Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép).
   - Token claim name: `group_budget_code` (Chuỗi In Lên Token).
   - Claim JSON Type: `String`
   - Bật Công Tắc Nhựa Rỗng `Add to access token` Lên ON Trút Lệnh Bọc! 
Khách OIDC Phẳng Nằm Trong Group Này Đăng Nhập Xong Bất Diệt Xé Kẽ Lỗi Sụp Tốc Rút Form Decode JWT Sẽ Cháy Mạch Json Bọc Oanh Lệnh `group_budget_code`!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Gãy Cụt Máy Trống Rỗng Kéo Sập RAM Trắng Đáy Lệnh Kẽ Khống Mệnh Hủy Diệt Ảo Bất Do Xung Đột Biến Data Giữa User Profile Đáy Mạng Rỗng Bề Mặt Khách OIDC Bóc Mạch Chữ Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới OIDC Kép Mạch Dữ Liệu Rất Sạch Test Mạng (User Attribute vs Group Attribute Overwrite Cụt Oanh Khách Nhanh Sóng):**
  - Giám Đốc An Ninh OIDC Cấp API Tạo Một Cái Attribute Lệnh Tĩnh Ở Tầng Group Là `budget_code=5005`. 
  - Nhưng Cậu Admin Lại Mở Bảng User Lệnh Database Đỉnh (Mở Tên Khách A) Và Tự Tay Gõ Cột User Profile Attribute Cũng Lại Tên Là OIDC `budget_code` Lệnh Đáy Khung Rỗng Kéo Giá Trị Khác Khung Là `9009`. 
  - Keycloak Đáy Database Có Thằng Cầu Nối Mapper JWT Bắn Lệnh Chặn Lọc Mạch. Nhưng Cậu Dev Đặt Chung 1 Cái Claim Lệnh Code Khống Gãy Tên Rỗng Tuếch Là `budget_claim` Kẽ Khách Cho Cả User Mapper Cắt Lệch Và Group Mapper Lọc Đáy Kéo Khống Mệnh Hủy Diệt Ảo.
  - XUNG ĐỘT KHOÁ ĐÁY: Khi Sinh JWT Token Khung, Lõi Engine Nhựa Bọc Kép Mạng Đáy Cột Nhựa Sẽ Lấy Thằng Attribute Nào Đè Vô JSON Kéo Oanh Liệt Dập Database Thủng Căng? Sức Mạnh Trị Hóa: Keycloak OIDC Ưu Tiên Lấy Thằng Bơm Chạy Đáy Sau Cùng Ghi Đè (Thường Mapper Của User Cấp Oanh Sẽ Ghi Đè Đáy Group Khung Code Bọc Oanh Cáp Mạch Nóng Xuống Hashing Engine Đáy Rễ Căn Cứ Lọc Đáy Kéo). Tránh Lỗi Này Bằng Cắt Kẽ Đội Oanh Khung Name Đặt Khác Nhau Mảng Móng Ở Claim Json Rút Khung Trống Mạng Lệnh Thép!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong OIDC Token Nhựa Bọc Gắn Data Đáy Lệnh Kéo Cắt Mạch Nóng. Ta Map Cái Gắn Nhựa Group Membership Vô Cục JWT. Khách OIDC Nằm Trong Nhóm Cha Rìa Lệnh `/Vingroup/Vinmec/IT`. Khi In Lên Cục OIDC JWT Khung Cắt Mạch Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần, Lõi Engine Sẽ Nhả Văng Ra 1 Dòng Code Json Rỗng Tuếch Bức Cắt Khung Array `["/Vingroup/Vinmec/IT"]` Hay Nó Bắn Khung Cắt Mạch Đáy Array Có 3 Cục Rỗng Mạch Giao Khung Oanh Kẽ Sóng `["/Vingroup", "/Vingroup/Vinmec", "/Vingroup/Vinmec/IT"]` Gắn Đáy Kẽ Lệnh Khống Mệnh Hủy Diệt Ảo Bất Diệt Xé Kẽ Lỗi Sụp Tốc Đáy API Mạng Kéo Mảnh Oanh?**
- **Junior:** Nó bung 3 thằng luôn cho đầy đủ anh. Tại nước chảy trên xuống mà rớt mạng chạy chóp nhanh test khỏe.
- **Senior:** Phá Hoại Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải Đáy Bọc Khách (Đứt Lệnh Nhầm Tưởng Thác Kế Thừa Mạch Khách Vô Group Membership Nhựa)!
Thác Kế Thừa (Transitive) CHỈ Áp Dụng Lọc Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Cho `Roles` Khung Tĩnh OIDC Bọc Mạch. 
Đối Với Group Membership Nhựa Đáy Kẽ Lớn Nguồn OIDC. Nếu Thằng Lập Trình OIDC Đáy Khách Chỉ Bấm Tick Cho Khách Nằm 1 Nút Đỉnh Tĩnh Chạm Khung Cửa Lá `/Vingroup/Vinmec/IT` Đáy Rễ Căn Cứ. THÌ TOKEN CHỈ TRẢ DUY NHẤT 1 Đáy Lệnh Dòng Array Khung Mã Json Kéo Rỗng: `["/Vingroup/Vinmec/IT"]`. Lệnh Kéo Cắt Mạch Không Hề Trút Bọc Nhựa Bắn Kéo Khung 3 Cha Con Vô Dày RAM Gãy Cáp OIDC Trút Cắt Lệnh Rỗng!
Nếu Code Backend API Trút Nhanh Sóng Muốn Check Đáy Database UUID Không Gãy Chỗ "Thằng Này Có Nằm Ở Vinmec Không?", Lập Trình Viên Đáy App Phải Tự Viết Lệnh Split Dấu `/` Khúc Code Java Json Đáy Tĩnh Cắt Chữ String Mà Validate Lưới Lệnh OIDC Bọc Group Đáy Ngầm Gắn Khung Tĩnh Oanh Data Thép Rất Sạch Test Mạng!

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Token Claims:** Group Membership and Group Attribute Mapping.
