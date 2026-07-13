> [!NOTE]
> **Category:** Practical/Lab
> **Goal:** Chuẩn bị sẵn sàng hệ thống máy tính cá nhân để có thể thực hành các bài Lab chuyên sâu về Keycloak, Java và Docker.

## 1. Kịch bản Thực hành (Lab Scenario)

Để có thể đóng vai trò là một Kiến trúc sư Hệ thống (System Architect) hoặc Kỹ sư Tích hợp (Integration Engineer) xuyên suốt giáo trình này, bạn cần một môi trường làm việc mạnh mẽ và đầy đủ các công cụ chuyên ngành. Môi trường sẽ được thiết lập xoay quanh hệ sinh thái Java (dùng để viết Custom Extensions cho Keycloak), Docker (dùng để triển khai hạ tầng) và các công cụ phát triển phần mềm chuẩn mực (Git, IDE).

Bài Lab này hướng dẫn chi tiết cách cài đặt và kiểm tra tính hợp lệ của toàn bộ Toolchain trước khi bạn bắt đầu hành trình với Keycloak.

## 2. Chuẩn bị Môi trường (Prerequisites)

- Máy tính chạy hệ điều hành Windows 10/11, macOS hoặc Linux (Ubuntu/Fedora).
- Kết nối Internet ổn định.
- Máy tính cần ít nhất 8GB RAM (khuyến nghị 16GB) vì chạy Keycloak và cơ sở dữ liệu trên Docker khá tiêu tốn bộ nhớ.

## 3. Các bước Thực hiện (Step-by-Step Instructions)

### Bước 3.1: Cài đặt Docker & Docker Compose
1. **Windows/macOS:** Truy cập [Docker Desktop](https://www.docker.com/products/docker-desktop) tải file cài đặt và làm theo hướng dẫn. Với Windows, hãy đảm bảo bạn đã bật WSL2 (Windows Subsystem for Linux) làm backend theo khuyến nghị của Docker.
2. **Linux:** Mở Terminal và thực thi lệnh cài đặt từ repository chính thức của Docker.
   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```
3. Sau khi cài đặt, khởi động Docker Daemon.

### Bước 3.2: Cài đặt Java Development Kit (JDK)
Keycloak 17+ (phiên bản Quarkus) yêu cầu ít nhất Java 17, nhưng phiên bản mới hiện tại khuyến nghị sử dụng Java 21. Chúng ta sẽ sử dụng bản phân phối OpenJDK.
1. Truy cập [Adoptium Eclipse Temurin](https://adoptium.net/).
2. Chọn tải xuống phiên bản **JDK 21** phù hợp với hệ điều hành của bạn.
3. Chạy trình cài đặt. Hãy nhớ tick vào tùy chọn **"Set JAVA_HOME variable"** và **"Add to PATH"** (nếu cài trên Windows).

### Bước 3.3: Cài đặt Apache Maven
Maven là công cụ quản lý dự án và đóng gói mã nguồn Java, sẽ được sử dụng để biên dịch các Keycloak SPIs (Custom Mappers, Custom Authenticators).
1. Truy cập [Apache Maven](https://maven.apache.org/download.cgi) và tải file nhị phân (ví dụ: `apache-maven-3.9.x-bin.zip`).
2. Giải nén vào một thư mục cố định (VD: `C:\Program Files\Apache\maven`).
3. Cấu hình biến môi trường: Thêm đường dẫn `C:\Program Files\Apache\maven\bin` vào biến hệ thống `PATH`.

### Bước 3.4: Cài đặt Git & IDE
1. **Git:** Tải và cài đặt tại [git-scm.com](https://git-scm.com/). Quá trình cài đặt cứ để các thiết lập mặc định (Next to all).
2. **IDE:** Chúng ta khuyến nghị sử dụng **IntelliJ IDEA Community Edition** (miễn phí) hoặc **Visual Studio Code** (VS Code) có kèm theo gói "Extension Pack for Java".

## 4. Nghiệm thu & Kiểm tra (Verification & Troubleshooting)

**Nghiệm thu (Verification):**
Mở một cửa sổ Terminal (hoặc Command Prompt/PowerShell) hoàn toàn MỚI và chạy các lệnh kiểm tra sau.

1. **Kiểm tra Docker:**
   ```bash
   docker version
   docker compose version
   ```
   *Kỳ vọng:* Hiển thị thông tin phiên bản Docker Engine (VD: `24.x.x`) và Docker Compose plugin (VD: `v2.x.x`).

2. **Kiểm tra Java:**
   ```bash
   java -version
   ```
   *Kỳ vọng:* Hệ thống in ra chuỗi có nội dung `openjdk version "21.x.x"`.

3. **Kiểm tra Maven:**
   ```bash
   mvn -v
   ```
   *Kỳ vọng:* Hiển thị thông tin phiên bản Apache Maven, kèm theo vị trí của Java Home trỏ đúng về thư mục cài đặt JDK 21.

4. **Kiểm tra Git:**
   ```bash
   git --version
   ```
   *Kỳ vọng:* Hiển thị `git version 2.x.x`.

**Các lỗi thường gặp (Troubleshooting):**
- **Lỗi 'command not found' (hoặc is not recognized):** Nghĩa là bạn chưa cấu hình đúng biến môi trường `PATH`. Hãy quay lại kiểm tra `System Properties -> Environment Variables` trên Windows và khởi động lại Terminal.
- **Docker báo lỗi 'Cannot connect to the Docker daemon':** Service Docker chưa chạy. Trên Windows/macOS, hãy mở ứng dụng Docker Desktop lên và chờ biểu tượng ở thanh taskbar chuyển sang trạng thái màu xanh lá (Engine Running). Trên Linux, chạy lệnh `sudo systemctl start docker`.
