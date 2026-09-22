# BÁO CÁO LAB 3: NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ATTT

## 1. Thông Tin Sinh Viên
* **Họ và tên:** Nguyễn Thanh Đăng
* **MSSV:** 1150080008 | **Lớp:** 11-THMT
* **Link Video:** N/A

## 2. Môi Trường Thực Hành
* **OS:** Windows 11 25H2 x64 (Build 26200.9445 - KB5124008) | **VM:** VMware Workstation Pro 26H1
* **Tools:** Microsoft Defender (ON), Sysmon 15.22, Autoruns 14.3, Process Explorer 17.14, Wireshark 4.6.8, Python 3.14.7

## 3. Cách Dựng Môi Trường
1. Tạo VM Win11 Host-only, update KB5124008, tạo snapshot `LAB3_CLEAN_20260914`.
2. Tạo thư mục `C:\LAB3`, giải nén `LAB3_Threats_Assets.zip` và verify SHA-256.
3. Cài đặt Python 3.14.7, Wireshark, Sysmon và bộ công cụ Sysinternals.

## 4. Kết Quả Thực Hiện (Tasks)

| STT | Tình huống | Kết quả | Ghi chú / Bằng chứng chính |
|---|---|---|---|
| 1 | **TH1:** Risk Register | PASS | Lập Risk Register 5 tài sản & phân loại 5 nguồn đe dọa |
| 2 | **TH2:** Malware (EICAR) | PASS | Defender phát hiện & quarantine EICAR |
| 3 | **TH3:** Password & Keylogging | PASS | Ghi log Event 4624/4625/4648, đổi mật khẩu thành công |
| 4 | **TH4:** Backdoor & Persistence | PASS | Phát hiện Run key, Scheduled Task & listener 127.0.0.1 |
| 5 | **TH5:** Sniffing / MITM | PASS | Capture HTTP plaintext (loopback) vs HTTPS TLS/443 |
| 6 | **TH6:** DoS, DDoS & Mailbomb | PASS | Chạy local load test 127.0.0.1, phân tích log offline |
| 7 | **TH7:** Social Engineering | PASS | Chỉ ra 5 chỉ dấu Phishing, phân loại 6 case |

## 5. Lỗi Gặp Phải & Khắc Phục
* **Lỗi:** Wireshark không thấy giao diện Loopback.
* **Khắc phục:** Cài đặt Npcap đi kèm Wireshark và chọn `Adapter for loopback traffic capture`.
