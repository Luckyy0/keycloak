> [!NOTE]
> **Category:** Theory
> **Goal:** Nghiên cứu sâu về `First Broker Login Flow` trong Keycloak, cơ chế quyết định cách xử lý người dùng mới trong lần đầu tiên họ đăng nhập thông qua một External Identity Provider.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong cấu trúc Identity Brokering của Keycloak, khi một người dùng đăng nhập bằng hệ thống ngoại vi (Identity Provider - IdP) như Google, Azure AD, hay GitHub, Keycloak cần phải quyết định sẽ làm gì với hồ sơ (profile) vừa nhận được từ IdP đó. Quá trình ra quyết định này trong lần đăng nhập đầu tiên được gọi là **First Broker Login Flow**.

**Tại sao First Broker Login Flow tồn tại?**
- **Đồng bộ hóa dữ liệu (Provisioning)**: Khi nhận dữ liệu từ IdP, Keycloak cần sao chép (import) thông tin đó vào database cục bộ của mình để cấp phát Access Token độc lập.
- **Giải quyết xung đột (Conflict Resolution)**: Nếu IdP trả về một Email đã tồn tại trong Keycloak, hệ thống cần có cơ chế để hỏi ý kiến người dùng (Link Account) hoặc từ chối để tránh vi phạm bảo mật.
- **Bổ sung thông tin (Profile Update)**: Đôi khi IdP trả về thiếu thông tin quan trọng (ví dụ: thiếu số điện thoại). Flow này cho phép yêu cầu người dùng điền thêm trước khi cho phép vào hệ thống.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Quá trình `First Broker Login Flow` hoạt động như một hệ thống State Machine (Máy trạng thái) nằm trong thành phần Authentication SPI của Keycloak.

```mermaid
flowchart TD
    A[Nhận Token/Assertion từ IdP] --> B{Email/Username đã tồn tại?}
    B -- Không --> C[Create User If Unique]
    B -- Có --> D{IdP có Trust Email?}
    D -- Có --> E[Tự động Link Account]
    D -- Không --> F[Handle Existing Account]
    F --> G[Yêu cầu User xác minh (Verify Email / Password)]
    G --> H[Hoàn tất Account Linking]
    C --> I{Thiếu thông tin bắt buộc?}
    I -- Có --> J[Update Profile Page]
    I -- Không --> K[Hoàn tất Provisioning]
    J --> K
    K --> L[Cấp phát Session & Token cho Client]
    H --> L
```

**Cơ chế cấp thấp:**
1. **IdP Callback**: Keycloak nhận phản hồi qua đường dẫn `/auth/realms/{realm}/broker/{alias}/endpoint`.
2. **Review Profile**: Dữ liệu từ IdP được mapping thông qua các `Identity Provider Mappers` để chuyển đổi các claims bên ngoài thành các thuộc tính người dùng cục bộ.
3. **Execution Pipeline**: Keycloak sẽ duyệt qua danh sách các `Authenticators` được cấu hình trong `First Broker Login Flow`. Nếu một Authenticator không thành công, toàn bộ Flow bị tạm dừng và trả về thông báo lỗi hoặc trang Form yêu cầu nhập thêm.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!WARNING]
> **Tấn công Account Takeover qua Email Không tin cậy**: Không bao giờ kích hoạt Execution `Automatically Set Existing User` trừ khi bạn kiểm soát hoàn toàn IdP (như Keycloak to Keycloak) hoặc IdP đó luôn xác minh Email một cách nghiêm ngặt. Đối với các IdP kém an toàn, hãy luôn yêu cầu `Verify Existing Account by Email`.

> [!IMPORTANT]
> **Quản lý Mappers chặt chẽ**: Chỉ map những Claims (thuộc tính) mà bạn thực sự cần từ IdP. Việc map thừa thông tin có thể ghi đè (override) dữ liệu nhạy cảm đã có sẵn của user trên Keycloak nội bộ, gây sai lệch trạng thái hệ thống.

- **Đồng bộ hóa Role (Role Sync)**: Hạn chế dùng First Broker Login để gán trực tiếp quyền (Role). Nên sử dụng `Hardcoded Role` Mapper kết hợp với các điều kiện Logic để đảm bảo an toàn.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Bạn có thể chỉnh sửa First Broker Login Flow thông qua **Authentication** -> tab **Flows**. Cấu trúc mặc định bao gồm các bước sau:
1. `Review Profile` (Alternative/Disabled): Cho phép user xem và chỉnh sửa thông tin được lấy về từ IdP.
2. `Create User If Unique` (Alternative): Tạo user mới nếu email chưa tồn tại.
3. `Handle Existing Account` (Alternative):
   - `Verify Existing Account by Email` (Required): Gửi email có chứa link xác nhận để đảm bảo họ là chủ sở hữu.
   - `Verify Profile` (Required): Bắt buộc người dùng cung cấp thêm một phương thức xác thực nếu cần.

*Đoạn mã ví dụ để tự động hóa cấu hình bằng Keycloak CLI (kcadm.sh):*
```bash
# Lấy ID của First Broker Login Flow
FLOW_ID=$(kcadm.sh get authentication/flows -r myrealm --fields id,alias | grep -B1 "first broker login" | head -n1 | cut -d '"' -f4)

# Thay đổi cấu hình của execution "Review Profile" sang Disabled để tăng tốc UX
kcadm.sh update authentication/flows/$FLOW_ID/executions \
  -r myrealm \
  -b '{"requirement": "DISABLED"}'
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Người dùng bỏ dở quá trình (Flow Abandonment)**: Nếu user đang ở bước Update Profile hoặc Verify Email nhưng lại đóng trình duyệt. Lần sau họ quay lại, session có thể hết hạn, dẫn đến lỗi `Expired Code` hoặc `Action Expired`. Cần cấu hình thời gian sống của session đủ hợp lý trong mục `Realm Settings -> Tokens -> Action Token Lifespan`.
- **Identity Provider thay đổi Email**: Nếu ở lần đăng nhập tiếp theo, người dùng thay đổi Email tại IdP của họ, Keycloak sẽ coi nó là một luồng đăng nhập bình thường nếu tài khoản đã được Link từ trước, vì Keycloak dựa vào `BROKER_USER_ID` chứ không chỉ dựa vào Email. Tuy nhiên, việc đồng bộ hoá Email mới hay không phụ thuộc vào cấu hình Mapper.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**Junior Level:**
1. First Broker Login Flow là gì và nó kích hoạt khi nào?
   - *Đáp án:* Là luồng xử lý xác thực kích hoạt khi một người dùng đăng nhập lần đầu tiên vào Keycloak thông qua một Identity Provider ngoại vi.
2. Tại sao người dùng lại bị hiển thị màn hình "Update Account Information" dù họ đã đăng nhập thành công qua Google?
   - *Đáp án:* Do bước `Review Profile` trong Flow đang được bật, hoặc IdP (như Google) không cung cấp đủ các trường thông tin bắt buộc mà Keycloak yêu cầu (ví dụ: thiếu First Name).

**Senior Level:**
3. Trình bày chi tiết luồng xử lý nếu hai Identity Providers khác nhau cùng trả về một email nhưng thuộc về hai user vật lý khác nhau?
   - *Đáp án:* Keycloak sẽ phát hiện email xung đột tại bước `Create User If Unique`. Sau đó, nó chuyển sang nhánh `Handle Existing Account`. Nếu hệ thống không cấu hình bước Verify (qua Email/OTP), tài khoản có thể bị liên kết sai. Nếu có cấu hình, user thứ hai sẽ không thể xác minh qua email và sẽ bị từ chối truy cập.
4. Làm cách nào để bạn xây dựng một quy trình hoàn toàn "Silent Provisioning" (Không hiển thị bất kỳ màn hình nào cho User) khi tích hợp với Azure AD?
   - *Đáp án:* Phải Disable toàn bộ bước `Review Profile`, gán `Trust Email` trong cấu hình của Identity Provider, và đảm bảo mọi required attributes đều được mapping đầy đủ qua IdP Mappers.
5. Giải thích sự khác biệt giữa `First Broker Login Flow` và `Post Broker Login Flow`?
   - *Đáp án:* First Broker Login chỉ chạy đúng một lần duy nhất khi User chưa tồn tại trong Local DB (hoặc chưa được Link). Post Broker Login chạy ở MỌI LẦN đăng nhập qua IdP, thường được dùng để force update các điều khoản pháp lý hoặc cập nhật Role đồng bộ từ hệ thống bên ngoài.

## 7. Tài liệu tham khảo (References)

- [Keycloak Official Documentation - First Broker Login Flow](https://www.keycloak.org/docs/latest/server_admin/#_first_broker_login)
- [Keycloak SPI Documentation - Authentication Flows](https://www.keycloak.org/docs/latest/server_development/#_auth_spi)
