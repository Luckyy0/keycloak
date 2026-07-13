# Chapter 20: Identity Brokering (Cò Mồi Định Danh)

Chào mừng bạn đến với **Chương 20: Identity Brokering (Federation)**.
Trong các hệ thống Enterprise lớn, Keycloak không phải là Nơi Duy Nhất Lưu Trữ Mật Khẩu. Bạn có thể muốn cho phép Khách Hàng đăng nhập bằng Google, Facebook, hoặc tài khoản Active Directory nội bộ của công ty mẹ (Nối thông qua OIDC hoặc SAML).
Khi đó, Keycloak đóng vai trò là một gã **Identity Broker (Cò Mồi Định Danh)**. Nó đứng ở giữa, nhận yêu cầu từ Ứng Dụng của bạn, và "bán lại" yêu cầu đó cho các Mạng Xác Thực bên ngoài (Google/SAML IdP). 

## Mục Tiêu Học Tập (Learning Objectives)
Kết thúc chương này, bạn sẽ nắm vững:
1. Kiến trúc Brokering: Cách Keycloak hóa thân thành Cò Mồi để giao tiếp 2 chiều (Vừa làm IdP cho App của bạn, Vừa làm SP đi xin Token từ Google).
2. Xử lý Lần Đăng Nhập Đầu Tiên (First Broker Login Flow).
3. Bí thuật Liên kết Tài Khoản (Account Linking) Trượt Nhựa Giữa 2 Thế Giới.
4. Cấu hình Identity Providers (Google, Github, SAML).

## Cấu Trúc Thư Mục (Directory Structure)
- `Module-1-Federation/`: 4 bài lý thuyết giải phẫu toàn diện nghệ thuật Brokering Đỉnh Chóp!
- `Labs/`: Thực hành đấu nối Keycloak với thế giới bên ngoài.
- `code/`: File docker-compose khởi tạo môi trường thực hành.

## Danh Sách Bài Học (Lesson List)
- Lesson 1: Brokering Architecture (Kiến Trúc Cò Mồi)
- Lesson 2: First Broker Login Flow (Vượt Cửa Ải Đầu Tiên)
- Lesson 3: Account Linking (Gắn Kết Đáy Lụa)
- Lesson 4: Identity Providers (Đấu Nối Mạng Lưới Rỗng Kẽ)

Sẵn sàng biến Keycloak Thành Con Nhện Chăng Tơ Nối Kết Mọi Hệ Thống Xác Thực Trên Thế Giới Oanh Cáp Trọng Lõi Tự Trị!
