LAB 7 – Xây dựng chính sách Web Protection với pfSense và Web Proxy
1. Giới thiệu
LAB 7 thực hành xây dựng chính sách kiểm soát truy cập website trong mạng nội bộ bằng mô hình pfSense + Web Proxy. pfSense đảm nhiệm vai trò firewall/gateway, còn máy chủ Web Proxy chạy riêng trên Ubuntu Server để lọc nội dung web, cảnh báo/chặn website và kiểm tra tải tệp.

Thiết kế của bài lab không cài Squid/E2Guardian trực tiếp trên pfSense. Web Proxy được triển khai trên máy Ubuntu Server riêng, giúp phân biệt chức năng firewall ở lớp mạng/vận chuyển với proxy lọc nội dung web.
2. Mục tiêu
- Cấu hình pfSense kiểm soát lưu lượng web từ máy khách.
- Ép Client-A và Client-B truy cập web thông qua Web Proxy, ngăn truy cập trực tiếp TCP/80 và TCP/443 để bypass proxy.
- Triển khai Squid, E2Guardian và ClamAV trên Ubuntu Server 24.04 LTS.
- Tạo website mô phỏng trong DMZ để kiểm thử các chính sách.
- Thực hiện hard block, warning/temporary bypass, exception/whitelist và chính sách theo khung giờ.
- Kiểm tra cơ chế quét tệp tải xuống bằng ClamAV.
- Theo dõi log và thu thập ảnh minh chứng.
- Tìm hiểu HTTPS inspection như phần mở rộng trong môi trường lab cô lập.
3. Công nghệ sử dụng
Thành phần	Vai trò
pfSense CE	Firewall/gateway, alias, firewall rules và schedule
Ubuntu Server 24.04 LTS	Máy chủ Web Proxy
E2Guardian	Lọc URL/nội dung, cảnh báo và temporary bypass
Squid	Forward proxy phía upstream
ClamAV	Quét virus cho nội dung tải xuống được chuyển đến scanner
Windows Server AD/DNS (DC01)	Active Directory và phân giải DNS trong lab
Nginx trên DMZ-WEB	Lưu trữ các website mô phỏng
Client-A, Client-B	Máy khách dùng để kiểm thử chính sách


4. Mô hình mạng
Địa chỉ IP
Máy/thành phần	Địa chỉ IP / mạng	Vai trò
pfSense – MGMT	192.168.56.254	Quản trị
Máy thật quản trị	192.168.56.1/24	Quản trị môi trường lab
pfSense – LAN	10.6.0.1	Gateway LAN
DC01	10.6.0.10/24	AD/DNS
Client-A	10.6.0.21/24	Máy khách kiểm thử
Client-B	10.6.0.22/24	Máy khách kiểm thử
WEB-PROXY	10.6.0.30/24	Web Proxy
pfSense – DMZ	172.16.6.1	Gateway DMZ
DMZ-WEB	172.16.6.10/24	Website mô phỏng


Luồng truy cập web
1. Client-A/Client-B cấu hình proxy trình duyệt là 10.6.0.30:8080.
2. E2Guardian nhận yêu cầu từ client và áp dụng chính sách.
3. Squid chạy tại 127.0.0.1:3128 làm upstream forward proxy.
4. ClamAV được dùng để quét nội dung tải xuống khi scanner được cấu hình.
5. pfSense cho phép WEB-PROXY tạo kết nối web cần thiết, đồng thời chặn client truy cập trực tiếp TCP/80 và TCP/443.
Lưu ý: Squid chỉ nên lắng nghe trên localhost theo thiết kế bài lab để client không thể kết nối trực tiếp tới Squid và bỏ qua E2Guardian.
5. Cấu hình Web Proxy
Thông số máy WEB-PROXY theo tài liệu:
- Hệ điều hành: Ubuntu Server 24.04 LTS
- RAM: 2 GB
- CPU: 2 vCPU
- Ổ đĩa: 20 GB
- IP: 10.6.0.30/24
- Gateway: 10.6.0.1
- DNS: 10.6.0.10
Các dịch vụ và cổng:
Dịch vụ	Địa chỉ nghe	Chức năng
E2Guardian	10.6.0.30:8080	Proxy đầu vào cho client và áp dụng policy
Squid	127.0.0.1:3128	Upstream forward proxy
ClamAV daemon	Local socket/service	Quét nội dung được chuyển đến scanner


Cài đặt và cấu hình theo hướng dẫn LAB 7 hoặc script trong bộ tài nguyên do giảng viên cung cấp. Sau khi cài đặt, kiểm tra kết nối tới gateway 10.6.0.1, DNS 10.6.0.10 và trạng thái các dịch vụ trước khi chuyển sang bước tiếp theo.
6. Website mô phỏng và chính sách
Các hostname trong lab sử dụng miền .lab7.test; chúng được mô phỏng trong mạng lab và không yêu cầu truy cập website thật.
Nhóm chính sách	Hostname	Kết quả mong đợi
Website bình thường	normal.lab7.test	Cho phép
Geo-Sim China	china.lab7.test	Hard block
Geo-Sim Middle East	mideast.lab7.test	Hard block
Job Search	jobs.lab7.test	Hard block
Weapons	weapons.lab7.test	Warning/temporary bypass
Politics	politics.lab7.test	Warning/temporary bypass
Religion	religion.lab7.test	Warning/temporary bypass
Education	education.lab7.test	Cho phép, không cảnh báo
Entertainment	entertainment.lab7.test	Theo schedule
Games	games.lab7.test	Theo schedule
Streaming	streaming.lab7.test	Warning/temporary bypass
Game bị chặn cụ thể	blockedgame.lab7.test	Hard block
Download	download.lab7.test	Cho phép truy cập, áp dụng kiểm soát AV/kích thước


Hard block
Đưa china.lab7.test, mideast.lab7.test, jobs.lab7.test và blockedgame.lab7.test vào danh sách chặn. Nếu E2Guardian có danh sách non-bypass, đưa các hostname này vào danh sách đó để không thể temporary bypass.
Warning / temporary bypass
Đưa weapons.lab7.test, politics.lab7.test, religion.lab7.test và streaming.lab7.test vào danh sách cảnh báo có thể mở tạm. Theo tài liệu, cấu hình bypass = 300 để minh họa thời gian bypass 5 phút.
Exception / whitelist
Các hostname normal.lab7.test, education.lab7.test và download.lab7.test được đưa vào exception/whitelist. Website download vẫn chịu kiểm tra antivirus và chính sách kích thước tệp.
Entertainment/Game theo thời gian
Dùng schedule LAB7_ENT_60M trên pfSense để giới hạn truy cập theo khung 60 phút do giảng viên quy định. Đây là time-window policy, không được mô tả là daily category quota tương đương hoàn toàn với Sophos.
Quét antivirus và giới hạn kích thước tải xuống
- Bật scanner ClamAV trong cấu hình E2Guardian theo hướng dẫn của bài lab.
- Dùng tệp kiểm thử EICAR tại download.lab7.test/files/eicar.com.txt để xác nhận cơ chế quét. EICAR là chuỗi kiểm thử antivirus, không phải mã độc thực.
- Kiểm tra 100MB.bin và 110MB.bin để đánh giá chính sách kích thước.
- Chỉ kết luận giới hạn trên 100 MB hoạt động khi cấu hình hỗ trợ và kết quả kiểm thử chứng minh tệp 110 MB bị từ chối. Tham số maxcontentfilecachescansize không mặc nhiên là hard download-size limit.
7. Cấu hình pfSense chống bypass
Các alias được sử dụng trong tài liệu:
Alias	Loại	Thành viên
LAB7_CLIENTS	Host	10.6.0.21, 10.6.0.22
LAB7_PROXY	Host	10.6.0.30
WEB_PORTS	Port	80, 443
MAIL_PORTS	Port	25, 465, 587, 110, 995, 143, 993


Ruleset LAN tối thiểu theo thứ tự trong tài liệu:
1. Cho phép DNS từ LAN tới DC01 10.6.0.10 qua TCP/UDP 53.
2. Cho phép LAB7_CLIENTS kết nối LAB7_PROXY qua TCP 8080.
3. Cho phép các cổng email theo yêu cầu của bài lab.
4. Chặn LAB7_CLIENTS truy cập trực tiếp các cổng trong WEB_PORTS.
5. Cho phép LAB7_PROXY truy cập các cổng web TCP/80 và TCP/443.
6. Cho phép LAB7_PROXY truy cập website thử nghiệm trong DMZ 172.16.6.10 qua TCP 80.
7. Đặt quy tắc chặn cuối theo thiết kế của bài lab.
Sau khi thay đổi rule, nhấn Apply Changes. Khi kiểm thử lại luồng đã tồn tại, có thể cần Reset States vì pfSense là stateful firewall.
Kiểm thử chống bypass
- Khi bật proxy trên trình duyệt, truy cập http://normal.lab7.test phải thành công.
- Khi tắt proxy, Client-A/Client-B phải bị pfSense chặn khi truy cập web trực tiếp TCP/80 hoặc TCP/443.
- Từ WEB-PROXY, kiểm tra kết nối tới website được phép.
- Bật lại proxy trên client trước khi kiểm thử các chính sách tiếp theo.
8. Kịch bản kiểm thử
STT	Kịch bản	Đích kiểm thử	Kết quả mong đợi
1	Baseline	normal.lab7.test	Qua proxy thành công; direct web bị chặn
2	Geo-Sim	china.lab7.test, mideast.lab7.test	Hard block, không temporary bypass
3	Job Search	jobs.lab7.test	Hard block
4	Warning content	weapons.lab7.test, politics.lab7.test, religion.lab7.test	Có trang cảnh báo; bypass ngắn hạn
5	Education	education.lab7.test	Truy cập được, không cảnh báo
6	Entertainment/Game	entertainment.lab7.test, games.lab7.test	Chỉ truy cập trong schedule
7	Streaming	streaming.lab7.test	Cảnh báo, có thể bypass tạm
8	Specific game	blockedgame.lab7.test	Hard block
9	Antivirus	download.lab7.test/files/eicar.com.txt	AV chặn/cảnh báo hoặc log scanner chứng minh phát hiện
10	File size	100MB.bin, 110MB.bin	Đánh giá theo kết quả thực tế và cấu hình size ACL


Mỗi kịch bản cần có tối thiểu:
1. Ảnh chụp policy/list/rule liên quan.
2. Ảnh trình duyệt hoặc lệnh kiểm thử.
3. Một đến hai câu giải thích kết quả.
Chỉ sử dụng ảnh chụp từ môi trường lab do sinh viên thực hiện; không dùng ảnh mẫu trong tài liệu làm minh chứng.
9. Logging và xử lý sự cố
Các log cần đối chiếu:
- pfSense: Status → System Logs → Firewall
- E2Guardian: /var/log/e2guardian/access.log (đường dẫn có thể khác tùy package)
- Squid: /var/log/squid/access.log
- ClamAV: journalctl -u clamav-daemon hoặc log tương ứng của package
- Client: lỗi trình duyệt, cài đặt proxy và kết quả phân giải DNS
Một số hướng kiểm tra nhanh:
Hiện tượng	Kiểm tra
Không truy cập được website	Proxy trình duyệt, ping 10.6.0.30, cổng 8080, dịch vụ E2Guardian/Squid, rule pfSense và DNS
Tắt proxy vẫn vào web được	Thứ tự rule chặn LAB7_CLIENTS tới TCP/80,443; Apply Changes và state cũ
Proxy báo 502/connection failed	Squid upstream, DNS của WEB-PROXY, route và gateway 10.6.0.1
Website cần chặn vẫn truy cập được	Danh sách FQDN, reload E2Guardian, exception/whitelist và cache trình duyệt
Không hiện warning/bypass	bypass = 300, reportinglevel và danh sách non-bypass
EICAR tải được	clamav-daemon, scanner clamd, cấu hình content scanner và danh sách ngoại lệ
HTTPS báo lỗi chứng chỉ	CA lab chưa được import, hostname, cache chứng chỉ và cấu hình MITM
100 MB và 110 MB không khác nhau	Kiểm tra khả năng hỗ trợ size ACL của Squid và log; không kết luận policy đã đạt


10. HTTPS inspection (phần mở rộng)
HTTPS inspection chỉ được thực hiện trên Client-A/Client-B trong môi trường lab cô lập:
1. Tạo CA lab bằng script generate_lab7_ca.sh.
2. Cấu hình các đường dẫn CA/key và SSL MITM theo README_HTTPS.txt.
3. Bật sslmitm = on trong cấu hình E2Guardian tương ứng.
4. Import ca.der hoặc ca.pem vào Trusted Root Certification Authorities trên client lab.
5. Khởi động lại E2Guardian, mở lại trình duyệt và kiểm thử website HTTPS.
6. Ghi nhận sự khác biệt về chứng chỉ trước và sau khi tin cậy CA lab.
An toàn: Không import CA LAB 7 vào máy cá nhân sử dụng hằng ngày. Không dùng SSL MITM để giải mã tài khoản thật, ngân hàng, email cá nhân hoặc dữ liệu nhạy cảm. Không để private key của CA rời khỏi WEB-PROXY.
11. Ảnh minh chứng cần có
- Topology/VM và địa chỉ IP của WEB-PROXY (10.6.0.30).
- Alias và ruleset pfSense ép Client-A/Client-B đi qua proxy.
- Website normal.lab7.test được phép và phép thử direct web bị chặn khi tắt proxy.
- Hard block Geo-Sim và Job Search.
- Warning và temporary bypass của ít nhất một website.
- Website Education được phép.
- Schedule 60 phút và kết quả trong/ngoài schedule.
- EICAR bị chặn hoặc log scanner chứng minh phát hiện.
- Log E2Guardian và Firewall tương ứng.
- Nếu thực hiện HTTPS inspection: lỗi chứng chỉ trước khi import CA và kết quả sau khi trust CA.
12. Câu hỏi phân tích
Báo cáo cần giải thích:
1. Vì sao LAB 7 không cài Squid/SquidGuard trực tiếp trên pfSense?
2. Firewall pfSense và Web Proxy khác nhau ở lớp nào và loại thông tin nào có thể kiểm tra?
3. Vì sao phải chặn direct TCP/80,443 từ Client-A/B khi đã cấu hình proxy?
4. Vì sao lọc hostname/category của website Internet thật có thể không ổn định nếu chỉ dựa vào IP?
5. Temporary bypass khác hard block ở điểm nào?
6. Vì sao EICAR qua HTTPS không thể quét nội dung nếu không có SSL MITM?
7. CA dùng cho HTTPS inspection tạo ra rủi ro gì nếu private key bị lộ?
8. maxcontentfilecachescansize có phải luôn là hard download-size limit không?
9. Vì sao Schedule 60 phút không đồng nghĩa với daily category quota của Sophos?
10. Khi trình duyệt báo lỗi nhưng E2Guardian không có log, giả thuyết đầu tiên cần kiểm tra là gì?
13. Yêu cầu nộp bài
- Thực hiện cá nhân trên máy ảo của sinh viên.
- Nộp báo cáo Word (.docx) theo mẫu LAB7_Report_Template.docx hoặc định dạng tương đương.
- Tên file: [Mã lớp]-Lab7_MSSV-HoTen.docx.
- Ảnh minh chứng phải lấy từ môi trường thực hành của sinh viên.
- Nộp kèm file backup cấu hình pfSense nếu giảng viên yêu cầu.
- Không nộp private key CA.
- Báo cáo phải giải thích từng chính sách và kết quả, không chỉ ghi đã chạy script thành công.
14. Tài liệu tham khảo
- Netgate – pfSense Documentation: Firewall, Aliases, Schedules, Captive Portal và Packages.
- Netgate – thông báo deprecated đối với các package Squid/SquidGuard/Lightsquid.
- Netgate – pfBlockerNG documentation (GeoIP/DNSBL, phần mở rộng tùy chọn).
- Ubuntu Packages – Squid, E2Guardian và ClamAV trên Ubuntu 24.04 LTS.
- E2Guardian Project Documentation/Wiki.
- EICAR – Anti-Malware Testfile documentation.
