# Chapter 00: Prerequisites
# Chương 00: Điều kiện tiên quyết

> [!NOTE]
> **Category:** Architecture/Design (Kiến trúc/Thiết kế)
> **Goal:** Provide the foundational networking, cryptography, and security knowledge required before learning Keycloak.
> **Mục tiêu:** Cung cấp kiến thức nền tảng về mạng, mật mã và bảo mật cần thiết trước khi học Keycloak.

## 1. Introduction / Giới thiệu
Welcome to Chapter 00. Before we can secure a system with Keycloak, we must intimately understand how the web operates, how data is encrypted, and the common attack vectors we must defend against.
Chào mừng đến với Chương 00. Trước khi có thể bảo mật một hệ thống bằng Keycloak, chúng ta phải hiểu tường tận cách web hoạt động, cách dữ liệu được mã hóa và các vectơ tấn công phổ biến mà chúng ta phải phòng thủ.

This chapter serves as a comprehensive refresher. It bridges the gap between basic web development and enterprise-grade security engineering.
Chương này đóng vai trò như một phần ôn tập toàn diện. Nó thu hẹp khoảng cách giữa phát triển web cơ bản và kỹ thuật bảo mật cấp độ doanh nghiệp.

## 2. Module Breakdown / Cấu trúc Mô-đun

To make learning structured, the massive list of prerequisites has been organized into four distinct modules:
Để việc học có cấu trúc, danh sách khổng lồ các điều kiện tiên quyết đã được tổ chức thành bốn mô-đun riêng biệt:

### Module 1: Web Networking & State (Mạng Web & Trạng thái)
Focuses on how browsers, clients, and servers communicate securely and maintain state.
Tập trung vào cách trình duyệt, máy khách và máy chủ giao tiếp an toàn và duy trì trạng thái.
- `Lesson-1-HTTP-HTTPS.md`: HTTP, HTTPS, TLS, SSL
- `Lesson-2-Web-State.md`: Cookies, Sessions, CORS
- `Lesson-3-Infrastructure.md`: DNS, Reverse Proxy, Load Balancer

### Module 2: Applied Cryptography (Mật mã Ứng dụng)
Focuses on securing data in transit and at rest, forming the foundation of modern tokens.
Tập trung vào việc bảo mật dữ liệu đang truyền và lưu trữ, hình thành nền tảng của các token hiện đại.
- `Lesson-1-PKI-Certificates.md`: PKI, Certificates
- `Lesson-2-Encryption-Hashing.md`: Cryptography, Hash, HMAC, Symmetric Encryption (AES), Asymmetric Encryption (RSA, ECC)
- `Lesson-3-Signatures.md`: Digital Signatures

### Module 3: Modern API & Tokens (API Hiện đại & Token)
Focuses on how decoupled applications format data and authenticate.
Tập trung vào cách các ứng dụng độc lập định dạng dữ liệu và xác thực.
- `Lesson-1-REST-Basics.md`: REST API Basics, JSON, XML
- `Lesson-2-Token-Standards.md`: JWT, JWS, JWE
- `Lesson-3-OAuth-Terminology.md`: Basic OAuth Terminology

### Module 4: Web Security (Bảo mật Web)
Focuses on identifying, understanding, and mitigating critical vulnerabilities.
Tập trung vào việc xác định, hiểu và giảm thiểu các lỗ hổng nghiêm trọng.
- `Lesson-1-OWASP-Top-10.md`: OWASP Top 10 Overview
- `Lesson-2-Frontend-Attacks.md`: XSS, CSRF, Clickjacking
- `Lesson-3-Session-Attacks.md`: Replay Attack, Session Fixation, Session Hijacking

---

## 3. Interview Questions / Câu hỏi Phỏng vấn
*(Note: As mandated by the Curriculum Generation Rules, every file includes an Interview Questions section. Since this is an architectural README, these questions test the high-level understanding of the prerequisites).*
*(Lưu ý: Theo quy định của Quy tắc Sinh Giáo trình, mọi tệp đều bao gồm phần Câu hỏi Phỏng vấn. Vì đây là tệp README kiến trúc, các câu hỏi này kiểm tra sự hiểu biết cấp cao về các điều kiện tiên quyết).*

**1. Why is it a severe security risk to deploy Keycloak without a Reverse Proxy handling TLS termination?**
**Tại sao việc triển khai Keycloak mà không có Reverse Proxy xử lý việc kết thúc TLS lại là một rủi ro bảo mật nghiêm trọng?**
- **Junior**: Because HTTPS is needed to encrypt passwords. / Vì cần HTTPS để mã hóa mật khẩu.
- **Senior**: Without TLS at the edge, all network traffic containing raw JWTs, authorization codes, and credentials could be intercepted via packet sniffing. A reverse proxy (like Nginx or HAProxy) offloads the expensive TLS handshake, allowing Keycloak to focus on CPU-intensive cryptographic token signing, whilst ensuring secure transport across the public internet.

**2. How does CORS impact a Single Page Application (SPA) authenticating against an Identity Provider?**
**CORS ảnh hưởng như thế nào đến một Ứng dụng Trang Đơn (SPA) khi xác thực với Nhà cung cấp Định danh?**
- **Junior**: CORS blocks cross-domain requests. You must configure Keycloak to allow the SPA's URL. / CORS chặn các yêu cầu chéo miền. Bạn phải cấu hình Keycloak để cho phép URL của SPA.
- **Senior**: When an SPA uses OIDC (e.g., Authorization Code Flow with PKCE) and attempts to exchange the auth code for a token at Keycloak's token endpoint (via AJAX), the browser enforces CORS via a preflight `OPTIONS` request. If Keycloak's Web Origins configuration doesn't explicitly whitelist the SPA's domain, the browser blocks the token response, halting the authentication flow.

**3. What is the fundamental difference between Symmetric and Asymmetric encryption, and where is each used in IAM?**
**Sự khác biệt cơ bản giữa mã hóa Đối xứng và Bất đối xứng là gì, và mỗi loại được sử dụng ở đâu trong IAM?**
- **Junior**: Symmetric uses one key, Asymmetric uses two (public and private). / Đối xứng dùng một khóa, Bất đối xứng dùng hai khóa (công khai và riêng tư).
- **Senior**: Symmetric encryption (like AES) is computationally fast and used for encrypting bulk payload data (e.g., JWE payloads, encrypted session cookies). Asymmetric encryption (like RSA/ECC) is computationally expensive and is used for establishing trust without sharing secrets. In Keycloak, asymmetric private keys digitally sign JWTs (JWS), so downstream resource servers can independently verify the token's authenticity using the public key fetched from the IdP's JWKS endpoint.

**4. If an attacker steals a valid Session Cookie, what attack is this and how do we mitigate it?**
**Nếu kẻ tấn công đánh cắp được Session Cookie hợp lệ, đây là cuộc tấn công gì và chúng ta giảm thiểu nó như thế nào?**
- **Junior**: This is Session Hijacking. We prevent it by using HTTPS. / Đây là Đánh cắp Phiên. Chúng ta ngăn chặn nó bằng cách dùng HTTPS.
- **Senior**: This is Session Hijacking. To mitigate it, cookies must be flagged as `Secure` (only sent over HTTPS), `HttpOnly` (inaccessible via JavaScript to prevent XSS exfiltration), and `SameSite=Strict` or `Lax` (to mitigate CSRF). Additionally, absolute session timeouts, anomaly detection (IP/User-Agent changes), and token binding techniques can be employed to render the stolen cookie useless.

**5. Why are stateless JWTs often preferred over stateful Session Cookies in Microservice architectures?**
**Tại sao JWT không trạng thái thường được ưa chuộng hơn Session Cookie có trạng thái trong kiến trúc Microservice?**
- **Junior**: Because JWTs are modern, stateless, and faster. / Vì JWT hiện đại, không trạng thái và nhanh hơn.
- **Senior**: Stateful cookies require the backend API gateway or individual services to query a centralized session store (like Redis) on every request, creating a single point of failure and a scalability bottleneck. JWTs are self-contained and cryptographically signed. Any microservice can independently verify the user's identity, claims, and permissions offline by checking the signature against the cached public key, enabling massive horizontal scalability and true service decoupling.
