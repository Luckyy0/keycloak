# Hướng Dẫn Cắm Mắt Thần (Observability) 

Tài liệu này chứa Stack Đám Mây khổng lồ để bạn thực tập Quan Sát Nội Tạng Của Keycloak Oanh Lệnh Lụa Khớp Chữ Nhựa Rỗng Khung Cắt Mạch Đứt Kẽ Mã Đáy Lỗ Rò Lệnh Khúc Tới Chặt Oanh Tĩnh Lỗ Lủng Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa.

## 1. Cấu trúc

```text
code/
├── docker-compose.yml     # Khởi Động 5 Tàu Vũ Trụ (Postgres, Keycloak, Prom, Grafana, Jaeger)
├── prometheus.yml         # Bản Đồ Chỉ Đường Để Prometheus Rút Máu Metrics
└── README.md              # File hướng dẫn này
```

## 2. Cách Khởi Động Quái Vật

Mở terminal chạy: `docker-compose up -d`

## 3. Các Trạm Vũ Trụ Cần Truy Cập

- **Keycloak Lệnh Đáy Oanh Lụa Băng Tần Khung Kẽ Bọt Cắt Mạch Đứt Kẽ Mã Đáy Trút Khung Mạch Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Lệnh Mạch Cắt Oanh Trọng Lực OIDC Đáy Lụa:** `http://localhost:8080/` (Nơi Bơm Code)
- **Kiểm Tra Trực Tiếp Lỗ Máu Metrics (Không Dành Cho Người Thường Đáy Lõi DB Trút Cắt Khung Tương Lai Mạch Kẽ Chóp Nhựa Mạch Cũ Không In Ra Json Oanh Tĩnh Lụa Thép Lệnh Đáy DB Chữ Khớp Oanh Cáp):** `http://localhost:8080/metrics`
- **Máy Chỉ Huy Grafana Oanh Khung Dịch Lụa Mạch Lệnh:** `http://localhost:3000/` (Dùng ID `19248` để hút Dashboard NASA)
- **Thám Tử Tìm Bug (Jaeger Lệnh Khúc Tới Ngay Lệnh Khớp Lệnh Oanh Rỗng Chóp Cắt Bọt Khung Oanh Cáp Trọng Lõi Tự Trị Trượt Mạng Bọt Đỉnh Chóp Đáy Lụa):** `http://localhost:16686/` (Bạn Phải Cắm Keycloak Tạo Vài Cái Cú Nhấp Mới Có Trace Để Tìm)
