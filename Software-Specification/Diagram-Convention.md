# Diagram Convention Specification
# Đặc tả Chuẩn Sơ đồ

## 1. Tooling / Công cụ
All diagrams must be generated using **Mermaid.js** directly inside the Markdown files.
Tất cả các sơ đồ phải được sinh ra bằng **Mermaid.js** trực tiếp bên trong các tệp Markdown.
This allows easy editing and rendering natively on GitHub and modern IDEs.
Điều này cho phép dễ dàng chỉnh sửa và hiển thị nguyên bản trên GitHub và các IDE hiện đại.

## 2. Supported Diagram Types / Các loại Sơ đồ được Hỗ trợ
The curriculum heavily relies on visual learning. Ensure the following types are used appropriately:
Giáo trình phụ thuộc nhiều vào việc học qua hình ảnh. Đảm bảo sử dụng các loại sau cho phù hợp:
- **Sequence Diagram (`sequenceDiagram`)**: Mandatory for all Authentication, Authorization, and Token flows (OAuth2, SAML, OIDC).
- **Biểu đồ Tuần tự**: Bắt buộc đối với tất cả các luồng Xác thực, Ủy quyền và Token (OAuth2, SAML, OIDC).
- **Flowchart (`flowchart TD / LR`)**: Used for decision trees, request routing, and internal component logic.
- **Lưu đồ**: Được sử dụng cho cây quyết định, định tuyến yêu cầu và logic thành phần nội bộ.
- **Architecture / Deployment Diagram**: Used to illustrate Keycloak Clusters, Reverse Proxies (Nginx), and Database (PostgreSQL/Redis) deployments.
- **Biểu đồ Kiến trúc / Triển khai**: Được sử dụng để minh họa các cụm Keycloak, Reverse Proxy (Nginx) và triển khai Cơ sở dữ liệu (PostgreSQL/Redis).
- **Class Diagram (`classDiagram`)**: Used for explaining custom SPI implementations and database schemas.
- **Biểu đồ Lớp**: Được sử dụng để giải thích các triển khai SPI tùy chỉnh và lược đồ cơ sở dữ liệu.

## 3. Styling Rules / Quy tắc Kiểu dáng
- Use clear entity names (e.g., `Browser`, `Frontend App`, `Keycloak (IdP)`, `Resource Server`).
- Sử dụng tên thực thể rõ ràng (ví dụ: `Browser`, `Frontend App`, `Keycloak (IdP)`, `Resource Server`).
- Add comments inside the Mermaid block to explain complex steps.
- Thêm chú thích bên trong khối Mermaid để giải thích các bước phức tạp.
- Use numbering in sequence diagram messages (e.g., `autonumber`) to map directly to the text explanations.
- Sử dụng đánh số trong các thông báo của biểu đồ tuần tự (ví dụ: `autonumber`) để ánh xạ trực tiếp với phần giải thích bằng văn bản.
