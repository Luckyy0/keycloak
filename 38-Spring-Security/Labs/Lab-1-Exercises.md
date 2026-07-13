> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tích hợp Spring Boot (đóng vai trò Resource Server) với Keycloak (Authorization Server) để bảo vệ REST API bằng JWT, và trích xuất Role từ Keycloak để phân quyền người dùng.

## 1. Kịch bản Thực hành (Lab Scenario)
Bạn đang phát triển một API backend cho hệ thống quản lý nhân sự. Yêu cầu đặt ra là mọi API đều phải được bảo vệ. 
- API `/api/public/info` cho phép truy cập tự do.
- API `/api/users/profile` yêu cầu người dùng phải đăng nhập (bất kỳ role nào).
- API `/api/admin/dashboard` yêu cầu người dùng phải có quyền (role) là `admin`.
Bạn sẽ sử dụng Keycloak để tạo các user/role này, sinh ra JWT (Access Token), và gọi vào Spring Boot để kiểm chứng quá trình phân quyền hoạt động.

## 2. Chuẩn bị Môi trường (Prerequisites)
- Keycloak Server đang chạy (có thể dùng Docker: `docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev`).
- JDK 17+ và Maven/Gradle.
- Một dự án Spring Boot mới có các dependencies: `spring-boot-starter-web`, `spring-boot-starter-security`, `spring-boot-starter-oauth2-resource-server`.
- Postman hoặc cURL để test API.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

**Bước 1: Cấu hình Keycloak**
1. Đăng nhập vào Keycloak Admin Console (http://localhost:8080).
2. Tạo một Realm mới tên là `spring-boot-realm`.
3. Tạo một Client mới:
   - Client ID: `spring-app`
   - Client Authentication: `Off` (Public client vì chúng ta chỉ dùng để sinh token qua Postman).
   - Valid Redirect URIs: `*`
4. Tạo Realm Roles:
   - Tạo role `admin`
   - Tạo role `user`
5. Tạo User:
   - User 1: `john_admin`, gán password, sau đó gán Role Mapping là `admin`.
   - User 2: `jane_user`, gán password, gán Role Mapping là `user`.

**Bước 2: Cấu hình Spring Boot application.yml**
Mở file `application.yml` trong project Spring Boot và thêm cấu hình kết nối tới Keycloak:

```yaml
server:
  port: 8081

spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/spring-boot-realm
          jwk-set-uri: http://localhost:8080/realms/spring-boot-realm/protocol/openid-connect/certs
```

**Bước 3: Viết mã cấu hình Security và REST Controller**
Tạo file `SecurityConfig.java`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.core.convert.converter.Converter;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("admin")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))
            );
        return http.build();
    }

    private JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(new Converter<Jwt, Collection<GrantedAuthority>>() {
            @Override
            public Collection<GrantedAuthority> convert(Jwt jwt) {
                Map<String, Object> realmAccess = (Map<String, Object>) jwt.getClaims().get("realm_access");
                if (realmAccess == null || realmAccess.isEmpty()) {
                    return List.of();
                }
                List<String> roles = (List<String>) realmAccess.get("roles");
                return roles.stream()
                        .map(roleName -> new SimpleGrantedAuthority("ROLE_" + roleName))
                        .collect(Collectors.toList());
            }
        });
        return converter;
    }
}
```

Tạo file `ApiController.java`:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class ApiController {

    @GetMapping("/public/info")
    public String publicInfo() {
        return "This is public info";
    }

    @GetMapping("/users/profile")
    public String userProfile() {
        return "This is user profile";
    }

    @GetMapping("/admin/dashboard")
    public String adminDashboard() {
        return "This is admin dashboard";
    }
}
```
Chạy ứng dụng Spring Boot.

**Bước 4: Sinh Token và Test API**
Mở Postman để sinh token từ Keycloak bằng luồng `Password Credentials` (cho tiện test) hoặc copy từ Keycloak Console:
```bash
curl -X POST http://localhost:8080/realms/spring-boot-realm/protocol/openid-connect/token \
  -d "client_id=spring-app" \
  -d "username=john_admin" \
  -d "password=mypassword" \
  -d "grant_type=password"
```
Copy trường `access_token` từ chuỗi JSON trả về.

Gọi API trên Spring Boot bằng token vừa sinh ra:
```bash
curl -H "Authorization: Bearer <access_token>" http://localhost:8081/api/admin/dashboard
```

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Kiểm tra tính đúng đắn:**
- Gọi `GET /api/public/info` không cần Header Authorization -> Phải trả về `200 OK`.
- Gọi `GET /api/admin/dashboard` không có token -> Trả về `401 Unauthorized`.
- Gọi `GET /api/admin/dashboard` với token của `jane_user` (chỉ có role user) -> Trả về `403 Forbidden` (Vì thiếu quyền admin).
- Gọi `GET /api/admin/dashboard` với token của `john_admin` -> Phải trả về `200 OK` kèm nội dung "This is admin dashboard".

> [!TIP]
> Nếu bạn bị lỗi 403 mặc dù đã đăng nhập bằng user admin, hãy giải mã (decode) JWT của bạn tại [jwt.io](https://jwt.io) để kiểm tra xem cấu trúc JSON có chứa block `realm_access.roles` với giá trị `"admin"` không.

> [!WARNING]
> Đảm bảo rằng cấu hình `jwtAuthenticationConverter` đã tự động gán thêm tiền tố `ROLE_` trước tên role. Vì phương thức `.hasRole("admin")` của Spring Boot tự động ngầm định nối tiền tố này khi kiểm tra. Nếu bạn convert không có `ROLE_`, Spring sẽ so sánh sai và trả về 403.
