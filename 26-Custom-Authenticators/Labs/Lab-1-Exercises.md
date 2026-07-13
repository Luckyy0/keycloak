> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Lập trình, cấu hình và triển khai một Custom Authenticator (Mã PIN Bí mật) tích hợp vào luồng đăng nhập (Authentication Flow) của Keycloak để bổ sung lớp bảo mật tùy chỉnh.

## 1. Kịch bản Thực hành (Lab Scenario)

Công ty của bạn yêu cầu một bước bảo mật phụ ngoài Mật khẩu. Tất cả nhân viên sau khi nhập đúng Mật khẩu sẽ phải điền một "Mã PIN Công ty" (Ví dụ: `12345`). Mã PIN này giống nhau cho tất cả mọi người và được cấu hình tĩnh trên hệ thống. Chỉ khi nhập đúng mã PIN này, người dùng mới được truy cập vào hệ thống.

Trong bài Lab này, bạn sẽ đóng vai trò Developer thực hiện việc xây dựng một **Secret PIN Authenticator**. Bạn sẽ:
1. Viết code Java cho `SecretPinAuthenticator` và `SecretPinAuthenticatorFactory`.
2. Tạo file giao diện `secret-pin.ftl` bằng FreeMarker.
3. Đóng gói (build) thành tệp JAR, triển khai vào thư mục `providers/`.
4. Sao chép Browser Flow mặc định, chèn Authenticator của bạn vào, và liên kết nó với quá trình đăng nhập của Realm.

## 2. Chuẩn bị Môi trường (Prerequisites)

- **Java JDK 17+** và **Apache Maven 3.8+**.
- **Keycloak Server** phiên bản Quarkus (đã được giải nén sẵn).
- Mã nguồn giao diện (Theme) tùy chỉnh đã được kích hoạt (hoặc ta có thể đặt trực tiếp file `.ftl` vào theme mặc định `base` để test).
- Tùy chọn: Một IDE như IntelliJ IDEA để chỉnh sửa code dễ dàng.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Khởi tạo Project Maven

Tạo dự án Maven `custom-pin-auth` với file `pom.xml` tương tự Lab SPI. Đảm bảo có các dependencies:
- `keycloak-server-spi` (scope: provided)
- `keycloak-server-spi-private` (scope: provided)
- `keycloak-services` (scope: provided)

### Bước 3.2: Lập trình Authenticator và Factory

1. Trong thư mục `src/main/java/com/example/keycloak/`, tạo class `SecretPinAuthenticator.java`:

```java
package com.example.keycloak;

import org.keycloak.authentication.AuthenticationFlowContext;
import org.keycloak.authentication.Authenticator;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.RealmModel;
import org.keycloak.models.UserModel;
import jakarta.ws.rs.core.MultivaluedMap;
import jakarta.ws.rs.core.Response;

public class SecretPinAuthenticator implements Authenticator {

    private static final String CORRECT_PIN = "12345"; // Hardcode cho mục đích Lab

    @Override
    public void authenticate(AuthenticationFlowContext context) {
        // Render form yêu cầu nhập PIN
        Response challenge = context.form().createForm("secret-pin.ftl");
        context.challenge(challenge);
    }

    @Override
    public void action(AuthenticationFlowContext context) {
        // Nhận dữ liệu POST từ người dùng
        MultivaluedMap<String, String> formData = context.getHttpRequest().getDecodedFormParameters();
        String enteredPin = formData.getFirst("secret_pin");

        if (CORRECT_PIN.equals(enteredPin)) {
            // Đúng mã -> đi tiếp
            context.success();
        } else {
            // Sai mã -> báo lỗi và hiển thị lại form
            Response challenge = context.form()
                    .setError("Mã PIN không hợp lệ.")
                    .createForm("secret-pin.ftl");
            context.failureChallenge(org.keycloak.events.Errors.INVALID_USER_CREDENTIALS, challenge);
        }
    }

    @Override
    public boolean requiresUser() { return false; }

    @Override
    public boolean configuredFor(KeycloakSession session, RealmModel realm, UserModel user) { return true; }

    @Override
    public void setRequiredActions(KeycloakSession session, RealmModel realm, UserModel user) {}

    @Override
    public void close() {}
}
```

2. Tạo class `SecretPinAuthenticatorFactory.java`:

```java
package com.example.keycloak;

import org.keycloak.Config;
import org.keycloak.authentication.Authenticator;
import org.keycloak.authentication.AuthenticatorFactory;
import org.keycloak.models.AuthenticationExecutionModel;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;
import org.keycloak.provider.ProviderConfigProperty;

import java.util.Collections;
import java.util.List;

public class SecretPinAuthenticatorFactory implements AuthenticatorFactory {

    public static final String PROVIDER_ID = "secret-pin-authenticator";

    @Override
    public String getDisplayType() { return "Secret PIN Validator"; }

    @Override
    public String getReferenceCategory() { return "Secret PIN"; }

    @Override
    public boolean isConfigurable() { return false; }

    @Override
    public AuthenticationExecutionModel.Requirement[] getRequirementChoices() {
        return REQUIREMENT_CHOICES; // Mặc định cung cấp REQUIRED, ALTERNATIVE, DISABLED
    }

    @Override
    public boolean isUserSetupAllowed() { return false; }

    @Override
    public String getHelpText() { return "Yêu cầu người dùng nhập mã PIN công ty."; }

    @Override
    public List<ProviderConfigProperty> getConfigProperties() { return Collections.emptyList(); }

    @Override
    public Authenticator create(KeycloakSession session) { return new SecretPinAuthenticator(); }

    @Override
    public void init(Config.Scope config) {}

    @Override
    public void postInit(KeycloakSessionFactory factory) {}

    @Override
    public void close() {}

    @Override
    public String getId() { return PROVIDER_ID; }
}
```

3. Đăng ký SPI: Tạo file `src/main/resources/META-INF/services/org.keycloak.authentication.AuthenticatorFactory` và điền:
`com.example.keycloak.SecretPinAuthenticatorFactory`

### Bước 3.3: Tạo Giao diện Form (FTL)

Để Keycloak render form, ta phải cung cấp tệp FreeMarker mẫu.
1. Khởi tạo một Theme mới (hoặc đặt tạm vào theme mặc định). Giả sử bạn tạo thư mục `src/main/resources/theme/mytheme/login/`.
2. Tạo tệp `secret-pin.ftl`:

```html
<#import "template.ftl" as layout>
<@layout.registrationLayout displayInfo=false; section>
    <#if section = "header">
        Nhập mã PIN Công ty
    <#elseif section = "form">
        <form id="kc-pin-form" class="${properties.kcFormClass!}" action="${url.loginAction}" method="post">
            <div class="${properties.kcFormGroupClass!}">
                <label for="secret_pin" class="${properties.kcLabelClass!}">Mã PIN</label>
                <input type="password" id="secret_pin" name="secret_pin" class="${properties.kcInputClass!}" autofocus />
            </div>
            <div class="${properties.kcFormGroupClass!}">
                <input class="${properties.kcButtonClass!} ${properties.kcButtonPrimaryClass!} ${properties.kcButtonBlockClass!}" name="login" type="submit" value="Xác nhận" />
            </div>
        </form>
    </#if>
</@layout.registrationLayout>
```
*(Cấu hình `theme.properties` nếu cần thiết).*

### Bước 3.4: Build và Deploy

1. Chạy lệnh: `mvn clean package`.
2. Copy tệp `custom-pin-auth-1.0-SNAPSHOT.jar` vào thư mục `providers/` của Keycloak.
3. Chạy `bin/kc.sh build`.
4. Chạy `bin/kc.sh start-dev`.

### Bước 3.5: Cấu hình Authentication Flow trên Admin Console

1. Đăng nhập Admin Console (Realm `master` hoặc custom).
2. Vào **Authentication** -> Nhấp vào nút ba chấm ở cạnh flow **Browser** -> Chọn **Duplicate**.
3. Đặt tên là `Custom Browser with PIN`.
4. Tìm đến dòng **Browser forms** (thuộc loại sub-flow).
5. Nhấp vào dấu **+** cạnh "Browser forms" (Add execution).
6. Tìm `Secret PIN Validator` và nhấn **Add**.
7. Chuyển Requirement của nó từ `DISABLED` sang `REQUIRED`.
8. Di chuyển vị trí của nó: Kéo nó lên nằm NAY DƯỚI `Username Password Form`.
9. Để áp dụng Flow này làm luồng mặc định: Quay lại danh sách flows, tìm dòng **Browser**, bấm menu, chọn **Bind flow**, chọn `Custom Browser with PIN` làm luồng mặc định cho Browser login.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu (Verification)

1. Mở cửa sổ ẩn danh và truy cập ứng dụng của bạn (hoặc Account Console của Keycloak).
2. Trang đăng nhập tiêu chuẩn (Username/Password) sẽ hiện ra. Đăng nhập bằng tài khoản hợp lệ.
3. **Mong đợi:** Thay vì vào được hệ thống ngay lập tức, một trang mới "Nhập mã PIN Công ty" sẽ hiện ra.
4. Nhập sai PIN (ví dụ: `111`): Giao diện sẽ báo "Mã PIN không hợp lệ" và yêu cầu nhập lại.
5. Nhập đúng PIN (`12345`): Hệ thống cho phép đi tiếp và cấp Access Token thành công.

### 4.2. Khắc phục sự cố (Troubleshooting)

- **Lỗi `FreeMarker template not found` (Trang lỗi trắng hoặc văng Stacktrace):**
  - *Nguyên nhân:* Keycloak không tìm thấy file `secret-pin.ftl`. Có thể bạn chưa cấu hình Theme đúng cách cho Realm, hoặc thư mục đóng gói sai.
  - *Khắc phục:* Đảm bảo bạn đã chọn `mytheme` trong phần **Realm Settings -> Themes -> Login theme**. Kiểm tra xem file FTL có nằm đúng ở thư mục `theme/mytheme/login/` bên trong tệp JAR hay không.
- **Thêm Execution rồi nhưng không thấy tác dụng:**
  - *Nguyên nhân:* Mặc dù bạn đã tạo luồng `Custom Browser with PIN`, nhưng bạn quên thực hiện bước **Bind flow** (Gán luồng). Do đó, Keycloak vẫn đang dùng flow `Browser` gốc.
  - *Khắc phục:* Vào Authentication -> Action -> Bind Flow -> Chọn luồng mới của bạn làm mặc định cho Browser.
- **Lỗi `Cookie not found` khi Submit PIN:**
  - *Nguyên nhân:* Trình duyệt ẩn danh của bạn bị cấu hình chặn Strict Cookies (đặc biệt là đối với các tên miền `localhost`).
  - *Khắc phục:* Cấu hình trình duyệt cho phép cookies từ Keycloak domain, hoặc không dùng chế độ chặn cross-site cookies nghiêm ngặt trong lúc Dev.
