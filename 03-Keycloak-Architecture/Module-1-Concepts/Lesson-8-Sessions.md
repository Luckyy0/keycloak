# Lesson 8: Quản lý Phiên (Sessions)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khám phá Dòng Chảy Thời Gian của Keycloak. Hiểu rõ Cấu trúc Bám Rễ của User Session, Sự Hủy diệt Hàng Loạt khi dọn rác, và Cứu cánh kỳ diệu mang tên: Phiên Ngoại Tuyến (Offline Sessions).

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. User Session và Lỗ Đen Kéo Mạng (The Spider Web)
Khi một User đăng nhập Thành công (Nhập Pass đúng). Keycloak Không Chỉ Vứt Cái JWT Ra Rồi Phủi Tay Đi Ngủ. Nó Mở Ra Một **Sổ Ghi Chép (Session)** Để Trói Chặt Định Mệnh Của Thằng User Với Cái Tòa Nhà (App) Mới Vào Xong Đó.
- **User Session (Phiên Toàn Cầu Nhện Mẹ):** Lưu lại thông tin "Cô Alice Vừa Login Vào Cổng Lúc 8h Sáng. Dùng Mạng Wifi Máy Macbook". Trụ Xương Sống Nằm Tại Bộ Nhớ Keycloak.
- **Client Session (Phiên Thuộc Địa Nhện Con):** Khi Cô Alice Xin Token Để Vào App Kế Toán. Từ Cái Nhện Mẹ, Keycloak Kéo Trổ Ra 1 Sợi Dây Nhện Con Gắn Vào App Kế Toán (Client Session). Nếu Lát Nữa Cô Đi Đòi Vào Cổng App Bệnh Viện, Nó Rút Thêm 1 Sợi Dây Đính Sang Mạng Cánh Bên Đó Kéo Sợi Thêm Chốt Bản Lề.

Tại Sao Phải Giăng Mạng Nhện Rườm Rà Nhớ Kỹ Lâu Như Vậy?
=> ĐỂ LÀM **SINGLE LOGOUT (ĐĂNG XUẤT HỦY DIỆT CHÉO)**.
Lúc 12h Trưa. Mạng Lệnh OIDC FrontChannel Dập Tới Mạng Báo "Cô Alice Yêu Cầu Đăng Xuất Toàn Bộ Khóa". Lõi Keycloak Chạy Thẳng Cổ Vô Rừng Sổ Ghi Nhớ, Truy Nhìn Thấy Nhện Mẹ (User Session) Bèn Xé Toạc, Rồi Dựa Theo Nhện Con Chạy Phóng Hỏa HTTP Gửi Băng POST Báo Án Phạt Gọn Lẹ Giết Chết Phiên Tại Đáy Thằng Máy Chủ Kế Toán Lẫn Cánh Mạng App Bệnh Viện Văng Hủy Ra Một Lượt Bất Thính Bàng Xuyên Suất Tuyệt Đối! Không Dư Vết Tàn Tích Bóng Ma Session.

### 1.2. Dọn Rác Giới Hạn CThời Gian Sinh Khí Của Token
- **SSO Session Idle:** Bấm Đồng Hồ Bấm Giờ Lúc Khách Ngủ Chết Bỏ Chuột (Ví Dụ: 30 Phút Bỏ Không Chạm Bàn Phím, Refresh Token Hết Nằm Khóc Đói. Session Bị Dọn Dẹp Không Trả Lại Vốn). Lời Khuyên Cứu Sinh App: Các Client Lâu Đời Có Trách Nhiệm Định Kỳ Xin Refresh Cắt Dây Đồng Hồ Về 0 Để Níu Giữ Lại Global.
- **SSO Session Max:** Bản Án Sinh Khí Sinh Mạng 1 Đời Phải Tử Vong Dù Rất Nỗ Lực Gắn Máy Thở (Ví Dụ: Bơm Oxy Tự Làm Mới Mở Suốt Trăm Lần Níu Dây Idle Liên Tục, Nhưng Tới Ngưỡng 10 Tiếng Session Max Đỉnh Rớt Chết Thẳng Giấc Ngủ Ngàn Thu Ép Xác Cửa Lôi Dậy Đăng Nhập Lại Yêu Cầu). 

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Bên Trong Cỗ Máy Của Vùng Nhớ Ram Infinispan - Bộ Lão Tà Giữ Session:

```mermaid
graph TD
    subgraph "Infinispan Cache Cluster (No SQL DB Đáy Phẳng Chứa Session Trọng Điểm)"
        S1[Máy Chủ KC 1]
        S2[Máy Chủ KC 2]
        S3[Máy Chủ KC 3]
        
        S1 -.->|Đồng Bộ Distributed Cache Phân Tán Mạng Phẳng Replicated Khớp Dữ Liệu Tức Khắc| S2
        S2 -.->|Replicated Mảng Dài Session ID Bơm Gói Đuôi JSON Băng Bỏ RAM| S3
        S3 -.->|Tưới Chéo| S1
        
        Client(Khách Đăng Nhập) -->|Request Rơi Váo Máy 1 Lần Đầu Lấy Kẹo Lưu Session Lõi| S1
        
        Client -->|Request Thứ 2 Bị Nginx Đuổi Tới Mạng Hầm Khác Máy 3 Gặp Cổng S3 Lạ Hoắc| S3
        
        Note over S1,S3: Máy 3 Hoàn Toàn Tự Đọc Được Bộ Nhớ RAM Infinispan Chéo Thấy Khách Này Đã Được Ghi Nhận Tại M2 Báo Cache Đuôi Về Rất Mạch Lạc, Khách Chơi Game Tự Do Chả Lo Nhớ Server Dính Sticky Cookie Rác.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Bi Kịch Treo Mạng Out Of Memory Do Nổ Rác Session Chậm**
> **Hiểm Họa Lập Trình Không Trục Xuất:** Khi Hệ Sinh Thái Dùng Tích Lũy Infinispan Bộ Nhớ Cache Lõm Dần Về RAM. Bạn Thiết Kế Tham Số `SSO Session Max` Dài Cỡ Cả Năm (365 Ngày), Còn Đuôi `Idle` Khoảng 6 Tháng Bỏ Quên Kệ Nó. Công Ty Có 50 Triệu Lượt Bóp Tạo Nick Khống Gọi Code (Chạy Cày Tool Automation). Infinispan Gồng Mình Chứa Hơn 50 Triệu Session Ghi Chép. Đứt Đai Mạch JVM Bộ Nhớ Xập Hết Ngành Cụm Sinh Sát Do Server Keycloak Cắn Gấp 3 Lần Lượng Khối Bộ Nhớ RAM Mặc Định Sinh Sát Cấu Hình Chạy Chứa Garbage Collection Cực Trọng Cắn Thủng Phản Lực CPU Tắt Đỏ Quạt.
> **Luật Kê Biên Ngắn Ngày (Short Living Defaults):** Max Sinh Chỉ Sống Cao Nhất 10-24 Giờ. Ép Các App Cần Background Xài Offline Token Kéo Xoay Trái, Tuyệt Đối Dọn Sạch Trí Nhớ Session Vắng Khách Hàng Ngay Lập Tức Dứt Khoát (Idle Khoảng Cỡ 30 Phút Cút Đi Rửa Sạch RAM Dưỡng Khí Tốt).

> [!CAUTION]
> **Vỏ Bọc Giao Thoa Lưu Database Vững Kéo Cánh (Offline Session Đâm Trúng Tường Bê Tông Đĩa SSD Lõi)**
> Offline Session Chứa Ngầm Yêu Cầu Chạy Cực Kỳ Hạng Nặng (Duy Trì Vĩnh Cửu Ngay Cả Lúc User Cúp Cầu Dao Tắt Vi Tính Đi Xa, Backend Có Quyền Dùng Offline Token Làm Đại Diện Hợp Pháp Liên Tục Vô Bờ Bến Đến Gọi Sang App Thứ 3 Rút Tiền Bơm Log Nhịp Cuối Cùng Rất Ảo).
> Nhưng Bản Chất Rất Ảo Mộng Khác Biệt: Keycloak Thông Minh Sẽ Đổ Nhẹ Dữ Liệu Offline Token Kéo Phẳng Rễ Xuống Mặt Đất Lõi Table PostgreSQL Bảng `OFFLINE_USER_SESSION` Khác Nhau Dữ Dội Không Ghi Sốc Bằng RAM (Vì Sợ Restart Máy Mất Điện Offline Bị Cắn Hủy Sống Gãy Mạch Làm Khách Lại Phải Mở Cổng Nhập Lại Login Tải Kẹt Bắt Khối App Bot Nhả Văng 401).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sức Mạnh Rứt Ráo Giải Phóng Kẻ Lộ Pass Bằng Thao Tác Chết (Revocation Policy Ngược Đường Khóa Gọn Lẹ):
Nếu Nghe Tin Báo Mạng Lộ Chùm Kéo 10 Vụ Bất Chợt Dò Pass Hàng Vạn Người: Đừng Lật Tìm Mở Trừng Pass Làm Gì. Keycloak Giữ Nốt Quyền Push Đòn 1 Trảm Sinh Sát Gọn Dẽ Từng Phân Hệ Dưới Realm:
- Chọn Mở Menu **Sessions**. Góc Khối Thấy Thẻ Đỏ Khung Rực Sống Nổi Bật Nút: **`Revocation`**.
- Bấm Ấn Định Ngày Giờ Ngay Lập Tức Tắt Còi Cột Mốc `Not Before` Múi Giờ Dừng Vạch Chặn (Đẩy Khung Hiện Tại Đè Lên). Toàn Bộ Các Session Sinh Khí Lúc Trở Về Trạng Thái Chết Kéo Toàn Vùng, Cả JWT Mặc Dù Ghi Vỏ Đọc Sạch Token Vẫn Thấy Đẹp Đẽ Valid Kéo Signature. NHƯNG Khi Móc Xác Thực Bằng Đầu Keycloak Introspection Sẽ Ói Ngược Trả Lại Invalid Liệt Vào Hắc Danh Sách Lập Tức (Bắn Phát Súng Lên Không Gửi Thông Báo Kéo Ánh Sáng Quét Cắt Đuôi Revocation Broadcast 4 Ngã Chết Tụ Sập Cổng Toàn Mảng Người Cũ Bắt Login Hết Lại Ráo Bắn Không Thương Đau).

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Mở Khóa Sự Kiện Đau Lòng Token Trôi Dạt Vô Hình (Stateless Access Token Sinh Đời Xa Session):**
  - Giám Đốc Rút Revoke Cắt Hủy User Session Trong RAM Đứt 1 Cách Ngoạn Mục.
  - Nhưng Ở Đời Thực. Nếu Anh Nhân Viên Cầm Được Cục Độc Ác JWT (Mới Sinh Phút Trước Có Access Lõi 15 Phút Sống Dài). Lệnh Đuổi Session Keycloak KÉO CẮT TẠI CỤM MÁY CHỦ BẦU TRỜI. Cục Nhỏ Cầm API Vẫn Tự Đập Xác Nhận Check Chữ Ký Nằm Tại Máy API Gateway Lõm Dưới Đáy Backend Không Chọc Sóng Lên Trời Hỏi Thử. API Gateway Bảo Khớp Chữ Ký! Cấp Quyền Đọc Lấy Log Data Ngân Hàng Kéo Về. Án Mạng Diễn Ra Cục Bộ 15 Phút Sau Trái Phiếu Đợi Đuôi Timeout Hết Khí 15 Phút JWT Mới Tự Rữa Nát Rớt (Lỗ hổng Chết Gãy Stateless Cách Ly Phân Tầng Sinh Tử Dựa Vào Tốc Độ Thời Gian Dịch Suy Đoán Lõm Kẽ).

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Khái Niệm Phẳng Định "Sticky Sessions" (Cookies Nhốt Máy Chủ Khác) Tại Các Nginx API Gateway Reverse Proxy Có Cần Gắn Ép Trọng Mạch Máu Của Mạng Lưới Keycloak Server Chống Rớt Token Không? Tại Sao Lại Thiết Kế Đòn Bọc Gắn Cookie Route Đó?**
- **Junior:** Bắt buộc gắn không rớt mạng đăng nhập. Khách đang ở máy 1 bị đá qua máy 2 là bắt login lại do nó không biết.
- **Senior:** Đó Là Câu Truyện Truyền Miệng Đời Cổ Trái Hệ Phân Tán Đa Kênh Lỗi Thời! Của Tomcat Hồi Xưa. Lõi Keycloak (Đặc Biệt Kỷ Nguyên Infinispan Cache Độc Thần Phẳng RAM Distributed) Tuyên Thệ Tự Động Rửa Trái Không Ép Buộc Load Balancer Cứng Cấu Cáp Rễ (No Sticky Sessions Required).
Bạn Có Thể Rải 1 Triệu Khách Vào Round-Robin Ngẫu Nhiên Hết Mọi Node Của Keycloak Ném Liên Tục Nảy Bóng. Request 1 Bám Nảy Vô Đít Server 1 (Lưu Lõi Login Ra Data Infinispan Đồng Bộ Bay Bám Thẳng Mảng S2 S3). Lần Login 2 Đuôi Xác Nhận Bị Nginx Đẩy Văng Qua Máy 3 Nằm Xa Tít Mạng Bên Kia Phân Khúc Băng Tần, Máy 3 Có Bộ Móc Rút Cache Đồng Bộ Bằng 0ms Kéo Sóng Ra Lôi Trọng Lịch Sử User S1 S2 Y Cũ Không Một Dấu Hiện Sượng Nào (Vừa Tối Đa Tái Cấp Không Phụ Thuộc Rủi Ro Rớt Cụm Mất Sticky). Bắn Node Chết Hẳn Lên Mây Cũng Không Cháy Session Mất Gắn Hạt.

**2. Keycloak Cache Tầng Đâu Phân Ngắn Được Bằng Công Nghệ Phân Vùng Lấy Đẩy Local Dạng Gì Để Né Áp Lực Tốn Băng Thông LAN Bắn Broadcast Liên Tục Mỗi Nửa Giây Của Infinispan Khi Dọn Phế Liệu Rác Idle Tụt Lạc Bơm Đẩy Phản Kích Xoay Chiều Network?**
- **Junior:** Cứ bắn broadcast ra các máy thì mạng mạnh lo. Chậm tí không sao.
- **Senior:** Đòn Đánh Quyết Định Nằm Ở Khái Niệm **Cache Owners (Cụm Chủ Nhận Phân Trích - Phân Vùng Dữ Liệu Distributed Cache Mode So Với Replicated Cache Đầy Mạng Rễ Khống Nhịp Chậm Băng).** 
Nếu Replicated (Copy Mọi Thứ Toàn Bộ Vào Từng Con Node), Keycloak Nổ Băng Thông Rơi Vào Bẫy Rác Mạng. Cứ Sinh JWT Phải Đóng Lệnh Chờ 10 Con Cụm Trả 10 Chữ OK Chậm Điên Tiết CPU Tắc Văng Lõi.
Keycloak Lên Distributed Owners (Ví dụ Cấu Hình Owner = 2). Lõi Phân Khúc Áp Data Ngẫu Nhiên Bằng Hashing Lõi Consistent Hash Dính Chặt 2 Máy Chủ Giữ Data Đứng Tên Nhện (Máy A Chủ, Máy B Phụ Cứu Chết Tự Bật Lên), Mấy Trăm Con Cụm Server Khác Thủng Hẳn Rỗng Bụng Sạch Bông RAM Không Ôm Chứa Gì Sất Về Nó Tránh Cắn Rỗng Trí Nhớ Giữ Của. Bọn Kia Ai Cần Đọc Thong Tin Đành Gọi Điểm Đến Máy A Rút Kéo Lên Trả Dữ Về Lệ Phí Băng Thông Xoay Chóp Ngọt Lẹ Vô Biên Cứu Tải (Đẳng Cấp Kỹ Thuật Distributed Caching).

**3. Offline Session Token (Đã Khai Báo Trọn Vẹn) Của Keycloak Có Trượt Khí Ngừng Thở Và Đổ Rác Sau Vài Tháng Không? Hay Trái Gắn Đời Nó Lưu Danh Chết Đống Trăm Năm Tại CSDL Postgres Sáng Đêm Gặm Mòn Phân Cắt Sụp Bề Bảng Data Nếu Khách Hàng Bỏ Đi Ngàn Năm?**
- **Junior:** Sống ngàn năm vĩnh viễn luôn. Thơm Bền Lâu.
- **Senior:** Vĩnh Cửu Cũng Nằm Ở Khung Thời Gian Giới Hạn Quét Đóng Vùng (Default Trị Dữ Liệu Cắt Máu OIDC Giữ Lửa Chống Cháy Ác Mộng Storage Bloat).
Có 1 Nút Thiết Lập Sâu Trong Realm Settings Giới Hạn Đỉnh Gọi Là: **Offline Session Idle**. Chứ Nó Không Vô Địch 100%. 
Luật Quy Đình Mặc Định: Cục Offline Này Tự Ngủ Êm 30 Ngày (Không Dùng Kéo Code Chặn Nhịp Gọi Gọi Refresh). Thì Khi Khách Rơi Khoảng Hẫng Timeout Đó Bảng PostgreSQL Quét Chổi Scheduled Task Cuối Đáy Quét Thấy Dọn Bay Gãy Dòng Data Của Thằng Này Ra Rác Rác Bãi Chôn Đồ Cũ. Cắt Ảo Mộng Token Vĩnh Cửu Sinh Tồn Ế Lạc Chặn Sụp Ổ Cứng Database Mở Rộng Ế Phình Kéo Ngàn Giga Trả Khí Lưu Thông Lành. (Đúng Nghĩa Hệ Thống Trị Sự Lõi Doanh Nghiệp 0 Bảo Trì Dài Lâu Chống Nổ Bom Time).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Infinispan Caching:** Distributed Setup Guides and Clustering.
- **OIDC Offline Access:** The `offline_access` Scope Usage in IAM.
