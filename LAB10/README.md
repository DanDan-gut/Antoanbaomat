LAB10/
├── LAB10_HuongDan.docx           # Tài liệu hướng dẫn chi tiết bài lab
├── LAB10_MauBaoCao.docx          # Mẫu báo cáo kết quả thực hành
└── LAB10_Resources/              # Tài nguyên kỹ thuật và mã nguồn
    ├── README_FIRST.txt          # Hướng dẫn tổng quan về tài nguyên
    ├── download_links.txt        # Đường dẫn tải các công cụ & hình ảnh máy ảo
    │
    ├── 01_DNS_SERVER/            # Cấu hình & mô phỏng DNS Resolver
    │   ├── named.conf.options.baseline
    │   ├── named.conf.local.baseline
    │   ├── vuln_resolver_lab10.py
    │   └── README.txt
    │
    ├── 02_VICTIM/                # Cấu hình máy Nạn nhân (Client)
    │   └── README.txt
    │
    ├── 03_ATTACKER/              # Kịch bản tấn công & Server giả mạo
    │   ├── kaminsky_remote_lab10.py
    │   ├── evil_dns_lab10.py
    │   ├── fake_index.html
    │   └── README.txt
    │
    ├── 04_AUTH_DNS_RELAY/        # Mô phỏng Authoritative DNS Server
    │   ├── auth_dns_relay_lab10.py
    │   └── README.txt
    │
    ├── 05_PFSENSE/               # Ma trận chính sách Tường lửa pfSense
    │   ├── firewall_policy_matrix.csv
    │   └── README.txt
    │
    ├── 06_WIRESHARK/             # Bộ lọc phân tích gói tin Wireshark
    │   └── filters.txt
    │
    └── 07_REPORT/                # Mẫu báo cáo bổ sung
        └── LAB10_Report_Template.docx


🌐 Sơ đồ mô hình mạng (Topology)

Thực thể

Vai trò

Địa chỉ IP

Client (Victim)

Máy nạn nhân gửi truy vấn DNS

192.168.10.10

Local DNS Resolver

Máy chủ DNS trung gian (Có lỗ hổng)

192.168.10.20

Attacker Machine

Máy thực hiện tấn công Kaminsky / Fake Web

192.168.10.30

Fake Auth DNS

Máy chủ DNS mạo danh của Attacker

192.168.10.40

Target Auth DNS

Máy chủ DNS quản lý tên miền hợp lệ (example.com)

192.168.10.50

⚙️ Yêu cầu môi trường

Hệ điều hành: Linux (Ubuntu/Debian) hoặc môi trường máy ảo mô phỏng (pfSense/VMware/VirtualBox).

Phần mềm & Thư viện:

Python 3.x

Thư viện Python: scapy, dnspython

BIND 9.x (đối với phần thực hành BIND & DNSSEC)

Wireshark (để bắt và phân tích luồng gói tin DNS)

🚀 Các bước thực hiện bài thực hành

1. Khởi tạo môi trường

Thiết lập DNS trên máy Client chỉ định địa chỉ 192.168.10.20 làm DNS Server chính:

sudo nano /etc/resolv.conf
# Thêm dòng: nameserver 192.168.10.20


2. Khởi chạy các dịch vụ mạo danh & Authoritative DNS Relay

Tại máy Auth DNS hợp lệ (192.168.10.50):

cd LAB10_Resources/04_AUTH_DNS_RELAY/
sudo python3 auth_dns_relay_lab10.py --ip 192.168.10.50


Tại máy Local DNS Resolver (192.168.10.20):

cd LAB10_Resources/01_DNS_SERVER/
sudo python3 vuln_resolver_lab10.py --ip 192.168.10.20 --port 53


3. Thực thi tấn công Kaminsky DNS Cache Poisoning

Tại máy Attacker (192.168.10.30):

Mở Web Server giả mạo (Phishing):

cd LAB10_Resources/03_ATTACKER/
sudo python3 -m http.server 80


Chạy kịch bản tấn công ngộ độc Cache:

sudo python3 kaminsky_remote_lab10.py \
     --target 192.168.10.20 \
     --domain example.com \
     --fake-ns ns.attacker.com \
     --fake-ip 192.168.10.30


4. Kiểm tra và Xác nhận kết quả

Tại máy Victim (192.168.10.10), thực hiện truy vấn DNS:

dig @192.168.10.20 example.com


Kết quả mong đợi:

Trước khi tấn công: example.com trỏ về IP hợp lệ (vd: 93.184.216.34).

Sau khi tấn công thành công: example.com bị ngộ độc cache và trỏ về IP độc hại của máy Attacker (192.168.10.30).

🛡️ Giải pháp khắc phục & Phòng thủ

Source Port Randomization: Cấu hình BIND9 sử dụng cổng ngẫu nhiên cho các truy vấn đệ quy gửi đi thay vì cố định cổng UDP 53.

Triển khai DNSSEC: Bật xác thực chữ ký số DNSSEC để đảm bảo tính toàn vẹn và xác thực nguồn gốc dữ liệu DNS:

dnssec-validation auto;


Giới hạn Recursion: Chỉ cho phép truy vấn đệ quy từ các IP nội bộ tin cậy.
