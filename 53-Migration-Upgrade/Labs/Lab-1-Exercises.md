> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Trang bị kỹ năng thực tế về cách sao lưu (Export), phục hồi (Import) một Realm và mô phỏng quá trình sao lưu Database an toàn theo đúng tiêu chuẩn doanh nghiệp.

## 1. Kịch bản Thực hành (Lab Scenario)

Bạn là DevOps Engineer. Dev team vừa cấu hình xong toàn bộ thông số phân quyền, Client, và Custom Mappers cho dự án "Alpha" trên máy chủ Keycloak môi trường Development. Giám đốc kỹ thuật yêu cầu bạn:
1. Trích xuất (Export) toàn bộ cấu hình (Không bao gồm Users) của dự án này ra file JSON.
2. Nạp (Import) file cấu hình này vào một Realm mới (đóng vai trò là môi trường Staging).
3. Viết và chạy một script sao lưu (Backup) cơ sở dữ liệu mô phỏng theo chuẩn an toàn.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Một máy chủ Keycloak đang chạy cục bộ ở chế độ Dev (có thư mục `bin` và `data` truy cập được).
- Trong máy chủ này, đã có sẵn một Realm mang tên `Alpha-Dev`. Realm này có cấu hình một số Clients bất kỳ.
- Đã cài đặt một CSDL như PostgreSQL hoặc sử dụng H2 mặc định (Ở lab này ta sẽ thao tác Export bằng công cụ của Keycloak nên không phụ thuộc DB).

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Export cấu hình Realm bằng CLI
Chúng ta sẽ sử dụng công cụ dòng lệnh (CLI) của Keycloak (Quarkus) để export.
1. Mở Terminal / Command Prompt.
2. Di chuyển vào thư mục cài đặt của Keycloak (VD: `/opt/keycloak/` hoặc `C:\keycloak`).
3. Đảm bảo rằng máy chủ Keycloak **ĐANG TẮT** (Stop the server). Đây là yêu cầu bắt buộc khi export từ CLI để tránh hỏng dữ liệu.
4. Chạy lệnh Export (Chỉ xuất cấu hình, bỏ qua users):
   - Linux/Mac:
     ```bash
     bin/kc.sh export --dir /tmp/kc-exports --users skip --realm Alpha-Dev
     ```
   - Windows:
     ```cmd
     bin\kc.bat export --dir C:\temp\kc-exports --users skip --realm Alpha-Dev
     ```
5. Truy cập thư mục `kc-exports`, bạn sẽ thấy một file tên là `Alpha-Dev-realm.json`.

### Bước 3.2: Chỉnh sửa file Export (Chuẩn bị cho Staging)
Vì ta sẽ Import vào môi trường Staging (trên cùng 1 server Keycloak này để giả lập), ta cần đổi tên ID để tránh xung đột (Conflict).
1. Mở file `Alpha-Dev-realm.json` bằng một Editor (VS Code hoặc Notepad++).
2. Tìm kiếm trường `"id": "Alpha-Dev"` ở dòng đầu tiên, đổi thành `"id": "Alpha-Staging"`.
3. Tìm kiếm trường `"realm": "Alpha-Dev"`, đổi thành `"realm": "Alpha-Staging"`.
4. Lưu file lại.

### Bước 3.3: Import cấu hình vào môi trường mới
Bây giờ chúng ta sẽ boot Keycloak lên và yêu cầu nó tự động Import file này.
1. Trong Terminal, chạy lệnh Start Keycloak kèm cờ Import:
   - Linux/Mac:
     ```bash
     bin/kc.sh start-dev --import-realm
     ```
   - Windows:
     ```cmd
     bin\kc.bat start-dev --import-realm
     ```
   *Lưu ý:* Khi chạy với cờ `--import-realm`, Keycloak sẽ tự động quét thư mục `data/import/` để tìm file JSON. Do đó, trước khi chạy lệnh trên, hãy **Copy** file `Alpha-Dev-realm.json` (đã chỉnh sửa) vào thư mục `[KEYCLOAK_HOME]/data/import/`.

### Bước 3.4: Mô phỏng chiến lược Backup An toàn (Linux)
Trong môi trường Prod, ta dùng kịch bản bash để sao lưu thư mục data (Nếu dùng H2) hoặc CSDL (Nếu dùng Postgres).
1. Tạo một script có tên `backup-kc.sh`:
   ```bash
   #!/bin/bash
   BACKUP_DIR="/tmp/kc-backups"
   mkdir -p $BACKUP_DIR
   DATE=$(date +"%Y-%m-%d")
   
   # Mô phỏng nén thư mục cấu hình và DB (H2 database nằm trong data/)
   tar -czvf $BACKUP_DIR/kc-backup-$DATE.tar.gz /opt/keycloak/data /opt/keycloak/conf
   
   echo "Backup thành công: $BACKUP_DIR/kc-backup-$DATE.tar.gz"
   ```
2. Chấp quyền thực thi: `chmod +x backup-kc.sh`.
3. Chạy lệnh `./backup-kc.sh` và kiểm tra file nén.

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu phần Import/Export:**
1. Mở trình duyệt, vào địa chỉ `http://localhost:8080/admin` và đăng nhập.
2. Nhấn vào Dropdown Realm ở góc trái trên cùng.
3. Bạn phải nhìn thấy 2 Realms: `Alpha-Dev` (cũ) và `Alpha-Staging` (mới).
4. Truy cập vào `Alpha-Staging`, kiểm tra tab **Clients** và **Roles**. Bạn sẽ thấy mọi cấu hình đã được sao chép nguyên vẹn sang đây, nhưng trong tab **Users** thì hoàn toàn trống rỗng (Đúng như ý định của `--users skip`).

**Troubleshooting (Xử lý sự cố):**
- **Lỗi Keycloak báo `Realm with same name exists` khi khởi động:** Bạn đã quên đổi trường `"id"` và `"realm"` trong file JSON ở bước 3.2. Hãy xóa file import bị lỗi, sửa file JSON đúng cách và khởi động lại.
- **Lệnh Export bị treo hoặc báo lỗi File Locked:** Nếu bạn đang sử dụng cơ sở dữ liệu H2 nội bộ, chỉ một tiến trình JVM được phép truy cập file DB cùng lúc. Bạn PHẢI tắt tiến trình Keycloak đang chạy trước khi gõ lệnh `kc.sh export`. Với PostgreSQL, bạn có thể export khi nó đang chạy nhưng nhà phát triển vẫn khuyến cáo tắt HTTP Server.

> [!TIP]
> Kỹ năng Import/Export từng phần này là nền tảng để bạn tích hợp Keycloak với GitOps hoặc Terraform. Thực tế, DevOps sẽ dùng Terraform để đọc file JSON và apply lên Cloud thay vì copy file tay.
