# LAB 2: MÃ HÓA ĐỐI XỨNG HIỆN ĐẠI

- **Họ và tên:** Nguyễn Thành Đăng
- **MSSV:** 1150080008
- **Lớp:** 11THMT
- **Học phần:** An toàn và Bảo mật thông tin

## 1. Cấu trúc thư mục
- `pom.xml` & `src/`: Mã nguồn dự án Java Maven (`LAB2_MaHoaHienDai`), bao gồm các thuật toán DES, TripleDES, AES-GCM (kèm đăng ký/đăng nhập SQL Server), RC4, Twofish, AES-CTR và menu chính `frm_Menu`.
- `aes_cbc_gui.py`: Chương trình mã hóa/giải mã AES-256-CBC viết bằng Python (Tkinter).
- `*.sql`: Các script khởi tạo cơ sở dữ liệu `LAB2_MaHoaHienDai`, bảng `app_users`, `crypto_history` và tài khoản thực hành.
- `images/`: Hình ảnh minh chứng kết quả kiểm thử các bài thực hành.

## 2. Hướng dẫn cài đặt & chạy chương trình

### A. Cơ sở dữ liệu SQL Server
1. Mở SQL Server Management Studio (SSMS), chạy lần lượt `01_Tao_CSDL.sql` và `02_Tao_Tai_Khoan_SQL.sql`.
2. Đảm bảo SQL Server bật TCP/IP tại cổng `14333` và chế độ Mixed Mode Authentication.

### B. Ứng dụng Java (Apache NetBeans)
1. Mở project `LAB2_MaHoaHienDai` bằng Apache NetBeans (JDK 17+).
2. Nhấp chuột phải vào project -> chọn **Clean and Build** (`Shift + F11`) để Maven tải thư viện `mssql-jdbc` và `bcprov-jdk18on`.
3. Bấm **Run Project** (`F6`) để khởi chạy giao diện chính `week_02.frm_Menu`.
4. Chọn **Kết nối SQL Server**, nhập thông tin kết nối (`127.0.0.1`, port `14333`, database `LAB2_MaHoaHienDai`, user `lab2_user`) để lưu lịch sử thao tác.

### C. Ứng dụng Python (AES-256-CBC)
1. Cài đặt thư viện `cryptography`:
   ```bash
   py -m pip install cryptography
