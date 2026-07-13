> [!NOTE]
> **Category:** Theory / Architecture
> **Goal:** Nắm bắt kiến trúc mạng cấp thấp đằng sau cơ chế Clustering của Keycloak thông qua thư viện JGroups, cũng như phân tích các thuật toán Tìm kiếm điểm nút (Discovery / PING) phù hợp cho từng hạ tầng mạng.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong hệ sinh thái Java / WildFly / Quarkus, khi nhắc tới Lưới dữ liệu phân tán (Distributed Data Grid), chúng ta nói đến hai tầng công nghệ kết hợp với nhau:
1. **Infinispan:** Chịu trách nhiệm về logic (Data Layer). Nó quyết định lưu cái gì, xóa cái gì, dùng thuật toán Băm (Hash) nào.
2. **JGroups:** Chịu trách nhiệm về vận chuyển và liên lạc (Network Layer). Nó nằm ngay bên dưới Infinispan. 

Trách nhiệm cốt lõi của JGroups bao gồm:
- **Membership (Tư cách thành viên):** Làm sao để khi một Node A vừa bật lên, nó biết được trong mạng đang có Node B, C để xin tham gia (Join) thành một cụm (Cluster). Quá trình này gọi là Discovery (Khám phá).
- **Group Communication (Giao tiếp Nhóm):** Đảm bảo tin nhắn (Message) gửi từ Node A đến Node B, C phải tới nơi an toàn (Reliable transmission), đúng thứ tự (Ordering).
- **Failure Detection (Phát hiện Lỗi):** Liên tục theo dõi (Ping/Heartbeat) để loại bỏ ngay lập tức một Node khỏi cụm nếu nó bị sập nguồn hoặc đứt cáp mạng.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Quá trình "Khám phá" (Discovery) là bước tối quan trọng khi khởi động cụm. JGroups hỗ trợ nhiều giao thức khác nhau để xử lý việc này:

```mermaid
flowchart TD
    subgraph Discovery Mechanisms (PING)
        UDP[UDP_PING / MPING\nMulticast: Gửi quảng bá tới toàn LAN\nChỉ dùng cho Local/On-Premise]
        TCP[TCPPING\nUnicast: Khai báo tĩnh danh sách IP\nKhó Auto-scale]
        JDBC[JDBC_PING\nDatabase: Lưu IP vào bảng `JGROUPSPING`\nHoạt động mọi nơi]
        DNS[DNS_PING / KUBE_PING\nDành riêng cho Kubernetes\nTruy vấn DNS / API Server]
    end

    NodeA[Keycloak Node A\nBắt đầu khởi động]
    NodeA --> |1. Dùng giao thức đã chọn| Discovery Mechanisms
    Discovery Mechanisms --> |2. Tìm thấy Node B, C| NodeA
    NodeA <--> |3. Gửi thông điệp JOIN| NodeB[(Keycloak Cluster\nCoordinator: Node B)]
    NodeB --> |4. Chấp nhận & Cập nhật View| NodeA
```

**Cơ chế cấp thấp của một số giao thức phổ biến:**
- **UDP_PING:** Hoạt động giống như việc bạn đứng giữa quảng trường và hét lên "Có ai là Keycloak ở đây không?". Các Node khác nghe thấy qua địa chỉ Multicast sẽ đáp lại. Nhược điểm chí mạng là các môi trường Đám mây (AWS, Azure) hoặc Container (Docker) hiện đại đều **Chặn** (block) gói tin Multicast để chống bão mạng.
- **JDBC_PING:** Biến Cơ sở dữ liệu (đã kết nối sẵn của Keycloak) thành một "Bảng thông báo". Khi Node A khởi động, nó ghi IP của mình vào bảng `JGROUPSPING`. Khi Node B khởi động, nó đọc bảng đó và chủ động kết nối TCP với IP của Node A.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

- **Tránh dùng mặc định trên Đám mây:** Keycloak mặc định sử dụng UDP_PING (hoặc MPING). Nếu bạn triển khai trên Cloud hoặc Docker Network, cụm sẽ không bao giờ được hình thành (Mỗi Node sẽ hoạt động như một cụm độc lập 1 thành viên). Bắt buộc phải chuyển đổi giao thức PING.
- **Khuyến nghị cho Bare-Metal/VMs:** **JDBC_PING** là phương pháp ổn định, dễ cấu hình và tương thích nhất khi bạn tự chạy Keycloak bằng tệp nhị phân hoặc Docker Compose thông thường, vì nó tận dụng chính Database PostgreSQL/MySQL sẵn có.
- **Khuyến nghị cho Kubernetes:** Sử dụng **DNS_PING**. Kubernetes tạo ra các "Headless Service" quản lý danh sách IP động của các Pod. DNS_PING chỉ cần hỏi DNS nội bộ của Kubernetes là lấy được toàn bộ IP của các Node Keycloak mà không cần cấu hình tĩnh.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Keycloak Quarkus cho phép thay đổi giao thức JGroups thông qua cờ `--jgroups-stack`. Tuy nhiên, để cấu hình chi tiết (ví dụ: JDBC_PING), bạn cần cung cấp một file cấu hình XML riêng cho JGroups.

**Cấu hình mẫu khởi chạy với JDBC_PING:**
1. Khởi chạy với file cấu hình JGroups:
```bash
bin/kc.sh start --cache-config-file=cache-ispn.xml \
  --jgroups-config-file=default-jdbc-ping.xml
```
2. Trong đó file `default-jdbc-ping.xml` thay thế lớp `<MPING/>` bằng `<JDBC_PING/>`:
```xml
<config xmlns="urn:org:jgroups">
    <!-- Giao tiếp bằng TCP (Unicast) ổn định -->
    <TCP bind_port="7800" ... />
    
    <!-- Sử dụng JDBC_PING để khám phá IP thay cho UDP/MPING -->
    <JDBC_PING connection_driver="org.postgresql.Driver"
               connection_url="jdbc:postgresql://postgres-db:5432/keycloak"
               connection_username="keycloak"
               connection_password="password"
               initialize_sql="CREATE TABLE IF NOT EXISTS JGROUPSPING (own_addr varchar(200) NOT NULL, cluster_name varchar(200) NOT NULL, ping_data bytea, constraint PK_JGROUPSPING PRIMARY KEY (own_addr, cluster_name))"/>
    
    <!-- Các giao thức ghép nối (MERGE), báo lỗi (FD_ALL) ... -->
    <MERGE3 />
    <FD_SOCK />
    <FD_ALL timeout="40000" interval="8000" />
</config>
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Chia rẽ Cụm (Split-Brain) do FD (Failure Detection) chậm:** Giao thức `FD_ALL` và `FD_SOCK` có nhiệm vụ liên tục PING các máy chủ để kiểm tra xem nó còn sống không. Nếu xảy ra hiện tượng Nghẽn mạng giả (Network Jitter) hoặc CPU của máy đích bị giật (Spike), JGroups có thể lầm tưởng máy đó đã chết và loại bỏ nó. Khi mạng bình thường lại, hai máy sẽ tạo thành 2 Cluster độc lập, dẫn đến "Hội chứng phân chia não" (Split-Brain). Cấu hình `timeout` phải đủ dài để chịu đựng độ trễ mạng tạm thời.
- **Rác JGROUPSPING (Stale Data):** Trong JDBC_PING, nếu một Node bị sập tắt nguồn đột ngột (Hard crash) mà không kịp gửi thông báo Shutdown, địa chỉ IP của nó vẫn nằm vĩnh viễn trong bảng Database `JGROUPSPING`. Các Node sau khởi động sẽ liên tục cố kết nối vào "bóng ma" này. (Tuy nhiên, JGroups có cơ chế dọn dẹp các node không hồi đáp sau một thời gian).

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **(Junior)** Mục đích của bộ thư viện JGroups trong kiến trúc Keycloak Cluster là gì?
   - *Đáp án:* JGroups là lớp mạng chịu trách nhiệm tìm kiếm các máy chủ khác trong cụm (Discovery) và quản lý giao tiếp tin cậy (nhắn tin/đồng bộ) giữa các máy chủ đó cho Infinispan.

2. **(Junior)** Khi nào thì phương thức mặc định UDP_PING bị vô hiệu hóa?
   - *Đáp án:* Khi chạy trên các nền tảng đám mây (AWS, Azure, GCP) hoặc bên trong mạng Container vì các môi trường này thường chặn các gói tin Multicast.

3. **(Senior)** Lợi ích lớn nhất của JDBC_PING so với TCPPING truyền thống là gì?
   - *Đáp án:* TCPPING yêu cầu bạn phải cấu hình cứng (hard-code) danh sách IP của tất cả các Node. Nó khiến việc tự động mở rộng (Auto-scaling) bất khả thi. JDBC_PING giải quyết việc đó bằng cách dùng Database trung tâm để cấp phát và trao đổi danh sách IP động.

4. **(Senior)** Trong Kubernetes, tại sao người ta ưu tiên DNS_PING hoặc KUBE_PING thay vì dùng JDBC_PING?
   - *Đáp án:* Dùng JDBC_PING sẽ gây thêm tải thừa lên Database không cần thiết. K8s đã có sẵn một cơ chế quản lý danh sách IP của Pods rất xuất sắc thông qua DNS của Headless Service hoặc API Server. DNS_PING tận dụng chính hạ tầng nội tại này nhanh hơn và an toàn hơn.

5. **(Senior)** Tham số `FD_ALL` (Failure Detection) hoạt động như thế nào và tại sao cấu hình `timeout` quá ngắn lại nguy hiểm?
   - *Đáp án:* `FD_ALL` liên tục gửi các Heartbeat/Ping giữa các Node. Nếu timeout quá ngắn (ví dụ 2 giây), một hiện tượng nghẽn mạng nhỏ hoặc thời gian "stop-the-world" do Java Garbage Collection (GC) quét bộ nhớ cũng sẽ khiến Node bị đánh dấu là "đã chết" (Dead) và loại khỏi Cluster một cách oan uổng.

## 7. Tài liệu tham khảo (References)

- [JGroups Official Manual](http://www.jgroups.org/manual/index.html)
- [Keycloak Guide: Configuring JGroups Stack](https://www.keycloak.org/server/caching)
- [WildFly - High Availability and JGroups configurations](https://docs.wildfly.org/24/High_Availability_Guide.html)
