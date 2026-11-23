# Biên bản bàn giao NT113 – Phần Khang

> Dự án NT113, đề tài 1 – Thiết kế và triển khai mạng Campus
> Bản mô phỏng [NT113_Khang.pkt](NT113_Khang.pkt)
> Môi trường nghiệm thu Cisco Packet Tracer 9.0.1
> Ngày bàn giao 23/11/2026

## 1. Phạm vi đã triển khai

- Cấu hình hai Core, EtherChannel L3 và các cổng định tuyến lên Core trên 8 DSW. Quảng bá các VLAN và loopback bằng OSPF area 0.
- Cấu hình hai ISP giả lập, hai router biên, PAT, NAT tĩnh cho WEB, LMS và E-LIB, cùng ACL trên WAN.
- Cấu hình IP, gateway HSRP và HTTP cho ba máy chủ DMZ.
- Triển khai VPN site-to-site từ BR-R1 và Easy VPN từ PC-Remote. Bật securityk9 trên ba router VPN và cấu hình DHCP tại cơ sở 2.
- Lưu lại các bản ghi DNS nội bộ cho WEB, LMS và E-LIB.

## 2. Kết quả kiểm tra

| Hạng mục | Kết quả quan sát trong Packet Tracer |
|---|---|
| OSPF | CORE-SW1 có 10 láng giềng FULL. DSW-A1 ping loopback DSW-K2 được 5/5. |
| Dual-ISP | DSW-A1 học hai default route OSPF E2 cùng chi phí. PC-Lab1 có phản hồi từ 8.8.8.8 và 1.1.1.1. Một lượt ping 8.8.8.8 từ PC-Lab1 nhận 3/4 phản hồi. |
| DMZ qua NAT | PC-Remote mở được WEB, LMS và E-LIB qua 203.113.100.2, .3 và .4. |
| DNS và DHCP | PC-Lab1 nhận DHCP và mở được lms.campus.edu.vn cùng www.campus.edu.vn. |
| HSRP DMZ | EDGE-R2 lên Active khi ngắt WAN Viettel. DSW-A1 vẫn ping 8.8.8.8 qua VNPT. EDGE-R1 trở lại Active sau khi khôi phục WAN. |
| VPN site-to-site | IKE báo QM_IDLE / ACTIVE. PC-BR1 ping được DHCP-DNS và mở được http://10.1.50.7. |
| Easy VPN | PC-Remote kết nối và mở được http://10.1.50.7 với ACL biên đang bật. |
| ACL biên | PC-Remote ping WAN EDGE-R1 và nhận 0/4 phản hồi ICMP Echo Reply. |

Các kết quả được ghi theo từng phép thử. Chưa chạy lại toàn bộ bảng trong một lượt sau thay đổi tài khoản Easy VPN cuối cùng. Bài thử Easy VPN và HTTP tới PORTAL-NB đã được chạy lại với tài khoản mới.

## 3. Thiết lập và thao tác khi demo

- Mở file bằng Packet Tracer 9.0.1 ở chế độ Realtime và chờ các láng giềng OSPF hội tụ.
- HSRP nhóm 100 dùng tracking WAN với mức giảm mặc định 10 điểm trong mô phỏng. Khi hai WAN hoạt động, EDGE-R1 có ưu tiên 105 và EDGE-R2 có ưu tiên 100.
- OSPF ưu tiên DSW-A1 cho VLAN 50 và DSW-C1 cho VLAN 40. Chi phí cao hơn trên SVI tương ứng ở switch dự phòng tránh các đường đã ghi nhận mất gói khi kiểm tra.
- ACL EDGE-R2 cho phép lưu lượng Easy VPN đã giải mã đến VLAN máy chủ 10.1.50.0/24. ISP-VNPT lọc địa chỉ nguồn giả mạo thuộc pool VPN trên các cổng hướng ra ngoài.
- Khi mở lại file, vào PC-Remote → Desktop → VPN và nhập hồ sơ REMOTE, máy chủ 113.161.1.2 và tài khoản mô phỏng do nhóm trao đổi riêng. Sau khi kết nối, mở http://10.1.50.7 để kiểm tra.
- Phép kiểm tra Wi-Fi/RADIUS và dự phòng HSRP trên các VLAN access thuộc phần nghiệm thu chung với Khánh.

## 4. Đối chiếu file

SHA-256 của NT113_Khang.pkt

    07498dbcdf372c24892eb5529e07fe76b0e7b2618a8d12bf70c5d8f88a1b8f18
