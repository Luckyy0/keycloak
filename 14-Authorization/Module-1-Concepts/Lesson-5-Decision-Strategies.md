# Lesson 5: Toán Học Quyết Định (Decision Strategies)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Khi bạn gắn NHIỀU LUẬT (Policy) vào 1 Quyền (Permission) duy nhất, sự cãi vã sẽ xảy ra. Luật A bảo "Cho qua", Luật B bảo "Cấm". Vậy cuối cùng Keycloak sẽ nhả token truy cập hay báo lỗi 403? Bài học này giúp bạn làm chủ Decision Strategy - Trọng tài phán xử của các chính sách!

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Decision Strategy Là Gì?
Khi có một cuộc bỏ phiếu từ các Cảnh Sát (Policies) đang gác cùng 1 cánh cửa, Decision Strategy là **Thuật toán kiểm đếm phiếu bầu** để đưa ra phán quyết cuối cùng (Final Decision). 
Có 2 cấp độ thiết lập Trọng Tài này:
1. **Tại Policy Gộp (Aggregated Policy):** Khi bạn gom nhiều Luật nhỏ thành 1 Siêu Luật lớn.
2. **Tại Permission:** Khi bạn móc nhiều Policy (hoặc Siêu luật) vào 1 cái Permission.

### 1.2. Ba Thuật Toán Bỏ Phiếu (The 3 Algorithms)
Keycloak cung cấp 3 phép toán logic (Boolean Logic) để bạn thiết lập:

1. **Unanimous (Nhất trí 100%):**
   - **Phép toán:** AND Logic. (`Policy_1 && Policy_2 && Policy_3`)
   - **Luật chơi:** TẤT CẢ các Policy đều phải giơ bảng "Pass" (Đồng ý) thì mới được qua. Chỉ cần 1 thằng giơ bảng "Fail" (Từ chối) là toàn bộ ngã ngựa.
   - **Dùng khi:** Các quy tắc bảo mật cực kỳ gắt gao. (VD: Vừa phải là Admin AND Vừa phải ở trong Mạng Công Ty).

2. **Affirmative (Chỉ Cần 1 Người Ủng Hộ):**
   - **Phép toán:** OR Logic. (`Policy_1 || Policy_2 || Policy_3`)
   - **Luật chơi:** Chỉ cần ÍT NHẤT MỘT Policy giơ bảng "Pass", là được qua ngay lập tức. Bất chấp 100 Policy khác đòi cấm.
   - **Dùng khi:** Tạo các lối đi tắt (Bypass). (VD: Một thằng được phép qua cửa nếu nó có Role Admin OR Nó có Role Sếp OR Hệ thống đang trong chế độ bảo trì).

3. **Consensus (Đa số thắng thiểu số):**
   - **Phép toán:** Phiếu Pass > Phiếu Fail.
   - **Luật chơi:** Đếm tổng số phiếu Pass và phiếu Fail. Nếu Pass nhiều hơn, cho qua. Nếu Fail nhiều hơn hoặc Bằng Nhau (Hòa), thì CHẶN.
   - **Dùng khi:** Ít dùng trong thực tế, thường dùng trong các hệ thống quản trị rủi ro phân tán (Risk Assessment) nơi các bộ đếm rủi ro tự động ném ra tín hiệu và cần lấy ý kiến số đông để quyết định.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Hành Trình OIDC Đi Qua Ma Trận Của Các Khối Logic OR Và AND:

```mermaid
flowchart TD
    A[Khách Yêu Cầu Gõ Cửa API] --> B{Permission X Mở Ra}
    
    B --> C[Thiết Lập Trọng Tài: AFFIRMATIVE (Chỉ cần 1 Pass)]
    
    C --> D[Chạy Policy 1: Có Role VIP Không?]
    D -- Không Có (Fail) --> E[Máy Tiếp Tục Chạy Cái Khác Tìm Cơ Hội Cứu]
    
    E --> F[Chạy Policy 2: Có Thanh Toán Trả Phí Mua Gói Tháng Không?]
    F -- Có (Pass) --> G[Lệnh AFFIRMATIVE Bắt Trúng Tín Hiệu Sống]
    
    G --> H[Ngừng Tính Toán Các Policy Khác Bên Dưới]
    H --> I[Quyết Định Cuối: PERMIT - Mở Cửa Trải Lụa]
    
    J[Ví dụ: Nếu Set UNANIMOUS] --> K[Policy 1 (Fail) Sẽ Lập Tức Chặt Luồng Không Cho Chạy Policy 2 Nữa -> Quyết Định DENY Tức Thì]
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Tuyệt Đỉnh Tẩy Khách Mạng Bọc (Nguy Cơ Hở Mạng Khi Dùng Affirmative Sai Chỗ)**
> **Tội Ác Thiết Kế:** Bạn muốn làm một tính năng an toàn: "Phải Có Role Kế Toán VÀ Phải Khớp IP Công Ty". Bạn lôi 2 cái Policy (1 cái kiểm Role, 1 cái kiểm IP) móc vào 1 cái Permission. Nhưng trong chỗ cấu hình Decision Strategy của Permission đó, bạn lại để mặc định là `Affirmative`.
> **Hậu Quả:** Một tay hacker (Không có Role Kế toán) nhưng hack được wifi nội bộ của công ty. Lệnh Affirmative thấy Policy IP báo "Pass", lập tức bỏ qua lỗi Fail của Role Kế toán. Nó mở cửa kho bạc cho tay hacker đi vào thoải mái!
> **Biện Pháp Sống Còn Lớp Trọng:** Trong phân quyền bảo mật, hãy rèn thói quen dùng lệnh **`Unanimous` (AND)** làm mặc định cho hầu hết mọi ngóc ngách! Chỉ được chuyển sang `Affirmative` (OR) khi bạn CHẮC CHẮN MÌNH ĐANG LÀM LỐI ĐI TẮT BYPASS! (Ví dụ: Thằng này là SuperAdmin thì cho qua luôn, khỏi check IP).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Lắp Ráp Hệ Thống Phân Quyền Hỗn Hợp Cho Bài Báo Mật Bằng Các Lệnh AND/OR Chồng Nhau:
*(Yêu cầu thực tế: Chỉ tác giả bài viết mới được sửa. HOẶC nếu thằng sửa có Role Admin thì cũng được sửa luôn).*
1. Bạn vào **Policies**, tạo Luật 1 (Javascript Policy) tên `Is-Author-Policy` (Đọc DB check xem nó có phải Author không).
2. Bạn tạo Luật 2 (Role Policy) tên `Is-Admin-Policy` (Check xem có chuỗi 'admin' không).
3. Sang Tab **Permissions**, tạo cái `Sua-Bai-Bao-Permission` loại Scope-Based móc vô hành động Sửa bài báo.
4. Ở phần chọn Policy, bạn THÊM CẢ 2 cái Luật 1 và 2 vào list áp dụng.
5. Cuộn xuống dòng **Decision Strategy**, bạn bắt buộc phải chọn **`Affirmative`**.
6. Giải thích: Vì đây là trường hợp HOẶC (Chỉ cần 1 trong 2 thỏa mãn). Nếu bạn để `Unanimous`, thằng tác giả bay vô sửa bài, máy tính toán: "Nó là tác giả (Pass) NHƯNG nó không có Role Admin (Fail) -> Đuổi Cổ Tác Giả!". Vậy là cấu hình Unanimous sẽ dội ngược lại logic mong muốn! Hãy tư duy như bảng mạch điện (AND/OR gates) khi nối dây các Policy.

---

## 5. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong Tab Authorization, Có Khái Niệm Decision Strategy Tại Cấp Độ Khối 'Permission' Và Cũng Lại Có Decision Strategy Tại Tầng Gốc 'Settings' Trọng Lực Của Toàn Bộ Client Policy Enforcer. Chúng Khác Nhau Chỗ Nào? Nếu Cậu Để Affirmative Ở Cấp Permission Mà Để Unanimous Ở Cấp Settings Thì Gì Sẽ Xảy Ra?**
- **Senior:** Hai khái niệm này quyết định ở 2 tầng khác nhau của Phễu (Trục Đứng Mạch Nhựa):
  - **Decision Strategy ở tầng Permission:** Quyết định sự cãi vã GIỮA CÁC LUẬT (Policies) bên trong chính cái Permission đó. (Luật A cãi Luật B xem có được Pass không).
  - **Decision Strategy ở tầng Settings Enforcer (Policy Enforcement Mode):** Quyết định sự cãi vã GIỮA NHIỀU PERMISSION CÙNG MÓC VÀO 1 TÀI NGUYÊN (Resource). (Ví dụ Tài nguyên Bài Báo 1 đang bị kẹp bởi 2 Permission X và Y).
  - Nếu Cậu Set Unanimous ở Tầng Gốc Settings: Giả sử User pass Permission X một cách nhẹ nhàng (nhờ lệnh Affirmative bên trong nó). NHƯNG User lại rớt đài ở Permission Y (Ví dụ Permission Y đòi IP mạng). Do tầng gốc Set Unanimous, máy sẽ đòi hỏi User PHẢI PASS TẤT CẢ PERMISSION ĐANG CỘT VÀO TÀI NGUYÊN. Nghĩa là rớt đài Tự hủy! Quyết định của Tầng Gốc luôn là Quyết định Tối Thượng Dập Dòng Cuối Cùng Của OIDC.

---

## 6. Tài liệu tham khảo (References)
- **Keycloak Documentation:** Authorization Services - Evaluating Permissions.
