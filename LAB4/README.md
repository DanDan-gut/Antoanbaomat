
 LAB 4: KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

>
> Nguyễn Thanh Đăng


1. Kiểm tra Mạng & Host Discovery
   # Kiểm tra IP máy quét Kali
ip -br addr

# Kiểm tra kết nối ICMP
ping -c 4 192.168.56.101

# Dò quét các máy đang hoạt động (/24)
nmap -sn 192.168.56.0/24


2. Dò quét Cổng TCP & UDP
   # TCP Connect Scan (Không cần root)
nmap -sT 192.168.56.101

# TCP SYN Scan (Cần quyền sudo/root)
sudo nmap -sS 192.168.56.101

# Quét Tường lửa (FIN, Xmas, NULL, ACK)
sudo nmap -sF 192.168.56.101
sudo nmap -sX 192.168.56.101
sudo nmap -sN 192.168.56.101
sudo nmap -sA 192.168.56.101

# Quét Top 20 cổng UDP
sudo nmap -sU --top-ports 20 192.168.56.101

3. Nhận diện Dịch vụ, Hệ điều hành & Bề mặt Tấn công
   # Định danh phiên bản dịch vụ
nmap -sV 192.168.56.101

# Nhận diện Hệ điều hành (OS Fingerprinting)
sudo nmap -O 192.168.56.101

# Quét tổng hợp (Aggressive)
sudo nmap -A 192.168.56.101

4. Đánh giá Lỗ hổng SMB (NSE Scripts)
   # Thu thập thông tin OS/Domain qua SMB
nmap --script smb-os-discovery -p 445 192.168.56.101

# Kiểm tra lỗ hổng EternalBlue (MS17-010)
nmap --script smb-vuln-ms17-010 -p 445 192.168.56.101


5. Xuất Kết quả & Tạo Báo cáo HTML
   # Xuất 3 định dạng (.nmap, .xml, .gnmap)
sudo nmap -sV -O 192.168.56.101 -oA scan_results

# Lọc các cổng 445/open từ file Grepable
grep "445/open" scan_results.gnmap

# Chuyển XML sang giao diện Web HTML
xsltproc scan_results.xml -o scan_results.html
