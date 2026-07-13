# Chapter 31: Tính Sẵn Sàng Cao (High Availability - HA)

## Giới thiệu (Introduction)
Chào mừng bạn đến với Lãnh Địa Của Sức Chịu Đựng (Resilience). 
Từ đầu khóa học đến giờ, chúng ta chỉ luôn chạy Keycloak trên MỘT MÁY CHỦ DUY NHẤT (Single Node). Nhưng ở môi trường Production thật (như Ngân Hàng, Viễn Thông), nếu máy chủ đó cháy ổ cứng hoặc sụp nguồn thì sao? Toàn bộ Khách Hàng của Công Ty sẽ không thể đăng nhập! Hàng triệu Đô La bốc hơi mỗi phút!

Đây là lúc bạn phải học cách nhân bản (Scale) Keycloak thành nhiều Máy Chủ chạy song song (Cluster). Khi một Máy Chết, máy kia lập tức gánh vác mà Khách Hàng không hề hay biết. 
Tuy nhiên, chạy nhiều Máy Chủ Keycloak Cùng Lúc LÀ MỘT CƠN ÁC MỘNG KỸ THUẬT NẾU BẠN KHÔNG HIỂU BẢN CHẤT CỦA NÓ. 
Chương này sẽ hướng dẫn bạn bí thuật "Bắt Tay Nhau" (Clustering) bằng công nghệ Infinispan, cách tối ưu hóa Database Connection, và thiết lập một Bộ Cân Bằng Tải (Load Balancer) chuyên nghiệp chặn trước mũi Keycloak.

## Mục lục (Table of Contents)

### Module 1: Xây Dựng Boong Ke Bất Tử (Concepts)
*   **Lesson 1: Clustering & Infinispan (Trái Tim Đồng Bộ):** Khám phá công nghệ Cache Phân Tán Infinispan giấu kín bên trong Keycloak. Cách mà 2 Node Keycloak ở 2 máy tính khác nhau có thể truyền dữ liệu Session (Phiên Đăng Nhập) cho nhau qua đường truyền Mạng LAN mà không cần ghi xuống Database.
*   **Lesson 2: Database Tuning (Ép Xung Trái Tim PostgreSQL):** Khi có nhiều Node Keycloak cùng cắm vòi hút vào một con Database, hiện tượng Nghẽn Cổ Chai (Bottleneck) sẽ xảy ra. Bài này hướng dẫn bạn chỉnh Connection Pool (Agroal) sao cho tối ưu nhất.
*   **Lesson 3: Load Balancing (Bộ Chia Bài Công Lý):** Cấu hình NGINX hoặc HAProxy đứng trước Cụm Keycloak. Giải quyết bài toán Kinh Điển: "Làm sao để biết Khách A đang ở Node 1, để đẩy Request tiếp theo của nó vào đúng Node 1?" (Sticky Sessions vs Sticky Cookies).

### Labs & Thực hành (Labs)
*   **Lab 1:** Xây Dựng Cụm Cluster Đích Thực: Viết file `docker-compose.yml` khởi động 1 NGINX Load Balancer, 2 Node Keycloak (KC1, KC2), và 1 DB PostgreSQL. Chứng kiến phép thuật: Tắt nóng KC1, và Khách Hàng vẫn đang lướt Web bình thường bằng KC2!

## Bắt đầu từ đâu? (Where to start?)
Tiến thẳng vào Trái tim của bài toán Đa Máy Chủ với [Lesson 1: Clustering & Infinispan](Module-1-Concepts/Lesson-1-Clustering-Infinispan.md)!
