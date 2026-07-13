# Lesson 10: Kiến trúc Zero Trust (Không Tin Tưởng Bất Kỳ Ai)

> [!NOTE]
> **Category:** Theory & Architecture (Lý thuyết & Kiến trúc)
> **Goal:** Xóa bỏ Trọng bệnh lớn nhất của Cấu trúc Mạng Doanh Nghiệp: "Tường lửa (Firewall) bảo vệ tất cả". Nắm bắt Triết lý Tối thượng của Kỷ nguyên Mới: "Luôn luôn Xác Minh, Không Bao Giờ Tin Tưởng - Never Trust, Always Verify".

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Sự Sụp Đổ Của Tòa Lâu Đài (The Castle-and-Moat Ruin)
Ngày xưa, Hệ thống mạng như 1 Tòa Lâu Đài. Bên ngoài (Internet) là Rừng Rậm đầy giặc. IT xây một Bức Tường Lửa (VPN / Tường Lửa Mạng) cực dày. Ai ở Ngoài Mạng là Kẻ Thù. Ai ĐÃ VÀO TRONG MẠNG (Cắm dây mạng LAN của Công ty) thì ĐƯỢC TIN TƯỞNG 100%. Mọi Máy chủ bên trong thoải mái chọc ngoáy nhau.
**Thảm họa xảy ra:** Năm 2013, siêu thị Target (Mỹ) bị Hacker đánh cắp 40 Triệu thẻ tín dụng. Hacker KHÔNG HACK TƯỜNG LỬA NGÂN HÀNG. Hắn Hack cái Hệ Thống Sưởi Điều Hòa (HVAC) ở ngoài. Do hệ thống Điều Hòa có Dây Mạng nối chung vào Mạng Nội Bộ (Vùng Tin Tưởng), từ Điều Hòa, Hacker nhảy sang Máy Cà Thẻ (PoS) và lấy trộm mọi thứ. (Đòn Lateral Movement - Tấn công ngang). Tường lửa Vô Dụng khi Giặc Đã Ở Trong Nhà.

### 1.2. Chân Lý Zero Trust
Kiến trúc Zero Trust tuyên bố: **Biên giới Mạng Mẽo (Network Perimeter) ĐÃ CHẾT.**
- Bất kể Mày đến từ IP ở Mỹ, hay Mày đang Cắm Dây Mạng ở ngay phòng Giám Đốc. TAO KHÔNG TIN MÀY.
- Mọi Yêu Cầu (Mọi Request API), dù là từ Trình duyệt gửi lên, hay là Từ Máy Chủ Kế toán gửi sang Máy Chủ Database, ĐỀU PHẢI ĐƯỢC XÁC THỰC VÀ KIỂM TRA QUYỀN. 
- Mạng LAN Nội bộ giờ đây Phải Bị Coi là Môi trường Độc Hại ngang với Public Internet.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Tầm quan trọng của CARTA (Continuous Adaptive Risk and Trust Assessment) trong Zero Trust:

```mermaid
graph TD
    subgraph "Mô Hình Cũ: Tin Tưởng Cố Định (Tĩnh)"
        A[Login Sáng Nay Ok] --> B[Token sống 8 Tiếng]
        B --> C[Trong 8 Tiếng, bị nhiễm Virus]
        C -->|Vẫn gọi được API bình thường do Token còn hạn| D[Dữ liệu bị trộm]
    end
    
    subgraph "Mô Hình Mới: Đánh Giá Liên Tục (Zero Trust)"
        E[Login Ok] --> F[Gọi API Phút thứ 5]
        F --> G{Đánh giá bối cảnh (Context)}
        G -->|Đang quét thấy 1 tiến trình Lạ (Virus)| H[HỦY DUYỆT TOKEN NGAY LẬP TỨC]
        G -->|Bình thường| I[Cho Phép Vào]
        Note over E,I: Quyền truy cập không phải cấp 1 lần là xong.<br/>Nó được Kiểm tra Lại Liên Tục ở MỌI Giao dịch.
    end
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Step-up Authentication (Tái xác thực Leo thang)**
> Cột sống của Zero Trust. Khi Kế toán ngồi mở Báo cáo tài chính (Quyền thấp), dùng Mật Khẩu là Đủ (Trust Level 1).
> Đột nhiên Kế Toán bấm nút "CHUYỂN KHOẢN 10 TỶ". Zero Trust lập tức Đánh Giá: Rủi ro Quá Cao. Nó Cắt Ngang Tiến Trình Bằng Cách ĐÁ KẾ TOÁN SANG KEYCLOAK, Bắt buộc cắm YubiKey / Quét Mống Mắt (Nâng cấp lên Trust Level 3). Xong mới cho Chuyển khoản. 
> Việc này đảm bảo kể cả Hacker cướp được Cookie Session (Trộm Session) cũng không thể Hủy diệt Công ty vì bị Kẹt ở Vòng Tái Xác Thực.

> [!CAUTION]
> **Micro-Segmentation (Chia Vách Ngăn Phân Tử)**
> Trong mạng nội bộ K8s, Không được phép để App A chọc Data Bằng HTTP trần (Unauthenticated) sang App B.
> **Luật Sinh Tử:** Giữa Server (Service-to-Service) cũng BẮT BUỘC dùng Mutual TLS (mTLS) và Trao đổi Token OIDC (Client Credentials Grant). Phải ép thằng Server A đi xin Keycloak cái Token, rồi cầm Token đó đưa cho Server B. Nếu Server B bị dính Đòn XSS, nó cũng không mò sang đập Server A được vì Không Có Token Hợp Lệ.

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Keycloak triển khai Zero Trust CARTA thông qua **ACR (Authentication Context Class Reference)**:

Trong OIDC, khi App Kế Toán muốn Khách hàng nâng cấp Cấp độ Xác Thực. Nó sẽ Gọi hàm Login sang Keycloak truyền thêm biến:
`https://keycloak.../auth?claims={"id_token":{"acr":{"essential":true,"value":"Level3_Fingerprint"}}}`

Keycloak đọc tham số này, nó Lục lại Hồ sơ của Session hiện tại:
*"À, Hồi sáng thằng cha này Login mới ở Level 1 (Password). Bây giờ App yêu cầu Level 3. Dừng lại! Bật Form Quét Vân Tay Lên!"*.
Sau khi quét Vân Tay xong, Keycloak Nâng Cấp Cái JWT Token lên Thành Bậc `acr: Level3_Fingerprint` và trả về. App Kế Toán đọc được cái Token Bậc 3 đó mới cho phép Chuyển Tiền.

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Cơn Trầm Cảm của Hệ thống Kế Thừa (Legacy Systems):**
  - Giám Đốc muốn áp dụng Zero Trust toàn Tập đoàn. Nhưng Kho dữ liệu Mainframe IBM COBOL mua từ năm 1990 TỪ CHỐI hiểu JWT Token là gì. Nó chỉ chấp nhận User/Pass tĩnh.
  - **Khắc phục:** Mẫu thiết kế Đại Sứ (Sidecar Proxy Pattern). Ta Đặt 1 con Envoy Proxy (Cửa hải quan) Đứng Ngay Sát Nách Con Máy Tính Cổ Đại (Cùng mạng Localhost). Mọi Request từ thế giới ngoài đập vào Envoy. Envoy tự Móc Cổ Token ra Verify. Nếu Hợp lệ, Envoy Mới mớm Mật Khẩu Tĩnh vào cho con Mainframe. Cứu rỗi được Zero Trust mà Không cần Sửa Code Mainframe.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Zero Trust, tại sao người ta nói "Identity is the New Perimeter" (Định danh là Biên Giới Mới)? Tường lửa mạng IP Table đã hết thời rồi sao?**
- **Junior:** Tường lửa vẫn xài, Identity thêm vô cho vui.
- **Senior:** Tường lửa Mạng (Dựa trên IP/Port) hoàn toàn Bất Lực trước Môi trường Cloud Phân Tán (Tức là Nhân viên Mở Laptop Ở Quán Cafe, dùng Mạng 5G, IP đổi liên tục mỗi 5 phút).
Bạn không thể cấu hình Tường lửa "Chỉ cho phép IP quán Cafe truy cập Database".
Do đó, Biên Giới Bảo Vệ Cột Mốc Bắt Buộc phải Chuyển dịch từ Tầng IP (Network Layer 3) lên Tầng Định Danh (Application Layer 7 - Identity). Bất kể Bạn ở Quán Cafe hay Mặt Trăng (IP gì cũng được). Miễn là Cái Token Identity của bạn ĐÚNG, và Tình Trạng Thiết Bị Của Bạn Đang KHÔNG NHIỄM VIRUS (Device Trust). Thì bạn được vào. Identity chính là Vạn Lý Trường Thành mới.

**2. Làm thế nào một Trình duyệt hoặc App Điện Thoại có thể gửi "Trạng Thái Nhiễm Virus Của Thiết Bị" (Device Posture) lên Keycloak để Keycloak áp dụng Luật Zero Trust (Từ chối Đăng nhập)?**
- **Junior:** Nó tự cài phần mềm báo cáo lên.
- **Senior:** Trình duyệt web thông thường HOÀN TOÀN KHÔNG CÓ QUYỀN ĐỌC RAM để xem máy có virus không (Vi phạm Sandbox).
Để áp dụng Zero Trust Cấp Độ Thiết Bị (Device Trust), Hệ thống Công ty BẮT BUỘC phải Cài đặt Tác Vụ Ngầm (MDM - Mobile Device Management) hoặc Endpoint Security (Ví dụ CrowdStrike, Microsoft Intune) lên Máy Của Nhân Viên.
Quy trình: 
1. Intune Quét máy. Phát hiện Máy Đang Tắt Tường Lửa Windows. Intune Gửi Cờ Đỏ lên Cloud Microsoft.
2. Trình duyệt gọi Keycloak xin Đăng nhập.
3. Keycloak gọi API Sang Microsoft Cloud hỏi: "Máy của thằng A này đang Xịn hay Đang Nát?".
4. Microsoft báo: Nát.
5. Keycloak Lập tức chặn màn hình: *"Truy cập Bị Cấm. Vui lòng Bật Tường lửa Windows Defender trước khi đăng nhập lại"*.

**3. Khái niệm "Just-In-Time (JIT) Privilege" (Cấp quyền đúng lúc) trong Zero Trust khác gì với việc Cấp Quyền Admin thông thường? Tại sao Amazon/Google áp dụng nó Triệt để?**
- **Junior:** Nó là tạo acc nhanh.
- **Senior:** Đây là Đỉnh cao của Quản Trị Quyền.
Cách Cũ (Tội lỗi): Bạn là Kỹ sư DevOps trưởng. Tài khoản của Bạn CÓ SẴN Role `SUPER_ADMIN`. Bạn có thể Xóa Database của Amazon bất cứ lúc nào bạn thích. (Quyền Lực Dư Thừa thường trực).
**JIT Privilege (Zero Trust):** Dù bạn là Kỹ Sư Trưởng, Tài khoản của Bạn Ngày Thường CHỈ ĐƯỢC PHÉP ĐỌC LOG (Read-only). Hoàn toàn không có quyền xóa.
Lúc 2h sáng, Database Bị Sự Cố. Bạn cần Quyền Xóa để Cứu Hộ. Bạn phải Làm Đơn Xin Quyền (Request Access). Hệ thống Gửi Lệnh Phê Duyệt Tự Động (Hoặc cho Sếp Bấm Duyệt). 
BÙM! Bạn được Bơm Quyền `SUPER_ADMIN` Bằng Một Cái Token CHỈ SỐNG ĐÚNG 30 PHÚT.
Trong 30 phút đó mọi thao tác Bị Quay Phim Lại. 
Hết 30 phút, Quyền Lực TỰ ĐỘNG BỊ TƯỚC ĐOẠT, bạn rớt về làm Dân Thường. Cách này Hủy Diệt Hoàn Toàn Nguy cơ Hacker cướp Acc của Kỹ Sư Trưởng lúc ổng đang ngủ (Bởi vì lúc đó ổng chả có quyền gì cả).

---

## 7. Tài liệu tham khảo (References)
- **NIST SP 800-207:** Zero Trust Architecture.
- **Google Cloud:** BeyondCorp (Zero Trust Enterprise Security).
