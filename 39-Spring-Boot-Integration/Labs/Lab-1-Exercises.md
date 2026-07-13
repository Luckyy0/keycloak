> [!NOTE]
> **Category:** Practical/Lab  
> **Goal:** Thực hành tích hợp hoàn chỉnh một ứng dụng Spring Boot đóng vai trò vừa là OAuth2 Client (để đăng nhập) vừa là Resource Server (để bảo vệ API) sử dụng Keycloak.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn được giao nhiệm vụ xây dựng một hệ thống Web Portal nội bộ cho công ty bằng Spring Boot. Yêu cầu đặt ra:
1. Khi người dùng truy cập vào trang chủ `/home`, họ phải bị điều hướng sang trang đăng nhập của Keycloak.
2. Sau khi đăng nhập thành công, Keycloak trả về một JWT Access Token.
3. Portal có một API nội bộ `/api/manager` chỉ cho phép những người dùng có Role `MANAGER` truy cập. 
4. Bạn phải ánh xạ đúng Role từ Keycloak để Spring Boot có thể cấp quyền bằng `@PreAuthorize`.

## 2. Chuẩn bị Môi trường (Prerequisites)

Để thực hiện bài Lab này, bạn cần có:
- **Keycloak Server:** Đang chạy tại `http://localhost:8080`.
- **JDK 17+** và **Maven**.
- **IDE:** IntelliJ IDEA hoặc Eclipse.
- **Dữ liệu Keycloak (Tạo trước):**
  - Tạo Realm: `company-realm`.
  - Tạo Client: `portal-app` (bật "Standard Flow" và "Client authentication").
  - Tạo Realm Role: `MANAGER`.
  - Tạo User: `john_doe` (mật khẩu `123456`), gán role `MANAGER` cho user này.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 1: Khởi tạo Project Spring Boot

Sử dụng Spring Initializr (hoặc IDE) tạo dự án với các Dependencies sau:
- Spring Web (`spring-boot-starter-web`)
- Spring Security (`spring-boot-starter-security`)
- OAuth2 Client (`spring-boot-starter-oauth2-client`)
- OAuth2 Resource Server (`spring-boot-starter-oauth2-resource-server`)

### Bước 2: Cấu hình `application.yml`

Mở file `src/main/resources/application.yml` và nhập cấu hình kết nối tới Keycloak:

```yaml
server:
  port: 8081

spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: portal-app
            client-secret: <COPY_SECRET_TU_KEYCLOAK_VAO_DAY>
            scope: openid, profile, email
            authorization-grant-type: authorization_code
        provider:
          keycloak:
            issuer-uri: http://localhost:8080/realms/company-realm
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/company-realm
```

### Bước 3: Viết logic Ánh xạ Role (Role Mapping)

Tạo file `SecurityConfig.java` để tùy chỉnh JwtConverter chuyển đổi thuộc tính `realm_access.roles` của Keycloak thành `ROLE_` của Spring Security:

```java
package com.example.portal.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationToken;
import org.springframework.security.web.SecurityFilterChain;

import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public").permitAll()
                .anyRequest().authenticated()
            )
            // Kích hoạt OAuth2 Login (Client) cho truy cập Web
            .oauth2Login(oauth2 -> {})
            // Kích hoạt Resource Server cho các truy cập gọi API bằng Bearer Token
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthConverter()))
            );
        
        return http.build();
    }

    private Converter<Jwt, ? extends AbstractAuthenticationToken> jwtAuthConverter() {
        return jwt -> {
            Map<String, Object> realmAccess = jwt.getClaim("realm_access");
            Collection<GrantedAuthority> authorities = Collections.emptyList();
            if (realmAccess != null && realmAccess.containsKey("roles")) {
                List<String> roles = (List<String>) realmAccess.get("roles");
                authorities = roles.stream()
                        .map(roleName -> "ROLE_" + roleName)
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());
            }
            return new JwtAuthenticationToken(jwt, authorities);
        };
    }
}
```

### Bước 4: Viết Controller

Tạo file `PortalController.java` để định nghĩa các Endpoint:

```java
package com.example.portal.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.core.oidc.user.OidcUser;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PortalController {

    @GetMapping("/public")
    public String publicEndpoint() {
        return "Trang này ai cũng xem được.";
    }

    @GetMapping("/home")
    public String home(@AuthenticationPrincipal OidcUser user) {
        return "Chào mừng " + user.getFullName() + ". Bạn đã đăng nhập thành công!";
    }

    @PreAuthorize("hasRole('MANAGER')")
    @GetMapping("/api/manager")
    public String managerApi() {
        return "Dữ liệu mật chỉ dành cho Quản lý.";
    }
}
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

### 4.1. Nghiệm thu
1. Mở trình duyệt ẩn danh, truy cập `http://localhost:8081/public`. Trình duyệt sẽ hiển thị nội dung lập tức mà không bị điều hướng.
2. Đổi URL sang `http://localhost:8081/home`. Trình duyệt lập tức bị Redirect tới trang đăng nhập của Keycloak.
3. Đăng nhập với tài khoản `john_doe` / `123456`.
4. Nếu thành công, trang sẽ điều hướng về `/home` hiển thị dòng chữ "Chào mừng...".
5. Truy cập tiếp `http://localhost:8081/api/manager`. Bạn sẽ thấy dòng "Dữ liệu mật chỉ dành cho Quản lý".

### 4.2. Khắc phục sự cố (Troubleshooting)
- **Lỗi 403 Forbidden khi truy cập `/api/manager`:** Bạn có thể đã quên gán quyền `MANAGER` cho user trong Keycloak, hoặc quên thêm đoạn `jwtAuthConverter` để thêm tiền tố `ROLE_`. Kiểm tra console log hoặc in token payload ra màn hình.
- **Lỗi Connection Refused trong Log Spring Boot:** Spring Boot không gọi được `http://localhost:8080`. Kiểm tra xem Keycloak có đang chạy không, hoặc tên miền cấu hình trong `issuer-uri` có chính xác không.
- **Lỗi Invalid Redirect URI:** Đảm bảo trong cấu hình Client `portal-app` trên Keycloak, ô `Valid Redirect URIs` đã được điền là `http://localhost:8081/login/oauth2/code/keycloak`. Mặc định Spring Boot sử dụng đường dẫn này để nhận mã Code.
