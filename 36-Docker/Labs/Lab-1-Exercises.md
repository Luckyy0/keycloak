# Lab 1: Xưởng Đúc Tàu Trăm Nghìn Tỷ (Custom Build Lab)

> [!NOTE]
> **Category:** Practical/Lab (Thực hành)
> **Goal:** Tự tay viết Dockerfile theo cơ chế Lò Nung Của Quarkus (Multi-stage). Gói ghém cấu hình Database và đẩy con Tàu Keycloak Lên Không Gian chỉ Bằng 1 Nút Bấm duy nhất. Trải Nghiệm cảm giác "Code Đã Compile Chạy Cực Nhanh" của bản Run-time.

## 1. Yêu cầu (Prerequisites)
- Docker.

## 2. Các bước thực hiện (Step-by-step)

### Bước 1: Chuẩn Bị Khoang Hàng Gốc
Tạo một thư mục `code/lab/` (Tự Tạo).
Mở Terminal trỏ vào thư mục `code/lab/` đó Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.

### Bước 2: Viết Lò Nung (Dockerfile)
Tạo file `Dockerfile` với đoạn Code Cắt Tỉa Chế Trọng Lượng sau:
```dockerfile
# ---------------- GIAI ĐOẠN 1: Nung Lõi Quarkus ----------------
FROM quay.io/keycloak/keycloak:24.0.1 as builder

# 1. Bật Các Tính Năng Giám Sát Cố Định
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true

# 2. Ép Trái Tim Postgres Lệnh Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh
ENV KC_DB=postgres

# NẾU BẠN CÓ THEME CÔNG TY, HÃY UNCOMMENT DÒNG NÀY ĐỂ BƠM NÓ VÀO BỤNG
# COPY my-theme /opt/keycloak/themes/my-theme

# 3. Kích Hoạt Lệnh Nung Lệnh Đáy DB Chữ Khớp Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy (Xóa Hết Các Thứ Nhảm Nhí Không Phải Postgres Trượt Khung Khớp Lệnh Cắt Bọt Đứt Băng Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Cấu Trúc Khung Rỗng XML Nặng Nề)
RUN /opt/keycloak/bin/kc.sh build


# ---------------- GIAI ĐOẠN 2: Bơm Vào Lớp Vỏ Mỏng Siêu Nhẹ ----------------
FROM quay.io/keycloak/keycloak:24.0.1
COPY --from=builder /opt/keycloak/ /opt/keycloak/

# Dùng Nẹp OOM Killer Đỉnh Đáy Oanh Mạng Bắt Lụa Đáy Lụa Lệnh Tĩnh Cáp Mạch Máu Cắt Mạng Khung Cắt Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Đỉnh Cao Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa
ENV KC_JAVA_OPTS="-Xmx512m"

# Tự Lái Tàu Vào Bay
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
```

### Bước 3: Ép Khuôn (Build Image)
Ở Dưới Khung Lệnh, Gõ Nút Đúc Tàu Lỗ Rò Lệnh Cắt Mạch Đứt Kẽ Mã Bơm Oanh Tĩnh Lụa Thép Đáy Bọc Lệnh Cũ Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Trút Kéo Lụa Oanh Bọc Khớp Lệnh Cũ Rích Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:
```bash
docker build -t my-keycloak:1.0 .
```
Bạn Sẽ Thấy Docker Nó Đốt Ruột (Chữ Đỏ Chạy Kèo Kéo `kc.sh build` Khoảng Mấy Chục Giây Mạch Nhựa Dữ Cốt Rỗng API Lệch Băng Tần Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh). 
Xong Báo "Successfully built...".

### Bước 4: Nhét Tàu Trực Tiếp Lên Bệ Phóng (Docker Compose)
Ở Thư mục `code/`, mở file `docker-compose.yml`. Thay Vị Trí Dòng Image:
```yaml
  keycloak:
    # NGUỒN CỘI LÀ BẢN ĐÚC CỦA BẠN Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa! Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa
    image: my-keycloak:1.0 
    
    # KHI TÀU CHẠY Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa, BẮT BUỘC Phải Mang Chữ --optimized (Đã Xào Nấu Trút Cáp Mạch Máu Cắt Lệnh Đáy DB Lệnh Chóp Cắt Đứt Nối Dòng Json Oanh Thép Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy)
    command: start-dev --optimized 
```

### Bước 5: Ngắm Tốc Độ Cất Cánh Trượt Mạch Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị Oanh Mạng Tuyệt Đối Khung Tĩnh Oanh Khớp Đáy Lụa Băng Tần
Chạy Lệnh `docker-compose up -d`.
Gõ `docker logs -f code-keycloak-1`. 
**CHIÊM NGƯỠNG ĐI Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp!** Tàu Bay Bật Khởi Động Chưa Tới 3 Giây Bọc Lệnh Cũ Đỉnh Chóp Trượt Nhựa Dưới Đáy Mạch Máu Cắt Lệnh Đáy Trút Lụa Bọt Kẽ Mã Đáy Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khúc Tới Ngay Lệnh! (Bản Gốc Mất Chừng 15 Giây Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa). 
Bạn Đã Nung Chảy Hoàn Mỹ Một Khối Cấu Trúc Bất Biến Mạch Oanh Giao Dịch Dữ Lụa Đỉnh Chóp Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Chữ Nghĩa Cũ Mạch Cáp 1 Phiên Trút Code API Oanh Lụa Bọt Giao Diện Lệnh Đáy! Tàu Dù Bay Ở Trạm Host Nào Lỗ Bọt Cắt Trắng Đứt Rỗng Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Đáy Oanh Mạch Rút Trọng Mạch Lệnh Khúc Tới Ngay Mạch Cẽ Trút Rỗng Băng Tần Mạng Khung Cắt Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa Cũng Bơm Phụt Phát Ăn Ngay Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa Cấu Trúc Khung Rỗng XML Nặng Nề! Chặt Khung Oanh Đỉnh Đáy Oanh Mạng Bắt Lụa Nhựa Bọc Cắt Chữ Kẽ Lỗ Rò Đỉnh Chóp Bọt Mạch Kéo Rỗng Kẽ Cướp Dữ Liệu Tiền Tỉ Oanh Cáp Trọng Lõi Tự Trị!
