# Chapter 33: Nghệ Thuật Thay Máu (Upgrades & Migrations)

## Giới thiệu (Introduction)
Red Hat phát hành phiên bản Keycloak mới với tốc độ chóng mặt (Khoảng 3-4 tháng 1 bản Major). 
Nếu bạn đứng yên ở một phiên bản cũ (Ví dụ Bản 17), bạn sẽ bỏ lỡ các tính năng đột phá, bị chậm chạp, và tệ nhất là: Bị Hacker đục thủng vì các Lỗ Hổng Bảo Mật (CVE) không được vá!
Nhưng Nâng cấp một hệ thống Xác Thực đang chạy thật sự là một cuộc Đại Phẫu Thuật Tim. Bạn không thể cứ thế mà Tắt Máy Chủ, Cài Bản Mới, và Mở Lên! Nếu Database bị vỡ cấu trúc thì sao? Nếu Code SPI cũ bị lỗi biên dịch thì sao? 
Chương này sẽ trang bị cho bạn bản lĩnh của một Kiến Trúc Sư: Từ việc đánh giá tính tương thích của Code cũ, hiểu cơ chế Liquibase tự động mổ xẻ Database của Keycloak, cho đến Tuyệt Kỹ "Zero-Downtime Upgrade" (Nâng cấp máy bay ngay khi đang bay mà khách hàng không rớt mạng).

## Mục lục (Table of Contents)

### Module 1: Cuộc Phẫu Thuật Đổi Đời (Concepts)
*   **Lesson 1: Version Compatibility (Rào Cản Phiên Bản):** Phân tích rủi ro khi nâng cấp. Chữ "Backwards Compatibility" của Red Hat có thật sự đáng tin? Hướng dẫn cách xử lý khi các File Theme Tự Chế (Freemarker) và Code SPI (Java) cũ bị gãy vụn ở bản mới.
*   **Lesson 2: Database Migration (Nâng Cấp Xương Sống):** Khám phá công cụ Liquibase được nhúng ngầm bên trong quá trình khởi động. Tại sao bạn KHÔNG BAO GIỜ được phép hạ cấp (Downgrade) Keycloak nếu Database đã bị Nâng Cấp? Khái niệm Backup trước giờ G.
*   **Lesson 3: Zero-Downtime Upgrades (Thay Máu Không Ngừng Thở):** Kỹ thuật đỉnh cao của HA. Làm sao để nâng cấp cụm 2 Node Keycloak V24 lên V25? Tắt Node 1, Nâng cấp Node 1, Mở Node 1, Tắt Node 2, Nâng Cấp Node 2 (Rolling Upgrade). Giải quyết xung đột Session Cache khi 2 Node chạy 2 phiên bản khác nhau.

### Labs & Thực hành (Labs)
*   **Lab 1:** Thực Hành Nâng Cấp Sinh Tử (Migration Lab). Bạn sẽ khởi động một cụm Keycloak Bản CŨ (Ví dụ 23.0.0). Nhét dữ liệu vào. Sau đó thực hiện Kịch bản Đổi Image Docker sang bản MỚI (Ví dụ 24.0.1) và theo dõi xem Database được biến đổi kỳ diệu như thế nào qua Log Của Liquibase.

## Bắt đầu từ đâu? (Where to start?)
Trang bị Kiến thức phòng thủ trước khi bấm nút Update tại [Lesson 1: Version Compatibility](Module-1-Concepts/Lesson-1-Version-Compatibility.md).
