# Lesson 4: Luật Lệ Thép Phủ Đầu (PBAC - Policy-Based Access Control)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khi bạn đã có kiến trúc 4 Cột Trụ của Fine-Grained Authorization, cột trụ quan trọng nhất là **Policy (Chính sách)**. PBAC là khái niệm mở rộng bao hàm cả RBAC và ABAC, cho phép bạn thiết lập những bộ luật phức tạp, từ kiểm tra nhóm, giới hạn thời gian, chạy Javascript, đến ném lệnh HTTP tới server bên thứ 3 để xin phép. Bài học này sẽ giới thiệu các "Loại Policy" (Policy Types) mạnh mẽ nhất trong Keycloak.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. PBAC Là Gì?
Policy-Based Access Control (PBAC) là mô hình kiểm soát truy cập định hướng hoàn toàn bởi các Bộ Luật (Policies). Thay vì hardcode các đoạn lệnh if/else trong code Backend (Java/NodeJS), bạn trừu tượng hóa tất cả chúng thành các "Chính sách" nằm trên Keycloak.
Lợi ích vĩ đại nhất: Khi Luật Công Ty thay đổi, bạn vào Giao Diện Keycloak sửa Policy và bấm Save. Ngay lập tức hàng chục ứng dụng (Clients) đằng sau răm rắp tuân theo mà **KHÔNG CẦN DEPLOY LẠI CODE** Backend!

### 1.2. Kho Vũ Khí Policy Types Của Keycloak
Keycloak cung cấp sẵn những loại Cảnh Sát (Policy Types) sau để bạn xây luật:
1. **Role Policy:** (Áp dụng RBAC). Trả về TRUE nếu Token khách chứa Role tương ứng (Ví dụ: Đòi role `admin`).
2. **User Policy:** Luật cứng đầu nhất. Chỉ đích danh tên của một người. (Ví dụ: Chỉ cho phép tài khoản `nguyenvana`).
3. **Group Policy:** Trả về TRUE nếu User nằm trong phòng ban (Group) cụ thể (Ví dụ: Nằm trong group `Kế toán`).
4. **Client Policy:** Trả về TRUE nếu truy cập xuất phát từ 1 phần mềm (Client) cụ thể. (VD: Cấm không cho phép Mobile App truy cập xóa data, chỉ Web App mới được phép).
5. **Time Policy:** Luật thời gian (ABAC). Cho phép truy cập vào khoảng giờ, ngày, tháng cụ thể.
6. **Aggregated Policy (Luật Gộp):** Khối lượng lớn. Nó cho phép bạn Gom các luật lẻ tẻ ở trên lại thành 1 siêu luật (VD: Vừa phải có Role Admin AND Vừa phải nằm trong Group Kế Toán).

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Của Bộ Não Máy Cắt Lưới Luật Chạy JavaScript Policy:

```mermaid
flowchart TD
    A[Hành Động Khách Hàng Gọi Gõ Cửa API Đòi Action Vào Đáy Resource] --> B[Keycloak Kích Hoạt Lõi Permission Đánh Thức Policy]
    
    B --> C{Cục Policy: Loại JavaScript Script (Cực Tàn Bạo Trọng Lực Đáy Lõi)}
    C --> D[Load File Script .js Vào Môi Trường Động Cơ Rhino/Nashorn Của Java Xử Lý Lệnh Tĩnh]
    
    D --> E{Code JS Truy Cập Đối Tượng '$evaluation'}
    E --> F{Hỏi 1: $evaluation.getContext().getAttributes() - Tìm Địa Chỉ IP?}
    F -- Khớp IP Ngoài Nước --> G[Kích Lệnh Dưới Tầng Đáy Mã JS: $evaluation.deny()]
    
    E --> H{Hỏi 2: Thẻ Tín Dụng Khách Hàng Còn Hạn Không API?}
    H -- Trả Tiền Đủ Số Lượng Lớn --> I[Kích Lệnh Dưới Tầng Đáy Mã JS: $evaluation.grant()]
    
    G --> J[Máy Chặt Mạch Văng 403 HTTP Cấm Cửa Trắng Tinh Không Trải Lụa]
    I --> K[Bộ Máy Nhả Tầng Token Cấp Phép RPT Vào API Trút Dữ Liệu Báo Lệnh Pass Chóp]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh An Toàn Cấp Kiến Trúc (Tuyệt Đối Không Lạm Dụng JAVASCRIPT POLICY)**
> **Tội Ác Thiết Kế:** Bạn phát hiện ra Keycloak cho phép nhúng JS để làm Policy (Script Policy). Thấy hay quá, bạn nhét toàn bộ logic kiểm duyệt nghiệp vụ Database dơ bẩn của Backend (như Query số dư tài khoản, kiểm tra nợ xấu) vào thẳng mớ code JS này bắt Keycloak chạy.
> **Hậu Quả:** Khủng khiếp ở 2 mặt. Thứ 1: Code JS chạy trong lòng Java Engine của Keycloak bị cô lập (Sandbox), nó chạy siêu chậm và tiêu tốn CPU khổng lồ khi Traffic cao, làm sập Auth Server. Thứ 2: Quá trình Maintain (Bảo trì) mã nguồn trở thành ác mộng vì đống Logic Tài Chính lẽ ra phải nằm ở Core Bank Backend thì lại bị ném vào Server Xác Thực.
> **Biện Pháp Sống Còn Lớp Trọng:** Authorization Server Keycloak chỉ làm Nhiệm Vụ Phân Quyền dựa vào Context (Ngữ Cảnh Mạng/Identity) Của OIDC Nhả Về Trọng Lực API! Mọi logic Nghiệp Vụ Chuyên Sâu, Logic Kinh Doanh (Kế Toán, Trừ Tiền, So Sánh Số Dư) BẮT BUỘC PHẢI CODE Ở BACKEND (Spring Boot, Node). Nếu dùng Rule Policy Script, chỉ để kiểm tra các JSON Claims Nhẹ Nhàng Bắn Nhanh Có Sẵn Trong Token Header Trút Kéo Cắt Nhanh API!

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Hệ Thống Aggregated Policy (Luật Gộp Hỗn Hợp) Cho Tính Năng Trọng Lực An Toàn Siêu Cấp:
1. Bạn có 2 Luật lẻ đã tạo từ trước (Tạo từ Menu Policies): 
   - `Policy-1-Role-Vip`: Loại Role, yêu cầu phải có Role tên là `vip`.
   - `Policy-2-Office-Hours`: Loại Time, yêu cầu chỉ cho pass trong khung giờ 08:00 AM đến 05:00 PM.
2. Bây giờ bạn cần 1 luật thứ 3 cho một Permission Xóa Báo Cáo, đòi hỏi phải CÓ CẢ 2 thứ trên. Nếu bạn gắn lộn xộn 2 cái vào 1 Permission thì sẽ khó quản lý sự AND/OR logic.
3. Ở màn hình Policies, bấm **Create** -> Chọn Loại (Type) là **`Aggregated`**.
4. Đặt tên siêu luật này là: `Vip-And-Office-Hours-Policy`.
5. Trong giao diện cấu hình của nó, ở cột **Apply Policy**, bạn thả móc kéo chọn 2 thằng Đàn Em nhỏ bé vừa nhắc ở trên (`Policy-1-Role-Vip` và `Policy-2-Office-Hours`).
6. Kéo xuống dòng cấu hình **Decision Strategy**. BẠN PHẢI CHỌN ĐÚNG Lệnh (Xem bài tiếp theo để rõ hơn), tạm thời hãy chọn: **`Unanimous`** (Toàn thể nhất trí bằng phép AND toán học).
7. Save Lại. Lúc này bạn đã lắp xong 1 Bộ Trục Lệnh Thép Đáy Hỗn Hợp! Gắn nó vào Permission Xóa Dữ Liệu là Sếp Vip nếu xóa ngoài giờ hành chính cũng bị chặn đứt đuôi.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Cấu Hình PBAC Của Keycloak UMA. Nếu Em Thiết Kế Một Cục 'Role Policy' Đòi Hỏi Role 'admin'. Khi API Gửi Bắn Request Lên Xin Phép OIDC Máy Chủ. Hệ Thống Sẽ Truy Vấn (Query) Bảng Database Xem Thằng Này Đang Sở Hữu Role Hay Sẽ Bóc Cục JWT Token Access Của Nó Ra Đọc Text Lớp Chữ Nghĩa Token Để Ra Quyết Định Nhanh Trút Kéo Lụa?**
- **Senior:** Dạ thưa sếp, Chìa khóa thiết kế cốt lõi của PBAC Keycloak là nó **BÓC TOKEN BẰNG ENGINE ĐỊNH TUYẾN MẠCH NỘI BỘ (Claims Evaluation)**.
  - Khi một Client gửi kèm chuỗi Bearer Token (RPT/Access Token) lên cổng kiểm duyệt (PEP). Cái Token đó sẽ bị giải mã phần ruột Payload JSON. Keycloak sẽ dùng Cục Policy kiểm tra đối chiếu trực tiếp dữ liệu đang mang theo ở các field Claims trong cái JSON đó (Ví dụ mảng chuỗi string `roles: ["admin"]`) thay vì phải mở Connection Chọt thẳng xuống cục Postgres DB để soi xét nhằm giảm thiểu Độ Trễ I/O Khủng Khiếp DB Bound Mạch Khớp Lệnh Đáy!
  - Điều này giải thích tại sao Token cũ (Chưa Cập Nhật Role Mới Được Gán Ở DB) sẽ vẫn làm Client Trượt Khớp Lệnh Oanh Mạng Báo Lỗi 403 API Mù Lòa! Vì Bề Mặt Máy Chỉ Đọc Nhãn Chữ Đóng Dấu Trên Cái Token Đang Cầm Ở Tay!

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Authorization Services - Policies.
