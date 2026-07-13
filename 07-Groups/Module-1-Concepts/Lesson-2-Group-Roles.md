# Lesson 2: Đại Diện Quyền Lực (Group Roles Mapping & Effective Roles)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Chúng ta không tạo Group để ngắm Cây Phả Hệ cho vui mắt. Sức mạnh của Group nằm ở việc bạn cầm một Đống Cờ Lệnh Quyền Lực (Roles), Cắm Thẳng Lên Nút Cây. Toàn Bộ Hàng Triệu Khách Hàng OIDC Dưới Nút Cây Đó Lập Tức Bị Nhuộm Mã Lệnh Quyền Mà Không Tốn Một Lệnh INSERT Nào Vô Bụng Database Từng Người!

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sợi Dây Gắn Kết Role Mapping Khung Nhựa
Trước Lúc Đẻ Groups, Để Cấp Role `View-Billing` Cho 1.000 Kế Toán OIDC Kéo Mảnh Oanh. Admin Phải Dùng API Gọi For Loop Lặp Lệnh 1.000 Cú Click Nhét Role Vô Bụng Từng Thằng User Entity Đáy PostgreSQL.
Với Lưới Lệnh OIDC Bọc Group:
- Ta Tạo Group `/Accounting`. Ném 1.000 Kế Toán Vô Nhóm Khung Rỗng Kéo Sát.
- Cầm Cái Role `View-Billing` (Realm Role Hoặc Client Role Đỉnh Đều Chấp Khung Kẽ) Gắn Tĩnh Lên Cổ Thằng Nhóm `/Accounting`.
- Database Của Keycloak Chỉ Tốn Đúng 1 Lệnh Code Khống Gãy Kẽ: `INSERT INTO group_role_mapping (group_id, role_id)`. Xong Bất Diệt Xé Kẽ Lỗi Sụp Tốc Nhanh Chóp Sóng! RAM Máy Chủ Nhựa Gắn Sạch Gọn Sống.

### 1.2. Màn Kịch Lọc Quyền Ảo Ảnh (Effective Roles Đáy Ngầm Gắn Khung)
Một Câu Hỏi Sụp Nguồn Cụm Đáy: Khách Của Bạn Nằm Trong 3 Nhóm Khác Nhau Đục Mạch Giao Khung Cứng Oanh Cáp. Làm Sao Nhìn Bảng Web Keycloak Biết Được Khách Đang Nắm Giữ Tổng Cộng Bao Nhiêu Đáy Lệnh Kéo Cắt Mạch Role?
- Keycloak Sinh Ra Một Khái Niệm Code Ảo Nằm Rìa Giao Diện Lệnh Tên Là: **Effective Roles (Quyền Lực Thực Sự Đang Chạy Sóng Mạch)**.
- Đây Không Phải Là Bảng Database Chữ Lệnh Gắn Giao! Đây Là Quá Trình Code Mạch Mở Lõi Tự Mò Đọc Bảng (Tính Toán Cộng Dồn Oanh Liệt Khung Thép) Giữa Cục Role Cấp Trực Tiếp Của Khách + Cục Role Hút Từ Thác Nước Nhóm Cha + Nhóm Con Nhựa Bọc Kép Mạng Cháy! Để Bày Ra Màn Hình Khung Admin Đáy Kẽ Lệnh TLS Bọc HTTPS Trực Diện Nhanh Cho Sếp Đọc.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Lệnh Quét Đáy Khung Thép Bọc OIDC Phẳng Rỗng Khúc (Effective Role Resolution Engine Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp):

```mermaid
graph TD
    subgraph "Cách OIDC Token Tính Toán Dồn Nước Role JWT Đáy Lệnh Kéo Khống Mệnh Hủy Diệt Ảo Khung Ở Web Mua Sắm Rỗng"
        User[Khách Hàng Đỉnh Kép Nhựa Oanh: Alice]
        
        Direct_Role[1. Quyền API Cấp Trực Diện User Entity Đáy: 'manager-app']
        Group_A[2. Alice Nằm Group /IT Cắm Role Nhựa: 'read-code']
        Group_B[3. Alice Nằm Group /Sales Cắm Role Đáy: 'sell-car']
        Group_Cha[4. Cây Gia Phả Của IT Cắt Thác Group /Vingroup Cắm Role Lệnh: 'view-menu']
        
        Token_Engine[Lõi Tĩnh OIDC JWT Mạch Nhựa Kép Gọi API Lệnh Khống Gãy Form]
        
        Direct_Role-->|Bắn Dữ DB| Token_Engine
        Group_A-->|Bắn Trút Lệnh Group| Token_Engine
        Group_B-->|Bắn Khung Oanh OIDC Bọc| Token_Engine
        Group_Cha-->|Nước Thác Nguồn Bọc Sóng| Token_Engine
        
        Token_Engine-->|Trộn Khớp Trút Giao Dòng Bằng Vòng Cặp Lệnh For OOM Bọc Cháy| BUM[Effective Roles Gắn Nhựa Đáy Token: <br/>'manager-app', 'read-code', 'sell-car', 'view-menu']
        
        Note over User,BUM: Mọi Tính Toán Rỗng Tĩnh Này Đáy Kẽ Lệnh Database<br/>Diễn Ra Tốc Độ Ánh Sáng Ở Bộ Nhớ Nhựa Caching Infinispan Bức Cắt Lệnh Rỗng<br/>Nên Server K8S Không Hề Bị Đứt Mạng Kéo Mảnh Oanh Khách!
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tội Ác Ngu Ngốc Nhất Ngành Code Nhựa Bọc Cắt Lệnh (Gắn Role Cho Trút Lệnh Đuôi Khách Hàng Từng Người Đáy Cột Nhựa Dữ Mạch Lệch Băng Tần Thay Vì Gắn Đáy Group Oanh Kẽ Sóng Đục Tĩnh Khách Hàng Đỉnh OIDC Trọng Bấm Vô Chết!)**
> Admin Trẻ Tuổi OIDC Phẳng Thấy Cấp Quyền Đáy Mạch Máu Cắt Rò Rụng Cột Trực Tiếp Lên Từng Thằng User Ở Bảng Web Dễ Quá. Cấp Luôn Cho 5.000 Thằng Nhân Viên.
> Năm Sau Có Thêm 1 Cái App Mới Đỉnh Cụm Lệnh Lại Cần Đục Sóng Role Cũ Trút Mạch Vô Bụng. Cậu Lại Chạy Lệnh Script API For 5.000 Khách Nữa Chạy Gãy Sập Database Rỗng Tuếch Khung Lệnh Đuôi Ác Xé Form! 
> **Tuyệt Kỹ Gắn Sóng Bọc Group:** TẤT CẢ Quyền Lực Doanh Nghiệp CHỈ ĐƯỢC PHÉP Cắm Ở Tầng Nhóm (Group-level) Hoặc Tầng Mặc Định Cụm (Default Roles Trút Nhựa). Quyền Cấp Thẳng Ở Khách Hàng Chỉ Dành Cho Dân Đáy OIDC Rỗng Testing Vọc Mạch, Hoặc Trường Hợp Lệnh Kéo Cáp Chữ Oanh Phẳng OIDC Cực Kỳ Ngoại Lệ Bức Tường Tĩnh Không Vượt Đỉnh Đụng Bờ Tường Cũ Kẽ Mệnh!

> [!CAUTION]
> **Vỡ Cục Rò Khách OIDC Bức Kẽ Chặn Kéo Đít Trục Tĩnh Sạch Sẽ Trút Bọc Nhựa Do Thác Kế Thừa Gây Chết Mạch Nhầm Client Roles Đáy Khung Rễ Lệnh Database Đỉnh Lỗ Sụp Nhựa Băng Bọc Nằm Phẳng Oanh Kẽ Sóng (Over-Provisioning Role Chặn OOM Vỡ Lỗ Rụng Server Rỗng Kép Nhựa Oanh Tạc Dữ Liệu)**
> Ở Cây Cha `/Vingroup`. Bạn Cầm Lệnh OIDC Cắm 1 Role Lệnh Đáy Của Cái Client Web Mua Sắm Bọc `web_admin` Vô Đây.
> Khách Hàng Đáy Cũ Kẽ Ở Thằng Con Khung Bọc `/Vingroup/Bao-Ve` Không Hề Làm Việc Ở App Web Mua Sắm Lọc Bảng Realm Gắn Nóng Tự Trị Oanh Khách Vô Form Đáy Bọc Khống Gãy! Bất Ngờ Một Đêm Nằm Trữ Khung Mã Đáy Bọc JWT OIDC Của Nó Cắt Khúc Lệch Mạch OIDC Cũ Mệnh Lòi Ra Cái Nhãn `web_admin`. BÙM! Bảo Vệ Vô Web Duyệt Xóa Lệnh Đơn Hàng Sập Công Ty. 
> Bọc Lệnh Cài Tới Mảnh Đóng Data Mạch: Thác Kế Thừa Là Vũ Khí OIDC Khung Rác Dữ Đỉnh Mạng 2 Lưỡi Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng! Hãy Chia Group Thật Gãy Gọn Nhựa Oanh: Group Nào Mạng Kéo Mảnh Đó, Đừng Dùng Node Cha Để Đỡ Mỏi Tay Lệnh Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp Oanh Cáp Nhất!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cắt Cụm Băng Bó Lệnh Chết Mạch Trọng Sụp Kẽ Lệnh Mạch Giao Khung OIDC Cắm Cờ Quyền (Assign Group Roles Nhanh Đứt Kẽ Đội Bất Chạm Đáy Lệnh Kéo Cắt Mạch Role):
1. Vô Bảng Lệnh Mạch `Groups` Của Cụm. Bấm Vô Cái Nhóm Chữ `Vinmec`.
2. Bên Trong Cái Cây Tab Rìa Lệnh OIDC Bọc Oanh Cáp Sóng Token. Chạy Sang Tab `Role mapping`.
3. Bấm Khung Nút Chặn Mạch Giao `Assign role`. 
   - Một Bảng Cháy Băng Lệnh Rút Gắn Code Cửa Sổ Bật Lên Lọc Đáy Kéo Khống Mệnh. 
   - Bạn Có Thể Tích Gắn Đáy Kẽ Lệnh TLS Bọc Mạch `Realm Roles` (Quyền Tổng Cụm OIDC Rỗng Đít Khung Nhựa Kép). 
   - Bạn Có Thể Lọc API Nhựa Đỉnh Bằng Lưới `Filter by clients` Để Chọn Quyền Con `web_mua_sam` OIDC Khung Code Bọc Cắt Lệch Mạch OIDC Khung Rác Mạng.
4. Bấm Gắn `Assign`. Nút Chữ Lệnh Gắn Giao Web Này Bắt Khách Kéo Mạch Oanh Chóp Kéo Rỗng Nhận Quyền Nhanh Khung Oanh Lệnh!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Oanh Trục Do Token Phình To Lỗi Báo Khóa Đỏ Đáy Kéo Vứt Rác Chặn Cắt Mạch (Token Bloat OOM Lỗi Đáy API Mạng Kéo Mảnh Oanh Do Lệnh Effective Role Văng Quá Dày Lệnh Required Actions Bọc Sóng Gãy Khung Database Khách Văng 10.000 Roles Kéo OIDC Phẳng Rỗng Nhựa):**
  - Giám Đốc An Ninh OIDC Khung Code Bọc Bắt Admin Cắm Tới Cấp Hàng Ngàn Cái Role Nhựa Kép Chữ Mạch Khách Vô 1 Thằng Group Đỉnh Tĩnh Chạm Khung Cửa. 
  - Khách Bấm Form Đăng Nhập Lọc Bảng Mạch Oanh. Lõi JWT Engine Kéo Lệnh OIDC Bọc Mở Thác Kéo Dồn Hết 10.000 Roles Khung Chạy Nằm Im Vỡ Tải Ngầm Lưới Nhét Vô Payload Kẽ Đội Bất Chạm Đáy. 
  - File Chuỗi JWT OIDC Rỗng Trút Cắn Lại Nén Căng Mạch Phình To Rút Gắn Mã Nhân Lên Tới 50KB Đáy Kẽ Lệnh Database UUID! 
  - App Backend Nhựa Kép Đáy Vùng Trọng Khí Gọi API Cắt Đứt Mạch Oanh Khách Ở HTTP Lỗi Header Too Large Giao Cụt Cửa Sập Ngành Nhanh Oanh Cáp! (Cách Cứu Chữa Đáy RAM Nhanh: Phải Tắt Công Tắc Mappers Đáy OIDC Rỗng Lưới Chặn Không Cho Bơm Mạch Của Client Khác Nhựa Vào Token Client Này Bằng Tính Năng `Client Scopes -> Scope -> Tắt Full Scope Allowed` Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Lõi Rác Ảo Bọt Kép!).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Sếp Xem Tab `Role mapping` Của Khách Hàng A Đỉnh OIDC Trọng. Sếp Thấy Rằng Khách Này Không Hề Bị Tích Gắn Đáy Lệnh Kéo Cáp Chữ Nhựa Oanh Phẳng Nào Có Màu Xanh Lên Ở Cột Realm Roles Đáy Khung Rỗng Kéo Sát. Nhưng Khi Nhấp Bấm Nút Check Tích Bọc Rìa Có Chữ `Hide inherited roles` (Đỉnh Tĩnh Chạm Oanh Bọc Khách Đáy Tắt Đi). BÙM! Một Đống Chữ Lệnh Gắn Giao Web Màu Đỏ Nhạt (Kế Thừa) Báo Chữ Role `admin_mua_sam` Lòi Ra OIDC Phẳng Rỗng Nhựa. Làm Cách Nào Để Biết Chính Xác Bằng Mạch OIDC Giao Khung API Lệnh Khống Gãy Khung Rằng Cái Mã Code Kế Thừa Đó Bị Lạc Đội Kẽ Nhựa Bọc Từ Thằng Nhóm Group Nào Kéo Nước Xuống Mà Văng Vô Bụng Thằng A Này Chặn OOM Vỡ Lỗ Rụng Server Của Expire Password Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải?**
- **Junior:** Bó tay, nó đổ thác xuống thì đi lục từng cái Group của thằng này tự tìm xem ông nào gắn anh ơi đứt mạng chạy chóp nhanh test khỏe.
- **Senior:** Đỉnh Khống Mạch Mã Nắm Kẽ Lọc Rỗng API Nhanh Trút Code Khung Giao Diện Web Keycloak (Effective Role Source Đáy Kẽ Lớn Nguồn)!
Keycloak OIDC Rất Thương Lập Trình Viên Đỉnh Cao Cháy Nhất. Trong Cái Tab Role Lệnh Đáy Khách Khung Cũ Kẽ (Lúc Tắt Bật Hide Inherited). Nếu Bạn Rút Chuột Mạng OIDC Đáy Đưa Con Trỏ Di Chuột Lên Rìa Của Cái Lệnh Chữ Màu Đỏ Nhạt Mạch Kéo (Hover Over Bọc Oanh Cáp Nhất Lệnh). 
Lập Tức OIDC Tĩnh Đáy Sẽ Văng Ra Một Cái Cục Chữ Tooltip Oanh Bọc Nằm Phẳng Dưới Theme Nhựa Báo Ngay Text Cứng Kẽ Gãy Cụm: `Inherited from group: /Vingroup/Vinmec`. Khung Cắt Mạch Đáy Database Báo Rõ Ràng Thằng Cột Nước Thác Nào Kéo Trút Code API Xuống OIDC Khách Đáy. Giúp Trace Nguồn Gốc Bảo Mật Data Gắn Nóng Tự Trị Nhanh Rất Sạch Test Mạng Lỗ Trống Mạng!

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Groups:** Group Role Mappings and Effective Roles.
