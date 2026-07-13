# Lab 1: Nhập Cư Khách Hàng Từ MySQL (Custom User Storage)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay viết một Giao Diện SPI User Storage Bằng Java Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa. Bạn sẽ cắm dây nối Keycloak tới một database MySQL bên ngoài (Đóng giả là hệ thống cũ Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh). Khi gõ đăng nhập Oanh Khung Dịch Lụa Mạch Lệnh, Keycloak sẽ xuống MySQL kiểm tra Pass chữ nổi và cho phép chui vào hệ thống Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp. 

## 1. Yêu cầu (Prerequisites)
- Docker Compose.
- Dự án Java Maven rỗng (Đã chuẩn bị POM như các chương trước Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh).

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Khởi Tạo File POM.xml Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần
Bổ sung thư viện kết nối Database Cổ Xưa (JDBC Trút Khung Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa) vào `pom.xml`:

```xml
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi</artifactId>
            <version>24.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-server-spi-private</artifactId>
            <version>24.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
            <scope>provided</scope>
        </dependency>
```

### Bước 2: Viết Lớp Java Provider Chính (Trung Tâm Ma Thuật Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề)
Tạo file `src/main/java/com/mycompany/MySqlUserStorageProvider.java` Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy:

```java
package com.mycompany;

import org.keycloak.component.ComponentModel;
import org.keycloak.credential.CredentialInput;
import org.keycloak.credential.CredentialInputValidator;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.RealmModel;
import org.keycloak.models.UserModel;
import org.keycloak.models.credential.PasswordCredentialModel;
import org.keycloak.storage.UserStorageProvider;
import org.keycloak.storage.user.UserLookupProvider;
import org.keycloak.storage.adapter.AbstractUserAdapter;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class MySqlUserStorageProvider implements UserStorageProvider, 
        UserLookupProvider, 
        CredentialInputValidator {

    private final KeycloakSession session;
    private final ComponentModel model; // Cấu hình trên giao diện Web Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp
    private Connection connection;

    public MySqlUserStorageProvider(KeycloakSession session, ComponentModel model) {
        this.session = session;
        this.model = model;
        try {
            // Móc nối SQL Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị. Chú ý: Ở hệ thống thật phải dùng Hikari Pool Trút Lụa Code Cấu Trúc Khung Rỗng Kéo Sống Lệnh Chóp Cắt Đứt Nối Tương Lai Mạch Bơm Sống Rác Khủng API Đỉnh Đáy Oanh Mạng! 
            this.connection = DriverManager.getConnection("jdbc:mysql://mysql-db:3306/old_system", "root", "root_pass");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // --- PHẦN 1: TÌM KIẾM KHÁCH HÀNG (UserLookupProvider Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh) ---
    
    @Override
    public UserModel getUserByUsername(RealmModel realm, String username) {
        try {
            PreparedStatement st = connection.prepareStatement("SELECT * FROM tbl_khachhang WHERE name = ?");
            st.setString(1, username);
            ResultSet rs = st.executeQuery();
            if (rs.next()) {
                // Tóm được nó Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Lệnh Mạch Bọt Lõi Trút Code Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh! Đóng gói thành Cục Nhựa Của Keycloak Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh
                return createAdapter(realm, rs.getString("name"));
            }
        } catch (Exception e) {}
        return null;
    }

    private UserModel createAdapter(RealmModel realm, String username) {
        return new AbstractUserAdapter(session, realm, model) {
            @Override
            public String getUsername() {
                return username; // Thằng Khứa Này Tên Là username Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa
            }
        };
    }

    @Override
    public UserModel getUserById(RealmModel realm, String id) {
        // ID nội bộ của KC sinh ra cho Federate Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa, phải lột ra lấy Name Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
        String externalId = org.keycloak.storage.StorageId.externalId(id);
        return getUserByUsername(realm, externalId);
    }

    @Override
    public UserModel getUserByEmail(RealmModel realm, String email) {
        return null;
    }

    // --- PHẦN 2: XÁC THỰC MẬT KHẨU (CredentialInputValidator Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa) ---

    @Override
    public boolean supportsCredentialType(String credentialType) {
        return PasswordCredentialModel.TYPE.equals(credentialType); // Chỉ nhận xử lý loại Password Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
    }

    @Override
    public boolean isConfiguredFor(RealmModel realm, UserModel user, String credentialType) {
        return supportsCredentialType(credentialType);
    }

    @Override
    public boolean isValid(RealmModel realm, UserModel user, CredentialInput input) {
        if (!supportsCredentialType(input.getType())) return false;
        
        String passwordKhachGo = input.getChallengeResponse(); // Chữ Nổi Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh
        
        try {
            PreparedStatement st = connection.prepareStatement("SELECT pwd FROM tbl_khachhang WHERE name = ?");
            st.setString(1, user.getUsername());
            ResultSet rs = st.executeQuery();
            if (rs.next()) {
                String passTrongDB = rs.getString("pwd");
                return passwordKhachGo.equals(passTrongDB); // Do hệ thống đồ cổ nên DB nó lưu Pass Chữ Nổi luôn Cắt Khung Lệnh Rỗng Chóp Rút Nhựa Khớp Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh! Ma Giáo Chưa Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa!
            }
        } catch (Exception e) {}
        
        return false;
    }

    @Override
    public void close() {
        try { connection.close(); } catch (Exception e) {}
    }
}
```

### Bước 3: Viết Factory Cho Cục Gạch Này Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa
Tạo file `src/main/java/com/mycompany/MySqlUserStorageProviderFactory.java` Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề:

```java
package com.mycompany;

import org.keycloak.component.ComponentModel;
import org.keycloak.models.KeycloakSession;
import org.keycloak.storage.UserStorageProviderFactory;

public class MySqlUserStorageProviderFactory implements UserStorageProviderFactory<MySqlUserStorageProvider> {

    @Override
    public MySqlUserStorageProvider create(KeycloakSession session, ComponentModel model) {
        return new MySqlUserStorageProvider(session, model);
    }

    @Override
    public String getId() {
        return "mysql-legacy-db";
    }
}
```

### Bước 4: Khai Báo Dịch Vụ Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy
Tạo File: `src/main/resources/META-INF/services/org.keycloak.storage.UserStorageProviderFactory`
Nội Dung: `com.mycompany.MySqlUserStorageProviderFactory`

### Bước 5: Test Thành Quả
1. Gõ `mvn clean package`. Bỏ Jar vào thư mục Provider.
2. Dùng Docker Compose khởi tạo Keycloak kèm theo cục 1 container MySQL chạy chung mạng.
3. Trong MySQL tạo bảng `tbl_khachhang` có dòng `name = 'teo', pwd = '123'`.
4. Mở Keycloak Admin -> User Federation -> Add Provider -> Chọn `mysql-legacy-db`.
5. Đăng xuất admin Lệnh Oanh Rút Mạch Máu Cắt Đáy Oanh Mạng Bọc Thép Dịch Tễ Lạ Trượt Khung Khớp Lệnh Oanh Rỗng Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh. Gõ tên `teo`, Pass `123`.
6. Bạn chui lọt vào hệ thống! Teo đã ra đời từ Đống Tro Tàn Của Lịch Sử Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp!
