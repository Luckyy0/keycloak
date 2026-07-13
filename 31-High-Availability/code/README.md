# Hướng Dẫn Chạy Môi Trường Cluster Bất Tử (HA Keycloak)

Thư mục này chứa cấu hình Docker Compose để khởi chạy một Hệ Sinh Thái Cân Bằng Tải chịu lỗi cao: 1 NGINX, 2 Keycloak Nodes (KC1, KC2), và 1 PostgreSQL.

## 1. Cấu trúc thư mục

```text
code/
├── docker-compose.yml     # File khởi động Hệ sinh thái
├── nginx.conf             # Cấu hình Cân bằng tải IP Hash cho NGINX
└── README.md              # File hướng dẫn này
```

## 2. Cách Vận Hành Và Khám Phá

1. Mở terminal tại thư mục `code/` và gõ: `docker-compose up -d`.
2. Theo dõi log khởi động của Node 1 để thấy chúng Bắt Tay Nhau:
   `docker logs -f code-kc1-1` (Bấm Ctrl+C để thoát xem log). 
   Hãy tìm dòng chữ `ISPN000094: Received new cluster view` để chắc chắn Infinispan đã lập cụm thành công.
3. **Mở Trình Duyệt**, truy cập thẳng vào NGINX (Không cần gõ port vì mặc định là 80): 
   `http://localhost/`
   Đăng nhập bằng tài khoản `admin` / `admin`. Tạo thêm User mới.
4. **Kiểm Tra Trải Nghiệm Khách Hàng Bất Tử:**
   - Mở cửa sổ Ẩn Danh. Đăng nhập vào trang quản trị Account bằng User vừa tạo: `http://localhost/realms/master/account/`.
   - Vẫn giữ nguyên cửa sổ Ẩn Danh đó, Quay lại Terminal chạy lệnh BẮN BỎ NODE 1:
     `docker stop code-kc1-1`
   - Chờ khoảng 3 giây. Quay lại cửa sổ Ẩn Danh, bấm F5 hoặc click sang các Tab khác. Bạn sẽ thấy MỌI THỨ VẪN HOẠT ĐỘNG!
   - Khách Hàng không hề bị văng ra (Log out) vì Session của họ đã được tự động chép sang Node 2 và NGINX đã tự động chuyển hướng gói tin (Failover).

## Lưu ý về Cân Bằng Tải Thực Tế
Trong cấu hình Nginx Lab này, chúng ta dùng `ip_hash` vì bản Nginx Miễn Phí không hỗ trợ Cắm Cờ Cookie Dính (Sticky Cookie). Ở môi trường Production thực tế, nếu bạn chạy trên Kubernetes, hãy dùng Ingress NGINX (Có hỗ trợ Nginx Ingress Cookie Affinity) hoặc dùng AWS ALB, HAProxy để cắm cờ Cookie (Ví dụ: cờ `AUTH_SESSION_ID.node1`) nhằm định tuyến chính xác 100% Khách Hàng về đúng Server cũ của họ!
