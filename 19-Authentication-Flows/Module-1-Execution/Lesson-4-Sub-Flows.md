> [!NOTE]
> **Category:** Theory / Architecture
> **Goal:** Cung cấp kiến thức chuyên sâu về Luồng phụ (Sub-Flows) và Xác thực có điều kiện (Conditional Authentication) trong Keycloak để xây dựng các kịch bản đăng nhập phức tạp cho hệ thống doanh nghiệp.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Hệ thống Xác thực (Authentication) của Keycloak không phải là một đường thẳng cố định, mà được thiết kế dưới dạng một **Cây phân cấp (Hierarchical Tree)**. Một Cây xác thực bao gồm:
- **Flow (Luồng):** Khung sườn bao bọc toàn bộ quá trình.
- **Execution (Bước thực thi):** Một thao tác xác thực cụ thể (VD: Nhập Password, Nhập OTP, Nhấn nút Accept).
- **Sub-Flow (Luồng phụ):** Là một Flow được lồng (nested) bên trong một Flow khác.

**Tại sao lại cần Sub-Flows?**
Trong các kịch bản nâng cao, bạn không chỉ kiểm tra tuần tự mật khẩu rồi đến OTP. Bạn cần gom nhóm (Group) các điều kiện phức tạp. Ví dụ, thiết lập một quy tắc: "Nếu người dùng là Quản trị viên (Role Admin), họ bắt buộc phải nhập OTP và WebAuthn. Nếu là người dùng thường, họ có quyền chọn (ALTERNATIVE) giữa OTP hoặc gửi mã qua Email." Sub-Flow cho phép bạn áp dụng các trạng thái thực thi (Requirement: REQUIRED, ALTERNATIVE, CONDITIONAL) lên một "nhóm" các bước xác thực thay vì từng bước đơn lẻ.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Khi người dùng nhấn Submit, hệ thống Execution Engine của Keycloak duyệt cây theo phương pháp Duyệt theo chiều sâu (Depth-First Search) từ trên xuống dưới. Sơ đồ dưới đây biểu diễn một Luồng Browser tùy chỉnh chứa Sub-Flow có điều kiện:

```mermaid
flowchart TD
    Start((Bắt đầu Login)) --> Form[Username Password Form\n(REQUIRED)]
    
    Form --> CondSubFlow{Sub-Flow: Xác thực nâng cao\n(CONDITIONAL)}
    
    CondSubFlow --> Cond1(Condition: User Role = Admin)
    CondSubFlow --> Cond2(Condition: User Network = External IP)
    
    Cond1 -- "True" --> OTP[OTP Form\n(REQUIRED)]
    Cond2 -- "True" --> OTP
    Cond1 -- "False" --> Bypass[Bypass Sub-Flow]
    Cond2 -- "False" --> Bypass
    
    OTP --> Success((Cấp Token))
    Bypass --> Success
```

**Cơ chế cấp thấp của các Trạng thái (Requirement) đối với Sub-Flow:**
1. **REQUIRED (Bắt buộc):** Toàn bộ nhóm Sub-Flow này bắt buộc phải chạy qua. Nếu bất kỳ Execution nào bên trong Sub-Flow thất bại, toàn bộ Flow thất bại.
2. **ALTERNATIVE (Lựa chọn):** Có thể chạy song song hoặc đưa ra danh sách các tuỳ chọn. Chỉ cần **MỘT** trong các cấu trúc bên trong Sub-Flow báo thành công, Keycloak sẽ bỏ qua các bước còn lại và đánh giá toàn bộ Sub-Flow là thành công.
3. **CONDITIONAL (Có điều kiện):** Sub-Flow này chỉ được kích hoạt nếu các Execution điều kiện (Condition - như kiểm tra Role, kiểm tra HTTP Header, IP) bên trong nó trả về giá trị `True`. Nếu `False`, hệ thống bỏ qua toàn bộ Sub-Flow một cách im lặng.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

- **Nguyên tắc "Sao chép trước khi sửa" (Copy before modify):** Bạn không bao giờ được phép chỉnh sửa trực tiếp các Flow mặc định của Keycloak (như `Browser` hay `Direct Grant`). Lỗi cấu trúc sẽ khiến bạn không thể đăng nhập vào cả Admin Console. Luôn "Duplicate" flow ra một bản nháp, sửa, test kĩ, rồi mới "Bind" nó làm luồng mặc định.
- **Dùng Sub-Flow để quản lý ALTERNATIVE thay vì Execution:** Việc đặt 2 Execution mang trạng thái ALTERNATIVE trực tiếp cạnh một REQUIRED Execution rất dễ gây nhầm lẫn về mặt logic đánh giá. Luôn bọc các bước tùy chọn vào một `ALTERNATIVE Sub-Flow`. Ví dụ: Bọc `OTP Form` (Alternative) và `WebAuthn` (Alternative) vào trong một `Sub-Flow 2FA`.
- **Cẩn thận với Xác thực có Điều kiện:** Nếu bạn đặt `CONDITIONAL` để qua mặt bước OTP cho mạng công ty, hãy chắc chắn rằng IP Header (`X-Forwarded-For`) không thể bị giả mạo bởi công cụ như Postman, nếu không lỗ hổng an ninh nghiêm trọng sẽ xảy ra.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Ví dụ thiết lập "Chỉ yêu cầu OTP nếu người dùng không thuộc mạng nội bộ":

1. Vào `Authentication` -> Copy luồng `Browser` thành `Browser Custom`.
2. Giữ nguyên bước `Username Password Form` ở trạng thái `REQUIRED`.
3. Thêm một Sub-Flow mới lồng phía dưới tên là `External 2FA Subflow`. Set trạng thái của nó là `CONDITIONAL`.
4. Trong `External 2FA Subflow`, thêm một execution điều kiện: `Condition - user configured for network`. Cấu hình mạng lưới (Vd: `192.168.1.0/24`) và set nó là `REQUIRED`.
5. Tiếp theo trong cùng `External 2FA Subflow`, thêm một execution: `OTP Form` và set là `REQUIRED`.

*Cấu trúc cây sẽ hiển thị như sau:*
- Browser Custom (Flow)
  - Username Password Form (REQUIRED)
  - External 2FA Subflow (CONDITIONAL)
    - Condition - user configured for network (REQUIRED)
    - OTP Form (REQUIRED)

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Vòng lặp vô tận (Infinite Loop) hoặc Kẹt Xác thực:** Xảy ra khi bạn bọc một Sub-Flow ở dạng `REQUIRED`, bên trong chứa một Execution mà người dùng chưa từng cấu hình (Ví dụ yêu cầu WebAuthn, nhưng User chưa khai báo WebAuthn ở lần đăng nhập đầu tiên, và bạn không bật tính năng tự động chuyển hướng đăng ký "Required Action"). Người dùng sẽ bị kẹt mãi mãi ở bước này và báo lỗi.
- **Điều kiện mâu thuẫn (Conflicting Conditions):** Khi có nhiều Condition trong một Conditional Sub-flow, Keycloak xử lý chúng theo logic **AND**. Nghĩa là TẤT CẢ các điều kiện phải thỏa mãn. Nếu bạn cấu hình "Condition: Role = Admin" và "Condition: Role = User", Sub-flow này sẽ không bao giờ được chạy.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **(Junior)** Tại sao phải nhóm các bước xác thực vào trong một Sub-Flow?
   - *Đáp án:* Để quản lý logic xác thực dễ dàng hơn, áp dụng các điều kiện chung (như trạng thái REQUIRED hay CONDITIONAL) cho cả một nhóm các bước (như OTP + Câu hỏi bí mật) thay vì cấu hình từng bước một.

2. **(Junior)** Một Sub-Flow mang trạng thái ALTERNATIVE hoạt động như thế nào?
   - *Đáp án:* Keycloak sẽ đưa ra các lựa chọn cho người dùng dựa trên các bước bên trong nó. Nếu người dùng thực hiện thành công bất kỳ một bước nào (chỉ cần 1), toàn bộ Sub-Flow được coi là thỏa mãn.

3. **(Senior)** Trong một Conditional Sub-Flow, nếu tôi thêm hai Condition (Ví dụ: IP nội bộ và Role Admin). Sub-Flow sẽ được kích hoạt khi nào?
   - *Đáp án:* Các execution mang nhãn "Condition" bên trong một Conditional Sub-Flow hoạt động như toán tử logic **AND**. Nó chỉ kích hoạt Sub-Flow nếu người dùng đáp ứng đồng thời CẢ HAI điều kiện.

4. **(Senior)** Làm thế nào để áp dụng logic **OR** cho Conditional Sub-Flow? (Ví dụ: Chạy OTP nếu IP nằm ngoài công ty HOẶC Role là Admin).
   - *Đáp án:* Keycloak mặc định không hỗ trợ toán tử OR trực tiếp trong một Sub-Flow. Giải pháp là tạo hai Conditional Sub-Flow riêng biệt hoạt động độc lập nối tiếp nhau, hoặc viết một Custom SPI Java cho Execution Condition tự đánh giá 2 logic đó.

5. **(Senior)** Nếu vô tình chỉnh sửa hỏng hoàn toàn luồng "Browser" mặc định (bị xóa mất form đăng nhập), quản trị viên sẽ bị khóa khỏi hệ thống. Cách khôi phục là gì?
   - *Đáp án:* Cách duy nhất là truy cập trực tiếp vào Cơ sở dữ liệu (Database) thông qua SQL, sửa lại bảng cấu hình Realm hoặc bảng Flow để trỏ lại luồng chuẩn. Đây là lý do quy tắc bất di bất dịch là phải "Duplicate" (sao chép) luồng ra trước khi sửa.

## 7. Tài liệu tham khảo (References)

- [Keycloak Authentication Flows & Sub-flows](https://www.keycloak.org/docs/latest/server_admin/#_authentication-flows)
- [Keycloak Conditional Authentication Steps](https://www.keycloak.org/docs/latest/server_admin/#_conditional_executions)
