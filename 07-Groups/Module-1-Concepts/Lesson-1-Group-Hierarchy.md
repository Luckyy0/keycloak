# Lesson 1: Cây Gia Phả (Group Hierarchy & Luồng Kế Thừa)

> [!NOTE]
> **Category:** Theory & Practice (Lý thuyết & Thực hành)
> **Goal:** Trong Kiến trúc Tập đoàn, Không Bao Giờ Có Các Nhóm Nằm Ngang Hàng Phẳng Lặng. Chúng Nằm Đè Lên Nhau Theo Cây Gia Phả (Cha - Con). Nếu Mạng OIDC Của Bạn Không Được Tổ Chức Dạng Cây, Quản Trị Viên Sẽ Bị Nhấn Chìm Trong Bãi Rác 10.000 Nhóm Không Ai Nhớ Tên!

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Cấu Trúc Bức Tường Ngăn Mảng (Group Trees)
Keycloak Xây Dựng Khung Groups Dưới Đáy Mạch Hoàn Toàn Theo Dạng Directory Tree (Giống Hệt Thư Mục Nằm Trên Hệ Điều Hành Linux).
- Bạn Tạo 1 Nhóm Gốc Cụm Là `/Vingroup`.
- Bên Dưới Bạn Bấm Trút Nút Sinh Nhóm Con `/Vingroup/Vinfast` và `/Vingroup/Vinmec`.
- Dưới Vinmec Lại Cắt Rễ Đẻ Đáy Nhựa `/Vingroup/Vinmec/IT`.
Người Dùng Của OIDC Có Thể Được Ném Vô BẤT KỲ ĐỐT NHÁNH NÀO Của Cây Này Đáy Kẽ. Khách Vô Đốt Root Cũng Được, Khách Rớt Đáy Khung IT Cũng Được Trút Mạch Vô Cùng Cứng Khung.

### 1.2. Thác Nước Quyền Lực (Transitive Inheritance)
Khi Bạn Xây Một Cây Thư Mục. Cái Trọng Tải Ghê Gớm Nhất Của Nó Là Cột Sóng Truyền Nghề Của Cha Cho Con!
- Khái Niệm OIDC **Kế Thừa Quyền Lực Lan Truyền**: Bất Kỳ Một Đặc Điểm Quyền Hạn (Role) Nào Của Thằng Cha `/Vingroup` Nắm. Thì MẶC ĐỊNH Mạng OIDC Lõi Sẽ Đục Nước Ép Chảy Thẳng Xuống Phễu Lệnh Cho TẤT CẢ Các Nhóm Con Và Các Thằng User Nằm Trong Nhóm Con Đó Nhận Được Kép Lệnh Cũ Kẽ!
- Chiều Rút Nước Ngược Lại KHÔNG BAO GIỜ XẢY RA: Quyền Của Thằng Con `/Vinmec` Chỉ Nằm Tĩnh Đáy Vùng Ruột Của Nó, Thằng Cha Không Thể Hút Trút Data Ngược Lên Bề Bắn Health Đỏ!

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Bắn Dòng Nước Kéo Khung Tốc Độ Không Phân Gãy Tải Lên Xuyên Nhựa Khi Cắt Đứt Đáy Mạch Oanh Khách (Identity Brokering Group Engine):

```mermaid
graph TD
    subgraph "Thác Nước Kế Thừa Lan Truyền (Downward Transitive Flow Bọc Cấp K8s)"
        TapDoan[Nhóm: /Vingroup <br/>(Sếp Lớn Bọc Quyền Role: 'tap-doan-view')]
        CongTy[Nhóm: /Vingroup/Vinmec <br/>(Có Quyền Role Riêng: 'vinmec-admin')]
        PhongBan[Nhóm: /Vingroup/Vinmec/IT <br/>(Không Code Lệnh Nào Đáy)]
        
        NhanVien[Khách: A_Nguyen <br/>(Chỉ Nằm Duy Nhất Ở Nhóm /.../IT Mạch)]
        
        TapDoan-->|Nước Chảy Kéo Quyền Cha 'tap-doan-view' Bọc Sóng| CongTy
        CongTy-->|Gộp Quyền Cha + Quyền Mình 'vinmec-admin' Dội Thác Cắt| PhongBan
        PhongBan-->|Mang Toàn Bộ 2 Trọng Tải Bọc Đáy Đổ Áp Lên Đầu| NhanVien
        
        Note over NhanVien,PhongBan: Khi Sinh Cục OIDC JWT Token Kẽ.<br/>Lõi Keycloak Tự Động Rút Mạch Mở Giao Đít Cấp Cho Khách A_Nguyen<br/>Hai Cục Role Khổng Lồ Từ Trên Thác Dội Xuống!
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Gắn Lệnh Cầm Mạng Group (Nguy Hiểm Vỡ Cục Dữ Liệu Chặn OOM Gây Cắt Đứt Lệnh Kéo Dọc Mũi Bằng Việc Cấp Quyền Rác Tại Nốt Gốc Cây - Root Node Mạch Sóng Đục Tĩnh)**
> **Tội Ác Ngu Ngốc Nhất Ngành Quản Trị Directory OIDC Lệnh Báo Code:** Anh DevOPS Nghĩ Đơn Giản, Group Gốc (Root) Bọc Lệnh Cài Tới Mảnh Đóng Là Tập Đoàn. Nên Anh Quăng Vô Đáy Khung Thép 1 Cái Code Quyền Siêu Khủng `Super-Admin-App-Mua-Hang`.
> Hậu Quả Ác Tuyệt Cắt Lệnh: Bởi Vì Thác Nước OIDC Chảy Trút Lệnh Đuôi Từ Trên Root Xuống. MỘT TRIỆU THẰNG Tạp Vụ Nằm Ở Các Nhóm Con `/Vingroup/Ve-Sinh`, Nhóm `/Vingroup/Bao-Ve` KHÔNG ĐỤNG ĐÁY DATA CŨNG TỰ ĐỘNG BỊ NHIỄM HƯỞNG BỌC OANH CODE QUYỀN TRỌNG `Super-Admin` Kéo Cáp Mạch Máu Cắt Lệnh! Sập Công Ty Cụt Đuôi Mạng Thủng Rác!
> **Luật Thép Sống:** Ở Các Node Đầu Nhánh Oanh Kẽ Sóng Gốc (Root). CHỈ CẤP Các Quyền Cắt Cụm Đọc View Basic Bọc Oanh Cáp Nhất Có Thể! Càng Xuống Sâu Rễ Tĩnh Khung (Lá Cây), Quyền Lực Mới Càng Được Nới Nhựa Kẹp Bóp Nóng Mạch!

> [!CAUTION]
> **Nỗi Lòng Đứt Form Sập App Bằng Bảng Lệnh Mạch Cứng Do Cấu Trúc Vỡ Lõi Bể Xé OIDC Tách Trùng Lặp Nhóm Nằm Ngang (Flat Structure Cắt Khúc Lệch Cột Kéo Lệnh Json Array Đáy Database UUID Không Gãy Chỗ Trọng Lệnh Đơn Giản Kéo Cáp OIDC Kẽ Nút Áp Tải Khống!)**
> Rất Nhiều Dev Bê Nguyên Mạch Khái Niệm Tự Kéo Bọc Của DB SQL Trút Vô Kẽ Khung OIDC Oanh Lệnh. Họ Không Xài Cây Mạng. Họ Tạo 1 Đống Nhóm Nằm Ngang Bằng Nhau Rìa Lệnh: `Group_Vingroup`, `Group_Vinmec`, `Group_IT`.
> Sau Đó Đứa Nào Vô Công Ty Là Họ Bấm Tick Chữ Mạch Khách Vô Cả 3 Cái Nhóm Này Cùng Lúc.
> Sức Mạnh Trị Hóa Mạch Rỗng Cấu Tĩnh: Màn Hình Admin Keycloak Không Có Code Để Lọc Mạng Mảng Flat Oanh Khách Này. Khi Khách Bị Tick Quá Nhiều Nơi Bằng Lệnh Phẳng. Việc Bạn Thu Hồi OIDC Trút Nhanh Lệnh Khống Ép Trọng Mạch Báo Lỗi Khách Trở Nên Bất Khả Thi (Lỡ Quên Gỡ Tích Của Nhóm Này Đứt Nhóm Kia). Tận Dụng Tuyệt Đối Code Cây Hierarchy Để Rút Gọn Cú Click Chuột Bọc OIDC Phẳng Nhựa!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Cây Tổ Chức Vingroup Bằng Lệnh Kéo Dọc Mũi Rỗng Đít Khung Tĩnh OIDC Bọc Oanh Cáp Sóng (Giao Diện Admin Đáy Tĩnh Khống API Lỗ Đục Rò Nhầm Lệ Lặp):
1. Vô Bảng Lệnh Mạch `Groups` Của Cụm Đỉnh Tĩnh Chạm.
2. Bấm Nút Tạo Trút Mạng Kéo `Create group`. Tên Lệnh Bọc `Vingroup`. Bấm `Save`.
3. BÙM, Giờ Nhìn Ra Bảng Bạn Sẽ Thấy Cái Nút Mạch Cây Oanh Liệt Dập Khung Có Tên `Vingroup`. Nhấn Vô Tên Lệnh Kéo Cáp Đáy Đó!
4. Bên Trong Bụng Nó Đang Trắng Rỗng Mạng Kéo Mảnh Oanh. Có Cái Nút Nằm Mạch Kéo `Create child group` (Tạo Thằng Con Đáy Ngầm Gắn Khung Tĩnh). Bấm Bọc Nút. 
5. Tên Lệnh Bọc Cấp Rỗng `Vinmec`. Bấm `Save`.
Giờ Cây Thác Nước Của Bạn Đã Tồn Tại Mạch Lưới Lệch Băng Tần Khác Sóng: `/Vingroup/Vinmec`. Rất Sạch Test Mạng!

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mạch Giao OIDC Giết Form Lạc Lệnh Kép Gãy Cụt Máy Trống Rỗng Kéo Sập RAM Trắng Đáy Kẽ Lệnh Database Đỉnh Do Vòng Lặp Xóa Node Root Sụp Nguồn Cụm Đáy Kéo Khách Khung Rỗng Vành Chặn Đỉnh Sóng Tắt Cụm Báo Lỗi (Cascade Delete Group OOM Vỡ Lỗ Rụng Server Của API Trút Mệnh Khung Áp Phẳng Nằm Im Vỡ Tải Ngầm Lưới):**
  - Giám Đốc An Ninh Bực Bội, Vô Admin Bấm Nút Delete Cụt Thằng Node Gốc Cha OIDC Rỗng `/Vingroup`. 
  - Trong Hệ Nén Chạy Data Đáy Lệnh PostgreSQL Cắt Đứt Khách. Keycloak Cấu Cắt Chữ Bức Tường Xóa Chữ Mạch Khách Cha Kéo Luôn Thằng Con Rụng Đứt Kẽ Đội Bất Chạm (Cascade Delete).
  - BÙM! Trục OIDC Token Lọc Oanh Liệt Dập Database Thủng Căng Lệnh Lỗ Trống Mạng. Toàn Bộ 10.000 Khách Hàng Đang Nằm Trữ Trong Các Nút IT Của Thằng Cha Đáy Kẽ Lệnh Đã Bị Văng Gãy Cụt Mạch Mất Sạch Dấu Nhóm! Không Ai Có Quyền Đăng Nhập Nữa. JWT Xin Token Báo Rỗng Tuếch Khung Lệnh Đuôi (Mất Sạch Roles Kế Thừa).
  - Trị Hóa Mạch Rỗng: Trước Khi Chạm Nút Delete Đáy Mạch Node Gốc, Cấu Trúc Khung Rẽ Buộc Phải Chuyển Oanh Các Nhánh Con Kéo Mảnh Oanh Sang 1 Thằng Cha Khác (Move Node Khung Tĩnh OIDC Bọc) Cắt Lệnh Rỗng Phun Sinh Data Trọng Lệnh Đơn Database UUID Không Gãy Chỗ Tránh Sập OOM!

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Realm Khách Hàng Nắm Cổng. Nếu Ta Cho Một Khách Hàng Nằm Vào Thằng Con `/Vingroup/Vinmec/IT`. Và Cấp Cho Thằng Khách Đó 1 Cái Role Riêng Tư Lệnh Đáy Database (User-level Role) Của Khung Mạch OIDC Là `quan-ly-server`. Lệnh Trút Nhựa Áp Phẳng Này Có Chảy Ngược Thác Lên Cho Thằng Nhóm Của Nó Nắm (Tức Là Bọn Khác Nằm Trong IT Cũng Hưởng Kép Lệnh Oanh Không Mạch Kẽ)?**
- **Junior:** Nó nằm trong nhóm thì nhóm nó chắc cũng được ké chung anh. 
- **Senior:** Phá Hoại Đáy Mạch Máu Cắt Rò Rụng Cột Network Lệnh Tải (Dòng Chảy OIDC Chỉ Đi 1 Chiều Đáy Rễ Căn Cứ)!
Nguyên Tắc Bức Tường Lưới Mạng Của OIDC Kéo Khống Mệnh Hủy Diệt Ảo: Role Cấp Riêng Cho Thằng User Nào Thì MÃI MÃI Đóng Đinh Ở User Đó Đáy Bọc Khách. Việc Khách Nằm Trong Group Kéo Cáp Mạch Nóng Chỉ Lệnh Cho Khách **Hút Nước** Quyền Lực Từ Nhóm Vô Bụng Mình (Inherit Đáy Kẽ Lớn Nguồn). Khách Không Bao Giờ **Bơm Nước** Quyền Lực Cá Nhân Ngược Lại Cho Cái Bồn Của Nhóm Mạch Lưới Lệch Băng Tần Đáy OIDC Rỗng.
(Luật Nước Chảy Chỗ Trũng Oanh Liệt Dập Database Thủng Căng: Root -> Branch -> Leaf -> User Khung Chạm Sóng Đỉnh!).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Server Administration Guide:** Group Management and Hierarchy.
