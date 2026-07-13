> [!NOTE]
> **Category:** Theory / Architecture
> **Goal:** Hiểu chuyên sâu về kiến trúc Cụm (Clustering) trong Keycloak và vai trò cốt lõi của hệ thống lưới dữ liệu bộ nhớ (In-memory Data Grid) Infinispan trong việc đảm bảo Tính sẵn sàng cao (High Availability) cho phiên đăng nhập.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Một máy chủ Keycloak độc lập (Standalone) chứa một điểm yếu chí mạng: Điểm lỗi duy nhất (Single Point of Failure - SPOF). Nếu máy chủ này khởi động lại hoặc gặp sự cố phần cứng, toàn bộ hệ thống SSO sẽ ngừng hoạt động. Khái niệm **Tính Sẵn sàng Cao (High Availability - HA)** giải quyết vấn đề này bằng cách chạy nhiều máy chủ (Nodes) Keycloak đồng thời.

Tuy nhiên, Keycloak **KHÔNG PHẢI là một ứng dụng không trạng thái (Stateless)** theo nghĩa hoàn hảo. Để cung cấp tốc độ phản hồi tính bằng mili-giây, Keycloak không liên tục truy vấn Cơ sở dữ liệu (Database) cho mọi thao tác. Thay vào đó, nó duy trì một trạng thái trong bộ nhớ RAM, đặc biệt là **Phiên người dùng (User Sessions)**, **Phiên đăng nhập phân tán (Action Tokens)**, và **Bộ đệm dữ liệu (User/Realm Cache)**.

Khi bạn có nhiều hơn một Node, bạn phải đối mặt với bài toán đồng bộ hóa trạng thái bộ nhớ này. Nếu User A đăng nhập ở Node 1 (và Node 1 lưu Session vào RAM), sau đó Load Balancer đẩy Request tiếp theo của User A sang Node 2, thì Node 2 sẽ không biết User A là ai, dẫn đến việc bắt User A đăng nhập lại. 

Để giải quyết, Keycloak nhúng (embed) một công nghệ của JBoss gọi là **Infinispan**. Đây là một kho dữ liệu NoSQL phân tán hoạt động hoàn toàn trên RAM. Các Node Keycloak sử dụng Infinispan để "nói chuyện" với nhau, đồng bộ hóa và sao chép (replicate) các phiên đăng nhập thành một lưới dữ liệu bộ nhớ (Data Grid) chung.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Infinispan sử dụng các cấu trúc liên kết mạng linh hoạt để chia sẻ dữ liệu giữa các Node. Sơ đồ dưới đây thể hiện cơ chế đồng bộ theo kiểu Phân tán (Distributed Mode):

```mermaid
flowchart TD
    subgraph Load Balancer Layer
        LB[Load Balancer]
    end

    subgraph Keycloak Cluster
        K1[Node 1\n(Infinispan Cache)]
        K2[Node 2\n(Infinispan Cache)]
        K3[Node 3\n(Infinispan Cache)]
    end
    
    DB[(PostgreSQL Database)]

    LB --> |User 1 Đăng nhập| K1
    K1 --> |1. Tạo Session A trong RAM| K1
    K1 <-->|2. Infinispan đồng bộ Session A (Số bản sao = 2)| K2
    
    LB --> |User 1 Request tiếp theo| K3
    K3 -.-> |3. Không có Session A? Hỏi Infinispan| K1
    K3 -.-> |Hoặc hỏi Node giữ bản sao| K2
    
    K1 ==> DB
    K2 ==> DB
    K3 ==> DB
```

**Các chế độ đồng bộ hóa (Cache Modes) cấp thấp trong Infinispan:**
- **Local:** Dữ liệu chỉ nằm ở RAM của Node đó (Dùng cho môi trường 1 node).
- **Replicated (Nhân bản):** Mỗi khi tạo ra 1 phiên đăng nhập, phiên đó được copy tới bộ nhớ của *tất cả* các Node khác. Rất an toàn (sập n-1 node vẫn còn data), nhưng mở rộng kém. 10 node thì 1 session phải tốn bộ nhớ nhân 10 lần và tốn băng thông mạng khủng khiếp.
- **Distributed (Phân tán - Mặc định cho Sessions):** Keycloak sử dụng thuật toán Băm (Hashing) để quyết định chia nhỏ các session. Một thông số quan trọng là `owners` (Mặc định: 2). Nghĩa là 1 session mới sẽ chỉ được copy sang đúng 2 Node trong toàn cụm, bất kể cụm đó có 3 hay 100 node. Giúp cân bằng hoàn hảo giữa khả năng chịu lỗi và tiết kiệm RAM.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

- **Số lượng Owners (Owners Number):** Đối với các phiên (Sessions), số lượng `owners=2` là cấu hình tối ưu nhất cho phần lớn doanh nghiệp. Nó có nghĩa là nếu 1 node chết đi, một node khác trong cụm vẫn giữ được bản sao dự phòng của các người dùng, và quá trình khôi phục được diễn ra âm thầm mà không bắt người dùng đăng nhập lại.
- **Cô lập Mạng ngang hàng (Network Isolation):** Luồng giao tiếp của Infinispan giữa các node (Data Plane) không bao giờ được phép mở ra Internet. Nếu kẻ tấn công có thể truy cập được các port liên lạc của cụm, chúng có thể inject dữ liệu độc hại vào bộ đệm của toàn bộ hệ thống. Nên cấu hình Infinispan chạy trên một thẻ mạng riêng ảo (Private VPC).
- **Tránh Tràn Bộ Nhớ (OOM - Out of Memory):** Infinispan lưu mọi thứ trên RAM. Với hàng triệu phiên đăng nhập rác (do User không logout mà chỉ tắt trình duyệt), Node sẽ bị tràn RAM và sập. Bắt buộc phải cấu hình chính sách dọn dẹp bộ đệm (Eviction Policy / Max Size) kết hợp với Lifespan (Thời gian sống) để tự động xóa các session cũ.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

Kể từ các phiên bản sử dụng Quarkus, Keycloak kích hoạt chế độ cụm tự động nếu bạn cung cấp cờ `--cache=ispn`. Mặc định Keycloak cung cấp file cấu hình `cache-ispn.xml`. Bạn có thể tùy chỉnh lại nó và cung cấp khi chạy lệnh start.

**Đoạn trích cấu hình `cache-ispn.xml` quy định số chủ sở hữu (owners):**

```xml
<infinispan>
    <cache-container name="keycloak">
        <transport lock-timeout="60000"/>
        
        <!-- Cấu hình Distributed Cache cho sessions -->
        <distributed-cache name="sessions" owners="2">
            <!-- Cấu hình Lifespan để tự động dọn dẹp rác, 
                 nhưng thường Keycloak ghi đè thông số này ở Realm Settings -->
            <expiration lifespan="-1"/> 
        </distributed-cache>

        <!-- Cấu hình Local Cache cho realm (Vì cấu hình Realm hiếm khi đổi và cần đọc siêu tốc) -->
        <local-cache name="realms">
            <!-- Giữ tối đa 10,000 realm/clients trên bộ nhớ RAM -->
            <memory max-count="10000"/>
        </local-cache>
    </cache-container>
</infinispan>
```

Khởi chạy máy chủ với file cấu hình:
```bash
bin/kc.sh start --cache=ispn --cache-config-file=conf/my-cache-ispn.xml
```

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Hội chứng phân chia não (Split-Brain Syndrome):** Trong môi trường phân tán, nếu hạ tầng mạng bị đứt gãy làm đôi (Network Partition), cụm 4 Node có thể bị tách làm hai cụm 2 Node, và chúng không nhìn thấy nhau. Mỗi bên sẽ bắt đầu tự vận hành và dẫn đến sai lệch dữ liệu phân tán (Split-brain). Infinispan xử lý điều này qua cơ chế phân xử (Quorum / Partition Handling).
- **Độ trễ Invalidate Dữ liệu (Invalidation Latency):** Khi một quản trị viên xóa một cấu hình trong Realm, thay vì đồng bộ nguyên cục dữ liệu mới, Infinispan phát đi một thông điệp "Hủy bỏ dữ liệu cũ" (Invalidation Message) đến toàn bộ cụm để các node khác tải lại từ Database. Nếu mạng có độ trễ, một vài Node có thể phản hồi lại thông tin cũ cho người dùng trong vòng vài chục mili-giây.

## 6. Câu hỏi Phỏng vấn (Interview Questions)

1. **(Junior)** Tại sao Keycloak không trực tiếp lưu trữ tất cả vào Database mà phải dùng hệ thống bộ nhớ phân tán Infinispan?
   - *Đáp án:* Do vấn đề về hiệu năng. Database quan hệ (như PostgreSQL) không được thiết kế để xử lý hàng vạn tác vụ đọc/ghi session liên tục mỗi giây. Infinispan đọc/ghi trên RAM với độ trễ siêu thấp, giúp Keycloak đạt chuẩn High Availability.

2. **(Junior)** Distributed Cache khác với Replicated Cache ở Infinispan như thế nào?
   - *Đáp án:* Replicated chép dữ liệu lên tất cả các node trong cụm. Distributed dùng thuật toán Băm (Hash) để chia dữ liệu, và chỉ chép dữ liệu cho một số lượng node nhất định (do thuộc tính `owners` quyết định), nhằm tiết kiệm RAM nhưng vẫn chịu lỗi tốt.

3. **(Senior)** Nếu thiết lập tham số `owners=1` trong cấu hình Distributed Session Cache, điều gì sẽ xảy ra với User?
   - *Đáp án:* `owners=1` đồng nghĩa với việc không có sao lưu dự phòng. Nếu Node đang lưu session bị sập đột ngột, toàn bộ phiên đăng nhập của người dùng nằm trên node đó sẽ bị xóa mất vĩnh viễn, người dùng đang dùng hệ thống sẽ lập tức bị bắt đăng nhập lại.

4. **(Senior)** Trong quá trình Scale Out (tăng số lượng Node từ 3 lên 10), Keycloak có thể bị chậm đi trong vài phút đầu. Tại sao lại xảy ra hiện tượng đó?
   - *Đáp án:* Do cơ chế Tái cân bằng trạng thái (State Rebalancing) của Infinispan. Khi có Node mới tham gia, Infinispan phải tính toán lại hệ số băm (Hash) và di chuyển/chép các session từ các node cũ sang node mới qua mạng nội bộ để cân bằng tải bộ nhớ, dẫn đến việc ngốn CPU và Băng thông mạng.

5. **(Senior)** Hãy giải thích cơ chế của "Invalidation Cache" được áp dụng cho bảng Realms/Users trong Keycloak, nó hoạt động khác với Session Cache ra sao?
   - *Đáp án:* Realms/Users là dữ liệu tĩnh, tần suất cập nhật thấp. Keycloak lưu nó ở Local Cache của từng Node. Khi cập nhật, thay vì đẩy toàn bộ cấu hình mới cho các Node khác, Infinispan chỉ dùng cơ chế Invalidation — gửi một tin nhắn báo hiệu "Bản ghi ID=123 đã lỗi thời", yêu cầu các Node tự động xóa bộ nhớ của ID đó và tải lại trực tiếp từ Database khi có request tới.

## 7. Tài liệu tham khảo (References)

- [Keycloak High Availability Guide](https://www.keycloak.org/high-availability/concepts-threads)
- [Infinispan Distributed Cache Documentation](https://infinispan.org/docs/stable/titles/configuring/configuring.html#distributed-caches)
- [JBoss / Infinispan Architecture](https://access.redhat.com/documentation/en-us/red_hat_data_grid/)
