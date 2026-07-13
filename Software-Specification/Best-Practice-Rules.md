# Best Practice Rules Specification
# Đặc tả Quy tắc Thực hành Tốt nhất

## 1. Enterprise Standards / Tiêu chuẩn Doanh nghiệp
- Cite architectural practices from industry leaders (Google, Microsoft, Netflix) and official RFCs (e.g., OAuth2 BCP, OIDC).
- Trích dẫn các thực hành kiến trúc từ các công ty hàng đầu trong ngành (Google, Microsoft, Netflix) và các RFC chính thức (ví dụ: OAuth2 BCP, OIDC).
- Follow strictly to OWASP Top 10 standards for identity and access management security.
- Tuân theo nghiêm ngặt các tiêu chuẩn OWASP Top 10 cho bảo mật quản lý định danh và truy cập.

## 2. Contextualization / Ngữ cảnh hóa
- State exactly *when* a best practice applies.
- Nêu chính xác *khi nào* một thực hành tốt nhất được áp dụng.
- State *when* a seemingly best practice might actually be an anti-pattern (e.g., using JWTs for session management in SPAs vs Backend-For-Frontend).
- Nêu rõ *khi nào* một điều tưởng chừng là thực hành tốt nhất lại có thể là một anti-pattern (ví dụ: dùng JWT cho quản lý phiên trong SPA so với Backend-For-Frontend).

## 3. Actionable Advice / Lời khuyên Hành động
- Provide concrete Keycloak configuration steps or Spring Security code snippets to enforce the best practice.
- Cung cấp các bước cấu hình Keycloak cụ thể hoặc đoạn mã Spring Security để thực thi thực hành tốt nhất đó.
