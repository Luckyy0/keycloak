# Lesson 6: Phân loại Ấn bản (Keycloak Editions)

> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Xóa bỏ sự nhầm lẫn giữa hàng loạt cái tên: Upstream, Downstream, Keycloak X, Red Hat SSO. Lựa chọn đúng Ấn bản (Edition) là quyết định ảnh hưởng đến sự Tồn vong và Ngân sách của Công ty khi Lên Production.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

### 1.1. Dòng thác Phần mềm (Upstream vs Downstream)
Mô hình phát triển của Red Hat tuân theo nguyên lý "Dòng thác":
- **Upstream (Thượng nguồn):** Là Dự án MÃ NGUỒN MỞ CỘNG ĐỒNG. Nước ở đây chảy xiết, rất nhiều Tính năng Mới (Feature) được cộng đồng ném vào mỗi ngày. Nhưng nước chưa được lọc kỹ, nhiều sỏi đá (Bug/Lỗi). Tên gọi của dự án Upstream là: **Keycloak (Community Edition)**.
- **Downstream (Hạ nguồn):** Là Sản phẩm THƯỢNG MẠI. Red Hat lấy nước từ Thượng nguồn về, đưa qua nhà máy Lọc nước (Đóng băng Tính năng, Vá lỗi Độc quyền, Kiểm thử hiệu năng 24/7). Nước ở đây siêu sạch, tinh khiết, nhưng chảy chậm (Update tính năng mới chậm hơn Thượng nguồn 1 năm). Tên gọi của nó là: **Red Hat Build of Keycloak (Trước đây gọi là RH-SSO)**.

### 1.2. Keycloak Legacy vs Keycloak.X
Từ năm 2022 trở về trước:
- **Keycloak Legacy:** Chạy trên nhân WildFly App Server. Đuôi cấu hình là `.xml`. (Hiện tại ĐÃ BỊ KHAI TỬ hoàn toàn).
- **Keycloak.X (Hiện nay chỉ gọi ngắn gọn là Keycloak):** Chạy trên nhân Quarkus. Đuôi cấu hình là `.conf`. Tốc độ khởi động siêu việt.

---

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Vòng đời Phát hành (Release Lifecycle) dẫn đến các quyết định của Kiến trúc sư:

```mermaid
graph TD
    subgraph "Dự án Keycloak (Upstream / Miễn phí)"
        KC1[Bản 22.0.0] -->|3 tháng sau| KC2[Bản 23.0.0]
        KC2 -->|3 tháng sau| KC3[Bản 24.0.0]
        Note over KC1,KC3: Cộng đồng liên tục ra bản Phân hệ Mới (Major Version).<br/>Chỉ vá lỗi (Bug fix) cho phiên bản mới nhất.<br/>KHÔNG CÓ khái niệm LTS (Long Term Support).
    end
    
    subgraph "Dự án Red Hat Build of Keycloak (Downstream / Trả tiền)"
        RH1[Bản 22.0.x (LTS)] -->|Vá lỗi Bảo mật| RH2[Bản 22.0.1 LTS]
        RH2 -->|Vá lỗi Bảo mật| RH3[Bản 22.0.2 LTS]
        Note over RH1,RH3: Red Hat khóa cứng ở Bản 22. Không thêm tính năng mới.<br/>Họ chỉ tập trung Sửa Lỗi Bảo Mật cho bản 22 này trong suốt 3 NĂM.<br/>Doanh nghiệp cực kỳ yên tâm, không sợ gãy code khi Update.
    end
    
    KC1 -.->|Copy Source Code| RH1
```

---

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!IMPORTANT]
> **Bài toán Niềm tin (Nên xài bản nào?)**
> - **Ngân hàng, Viễn thông, Quân đội:** BẮT BUỘC mua bản **Red Hat Build of Keycloak**. Chi phí mua gói Support (Hỗ trợ Kỹ thuật) của Red Hat là rất đắt (Hàng chục ngàn USD/năm). Đổi lại: Nếu Keycloak sập lúc 2h sáng, bạn có quyền gọi điện dựng đầu Kỹ sư Red Hat dậy bắt họ sửa. Nếu có Lỗ hổng XSS mới, bạn nhận được Bản Vá Độc Quyền (Private Patch) trước cả thế giới.
> - **SaaS Startups, B2B, Doanh nghiệp Vừa & Nhỏ:** Dùng bản **Keycloak Miễn Phí (Upstream)**. Bản miễn phí có ĐẦY ĐỦ 100% TÍNH NĂNG của bản trả tiền (Thậm chí có trước 1 năm). Tuy nhiên, bạn phải Sở Hữu một Đội Ngũ DevOps cứng cựa. Nếu sập, bạn Tự Lên Google tra lỗi, không ai cứu bạn. Cứ mỗi 3-6 tháng, bạn phải Tự Kiểm Tra Code và Lên lịch Cập nhật (Migration) lên phiên bản mới nhất để lấy bản vá bảo mật.

> [!CAUTION]
> **Thảm họa `:latest` trong Docker**
> Tuyệt đối KHÔNG BAO GIỜ deploy Keycloak trên Production bằng Image Tag: `quay.io/keycloak/keycloak:latest`.
> Vì Keycloak Community cập nhật Major Version liên tục. Nếu Server bạn Restart, nó sẽ kéo bản `25.0` đè lên bản `24.0`. Data Database không tương thích ngược sẽ lập tức NỔ TUNG. BẮT BUỘC phải ghim cứng phiên bản (Ví dụ: `quay.io/keycloak/keycloak:24.0.1`).

---

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Sự khác biệt về Registry Tải Image Docker:

- Tải bản Miễn phí (Chạy nền Quarkus):
  `docker pull quay.io/keycloak/keycloak:24.0.1`
- Tải bản Trả phí (Red Hat Build of Keycloak):
  `docker pull registry.redhat.io/rhbk/keycloak-rhel9:24.0`
  *(Lưu ý: Để tải Image từ `registry.redhat.io`, bạn bắt buộc phải có Tài khoản Red Hat Customer Portal đã đăng ký License xịn, nếu không Docker sẽ báo lỗi `unauthorized: authentication required`).*

---

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Sát nhập/Chuyển đổi (Migration từ Community sang Enterprise):** Giả sử lúc đầu Startup nghèo, xài bản Miễn phí. Giờ có tiền, muốn mua gói Support của Red Hat. Việc chuyển đổi (Migration) Database từ bản `quay.io` sang bản `registry.redhat.io` CÓ ĐƯỢC HAY KHÔNG?
  - **Câu trả lời:** Rất may mắn là ĐƯỢC, vì chung 1 gốc. Nhưng bạn PHẢI TÌM MUA ĐÚNG Phiên bản Red Hat tương đương. (Ví dụ Data đang chạy Keycloak 22 Miễn phí, thì phải Migrate sang RHBK 22). Nếu phiên bản lệch, quá trình Migration Data Schema sẽ dính lỗi chí mạng.

---

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**1. Trong thế giới Mã Nguồn Mở, tại sao Keycloak Community lại Cố Tình KHÔNG PHÁT HÀNH bản vá lỗi (Patch) LTS dài hạn cho các phiên bản cũ? (Ví dụ phát hiện lỗi ở bản 22, nhưng họ chỉ tung bản vá ở bản 24 mới nhất).**
- **Junior:** Vì họ không có thời gian code bản cũ.
- **Senior:** Đây là Chiến Lược Ép Buộc (Upstream Strategy). Đội ngũ Keycloak (Được trả lương bởi Red Hat) không muốn tốn nguồn lực duy trì Code cũ. Họ ÉP CỘNG ĐỒNG phải: Một là Liên tục Nâng cấp lên bản Mới Nhất để hưởng công nghệ mới. Hai là: Cảm thấy mệt mỏi với việc Cập nhật liên tục? Hãy bỏ tiền ra mua Bản Doanh Nghiệp (RHBK) để chúng tôi nuôi đội ngũ LTS bảo trì phiên bản cũ 3 năm cho bạn. 

**2. Nếu tôi thuê một đối tác bên ngoài Code một cái Custom Plugin (Ví dụ Quét FaceID) cho Keycloak của tôi. Plugin đó có được Red Hat hỗ trợ (Support) nếu tôi mua bản Quyền RHBK không?**
- **Junior:** Mua bản quyền là họ hỗ trợ hết.
- **Senior:** TUYỆT ĐỐI KHÔNG. Gói Hỗ trợ Doanh nghiệp của Red Hat có một "Ranh giới" cực kỳ khắc nghiệt.
Họ CHỈ HỖ TRỢ các Core Features (Tính năng Lõi) do chính Kỹ sư Red Hat viết ra. Mọi Custom Providers (Java JAR files) do công ty bạn tự viết thêm, hoặc bất kỳ Template HTML/CSS giao diện nào bạn tự sửa, Red Hat sẽ từ chối trách nhiệm (Unsupported). Nếu hệ thống sập do cái Plugin FaceID của bạn cắn hết RAM, Red Hat sẽ đóng Ticket Support ngay lập tức. Đây là bài học xương máu khi thiết kế Kiến trúc: Hạn chế tối đa việc Code Custom Plugin nếu muốn hưởng đặc quyền Enterprise.

**3. Công ty tôi có 1 Server chạy App Kế toán, hoàn toàn không có kết nối Internet (Air-gapped). Làm sao tôi tải Image Docker của Red Hat về cài?**
- **Junior:** Chắc phải dùng USB copy.
- **Senior:** Red Hat cung cấp giải pháp **Offline Repository (Kho lưu trữ Ngoại tuyến)**.
Bằng một Máy chủ Trung Gian CÓ INTERNET (Bastion Host), bạn cài đặt Jfrog Artifactory hoặc Nexus. Máy Bastion này sẽ dùng Token Red Hat tải Image RHBK về. Sau đó, nó Sync (Đồng bộ) toàn bộ Image đó qua Cổng mạng nội bộ vắt sang Máy chủ Kế Toán. Bạn trỏ cấu hình Docker trên Máy chủ Kế toán vào cái Registry Nội Bộ (Nexus) đó thay vì gọi ra `registry.redhat.io`. (Pull-Through Cache Architecture).

---

## 7. Tài liệu tham khảo (References)
- **Keycloak Blog:** Moving to Quarkus.
- **Red Hat Documentation:** Red Hat Build of Keycloak Supported Configurations.
