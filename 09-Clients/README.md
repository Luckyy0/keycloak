# Chương 09: Cầu Nối Giữa Các Thế Giới (Clients & Application Registration)

> [!NOTE]
> Khi Lãnh Thổ (Realm) đã có, Thần Dân (Users) đã đông, và Thiết Quân Luật (Roles/Groups) đã ban hành. Câu hỏi tiếp theo là: Làm sao để các Phần mềm (Web App, Mobile App, API Server, Hệ thống Kế toán Cũ) có thể Kết Nối vào Lãnh Thổ OIDC này để xin Token?
> Đáp án chính là **Clients** (Ứng dụng Chư Hầu). 
> Keycloak định nghĩa mọi Phần Mềm bên ngoài muốn nói chuyện với nó đều phải được Đăng Ký và Phân Loại Thành Các Cấp Bậc (Public, Confidential) để bảo vệ An Toàn Tuyệt Đối cho Hệ Sinh Thái.

## Mục tiêu của chương
- Nắm vững 4 Thể Loại Ứng Dụng Chư Hầu (Client Types): Public, Confidential, Bearer-Only và Máy Móc (Service Accounts).
- Khắc cốt ghi tâm nguyên lý: Tại sao Web ReactJS tuyệt đối không được phép giữ `Client Secret`.
- Tuyệt kỹ Bức Tường Lửa Redirect URI: Đóng cổng không cho Hacker cuỗm mất Token bay ra khỏi vương quốc.
- Nâng cấp Áo Giáp PKCE (Proof Key for Code Exchange) cho App Di Động.
- Cấu hình Màn Hình Bán Đứng Thông Tin (Consent Screen) như cách Google hỏi "Bạn có cho phép App này đọc Email không?".

## Cấu trúc bài học

**Lô 1: Giải Phẫu Các Thể Loại Ứng Dụng (Lesson 1 - 4)**
- `Lesson-1-Public-Client.md`: Thằng Hề Lộ Liễu. Dành cho React, Angular, Mobile App. Tại sao nó không được giữ Secret.
- `Lesson-2-Confidential-Client.md`: Tướng Quân Bí Mật. Dành cho Backend (Spring Boot, Node.js, PHP). Nắm giữ `Client Secret` an toàn dưới đáy Server.
- `Lesson-3-Bearer-Only.md`: Kẻ Gác Cổng Mù Lòa. API Server chỉ biết Giải Mã Token, không bao giờ tự mở Form Đăng Nhập.
- `Lesson-4-Service-Account.md`: Rô-bốt Giao Dịch (Machine-to-Machine). Khi Máy Chủ cần nói chuyện với Máy Chủ (Cronjob, Đồng bộ Data) mà không có Khách Hàng nào bấm Nút.

**Lô 2: Vũ Khí Bảo Mật Nâng Cao (Lesson 5 - 8)**
- `Lesson-5-Client-Authentication.md`: Cách Tướng Quân Trình Diện Lãnh Chúa (Client ID + Secret vs JWT Client Assertion).
- `Lesson-6-Redirect-URI.md`: Tấm Bản Đồ Sinh Tử. Nơi Token được ném về sau khi Đăng Nhập xong.
- `Lesson-7-PKCE.md`: Chống ăn cắp Token trên Đường Truyền Cáp Quang bằng Mật Mã Động.
- `Lesson-8-Consent.md`: Màn hình Cam Kết. Bắt Khách Hàng ký giấy bán thông tin cho Ứng Dụng Bên Thứ Ba.

## Hướng dẫn thực hành (Labs)
- Đăng ký 1 Public Client cho App React, 1 Confidential Client cho App Java. Dùng Postman đóng vai Ứng Dụng để Móc Lõi Token bằng Flow (Authorization Code) và (Client Credentials) để xem sự khác biệt giữa Token có Khách và Token không Khách (Máy Móc).
