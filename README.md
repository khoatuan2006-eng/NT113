# NT113 — Đề tài 1: Mạng Campus

Repo lưu topology Cisco Packet Tracer và tài liệu triển khai của nhóm.

- [Campus-Topology.pkt](Campus-Topology.pkt) — sơ đồ nền gồm 57 thiết bị và 81 dây nối. Mở bằng Cisco Packet Tracer 9.0.1.
- README này chứa phân công, lệnh CLI, thao tác qua giao diện, lịch ghép và bảng nghiệm thu. Thực hiện cấu hình theo phần việc bên dưới.

Nếu Packet Tracer hỏi cập nhật script cho bốn camera khi mở file, chọn **Yes** rồi lưu bản làm việc riêng. Các giá trị dạng `<ENABLE_SECRET>`, `<RADIUS_SHARED_SECRET>` và các placeholder tương tự cần được thay bằng thông tin lab do nhóm thống nhất trước khi áp dụng cấu hình. Với RADIUS và VPN, hai đầu phải dùng giá trị tương ứng giống nhau.

## Quy hoạch VLAN, IP và cổng

[Sheet quy hoạch gốc](https://claude.ai/artifact/8rpZGDr5M7z7Wm1azx1zty#sheet-sht_2sjmn9tr6d.0)

Các bảng bên dưới được trích từ cấu hình CLI, bảng DHCP và sơ đồ nối dây trong NT113. Chưa đối chiếu từng ô với sheet gốc. Tất cả địa chỉ và dịch vụ bên dưới thuộc mô phỏng Packet Tracer.

### Subnet và VLAN

| Khối | VLAN | Mục đích | Subnet | Gateway HSRP | IP hai DSW | Dải DHCP |
|---|---:|---|---|---|---|---|
| A | 10 | Ban giám hiệu và hành chính | `10.1.10.0/24` | `10.1.10.1` | `10.1.10.2` / `10.1.10.3` | `10.1.10.11` – `10.1.10.254` |
| A | 20 | Giảng viên | `10.1.20.0/24` | `10.1.20.1` | `10.1.20.2` / `10.1.20.3` | `10.1.20.11` – `10.1.20.254` |
| A | 30 | Wi-Fi sinh viên | `10.20.0.0/19` | `10.20.0.1` | `10.20.0.2` / `10.20.0.3` | `10.20.0.11` – `10.20.31.254` |
| A | 50 | Máy chủ nội bộ | `10.1.50.0/24` | `10.1.50.1` | `10.1.50.2` / `10.1.50.3` | IP tĩnh |
| A | 60 | Camera | `10.1.60.0/24` | `10.1.60.1` | `10.1.60.2` / `10.1.60.3` | `10.1.60.11` – `10.1.60.254` |
| A | 99 | Quản trị | `10.1.99.0/24` | `10.1.99.1` | `10.1.99.2` / `10.1.99.3` | IP tĩnh |
| B | 20 | Giảng viên | `10.2.20.0/24` | `10.2.20.1` | `10.2.20.2` / `10.2.20.3` | `10.2.20.11` – `10.2.20.254` |
| B | 60 | Camera | `10.2.60.0/24` | `10.2.60.1` | `10.2.60.2` / `10.2.60.3` | `10.2.60.11` – `10.2.60.254` |
| B | 70 | Access point | `10.2.70.0/24` | `10.2.70.1` | `10.2.70.2` / `10.2.70.3` | `10.2.70.11` – `10.2.70.254` |
| B | 80 | Thư viện | `10.2.80.0/24` | `10.2.80.1` | `10.2.80.2` / `10.2.80.3` | `10.2.80.11` – `10.2.80.254` |
| B | 99 | Quản trị | `10.2.99.0/24` | `10.2.99.1` | `10.2.99.2` / `10.2.99.3` | IP tĩnh |
| C | 40 | Phòng lab | `10.3.40.0/23` | `10.3.40.1` | `10.3.40.2` / `10.3.40.3` | `10.3.40.11` – `10.3.41.254` |
| C | 70 | Access point | `10.3.70.0/24` | `10.3.70.1` | `10.3.70.2` / `10.3.70.3` | `10.3.70.11` – `10.3.70.254` |
| C | 99 | Quản trị | `10.3.99.0/24` | `10.3.99.1` | `10.3.99.2` / `10.3.99.3` | IP tĩnh |
| K | 60 | Camera | `10.4.60.0/24` | `10.4.60.1` | `10.4.60.2` / `10.4.60.3` | `10.4.60.11` – `10.4.60.254` |
| K | 70 | Access point | `10.4.70.0/24` | `10.4.70.1` | `10.4.70.2` / `10.4.70.3` | `10.4.70.11` – `10.4.70.254` |
| K | 99 | Quản trị | `10.4.99.0/24` | `10.4.99.1` | `10.4.99.2` / `10.4.99.3` | IP tĩnh |

VLAN 999 dành cho cổng bỏ trống và làm native VLAN trên các trunk campus thông thường. Cổng WLC là ngoại lệ, dùng native VLAN 50. Pool DHCP cho AP cung cấp địa chỉ WLC `10.1.50.10`.

| Vùng ngoài campus | Subnet | Gateway hoặc cách cấp IP |
|---|---|---|
| DMZ | `10.100.0.0/24` | HSRP VIP `10.100.0.1`, hai router biên dùng `.2` và `.3` |
| Cơ sở 2 | `10.50.10.0/24` | BR-R1 `10.50.10.1`, BR-SW `10.50.10.2`, DHCP từ `10.50.10.11` |
| Remote Access VPN | `10.60.0.0/24` | Pool `10.60.0.10` – `10.60.0.110` |

### IP thiết bị và dịch vụ

| Thiết bị hoặc interface | Địa chỉ | Gateway hoặc ghi chú |
|---|---|---|
| DHCP-DNS | `10.1.50.5/24` | `10.1.50.1` |
| RADIUS | `10.1.50.6/24` | `10.1.50.1` |
| PORTAL-NB | `10.1.50.7/24` | `10.1.50.1` |
| WLC1 management | `10.1.50.10/24` | `10.1.50.1`, VLAN 50 |
| WLC1 vlan30 | `10.20.0.10/19` | `10.20.0.1`, SSID `UIT-SV` |
| WEB / LMS / E-LIB | `10.100.0.10/24` / `10.100.0.11/24` / `10.100.0.12/24` | `10.100.0.1` |
| PC-Remote | `113.161.2.2/30` | `113.161.2.1` |
| ASW-A1 / ASW-A2 | `10.1.99.11/24` / `10.1.99.12/24` | `10.1.99.1` |
| ASW-B1 / ASW-B2 | `10.2.99.11/24` / `10.2.99.12/24` | `10.2.99.1` |
| ASW-C1 / ASW-C2 | `10.3.99.11/24` / `10.3.99.12/24` | `10.3.99.1` |
| ASW-K1 / ASW-K2 | `10.4.99.11/24` / `10.4.99.12/24` | `10.4.99.1` |
| SRV-SW | `10.1.99.13/24` | `10.1.99.1` |

<details>
<summary>Loopback và IP các đường định tuyến</summary>

| Đầu A | IP A | Đầu B | IP B |
|---|---|---|---|
| CORE-SW1 Port-channel1 | `10.255.0.1/30` | CORE-SW2 Port-channel1 | `10.255.0.2/30` |
| ISP-Viettel GigabitEthernet0/0 | `203.113.1.1/30` | EDGE-R1 GigabitEthernet0/0 | `203.113.1.2/30` |
| ISP-VNPT GigabitEthernet0/0 | `113.161.1.1/30` | EDGE-R2 GigabitEthernet0/0 | `113.161.1.2/30` |
| ISP-Viettel GigabitEthernet0/1 | `100.64.0.1/30` | ISP-VNPT GigabitEthernet0/1 | `100.64.0.2/30` |
| ISP-Viettel GigabitEthernet0/2 | `203.113.2.1/30` | BR-R1 GigabitEthernet0/0 | `203.113.2.2/30` |
| ISP-VNPT GigabitEthernet0/2 | `113.161.2.1/30` | PC-Remote FastEthernet0 | `113.161.2.2/30` |
| EDGE-R1 GigabitEthernet0/1 | `10.255.3.1/30` | CORE-SW1 GigabitEthernet1/0/21 | `10.255.3.2/30` |
| EDGE-R2 GigabitEthernet0/1 | `10.255.3.5/30` | CORE-SW2 GigabitEthernet1/0/21 | `10.255.3.6/30` |
| CORE-SW1 GigabitEthernet1/0/1 | `10.255.1.1/30` | DSW-A1 GigabitEthernet1/0/1 | `10.255.1.2/30` |
| CORE-SW2 GigabitEthernet1/0/1 | `10.255.2.1/30` | DSW-A1 GigabitEthernet1/0/2 | `10.255.2.2/30` |
| CORE-SW1 GigabitEthernet1/0/2 | `10.255.1.5/30` | DSW-A2 GigabitEthernet1/0/1 | `10.255.1.6/30` |
| CORE-SW2 GigabitEthernet1/0/2 | `10.255.2.5/30` | DSW-A2 GigabitEthernet1/0/2 | `10.255.2.6/30` |
| CORE-SW1 GigabitEthernet1/0/3 | `10.255.1.9/30` | DSW-B1 GigabitEthernet1/0/1 | `10.255.1.10/30` |
| CORE-SW2 GigabitEthernet1/0/3 | `10.255.2.9/30` | DSW-B1 GigabitEthernet1/0/2 | `10.255.2.10/30` |
| CORE-SW1 GigabitEthernet1/0/4 | `10.255.1.13/30` | DSW-B2 GigabitEthernet1/0/1 | `10.255.1.14/30` |
| CORE-SW2 GigabitEthernet1/0/4 | `10.255.2.13/30` | DSW-B2 GigabitEthernet1/0/2 | `10.255.2.14/30` |
| CORE-SW1 GigabitEthernet1/0/5 | `10.255.1.17/30` | DSW-C1 GigabitEthernet1/0/1 | `10.255.1.18/30` |
| CORE-SW2 GigabitEthernet1/0/5 | `10.255.2.17/30` | DSW-C1 GigabitEthernet1/0/2 | `10.255.2.18/30` |
| CORE-SW1 GigabitEthernet1/0/6 | `10.255.1.21/30` | DSW-C2 GigabitEthernet1/0/1 | `10.255.1.22/30` |
| CORE-SW2 GigabitEthernet1/0/6 | `10.255.2.21/30` | DSW-C2 GigabitEthernet1/0/2 | `10.255.2.22/30` |
| CORE-SW1 GigabitEthernet1/0/7 | `10.255.1.25/30` | DSW-K1 GigabitEthernet1/0/1 | `10.255.1.26/30` |
| CORE-SW2 GigabitEthernet1/0/7 | `10.255.2.25/30` | DSW-K1 GigabitEthernet1/0/2 | `10.255.2.26/30` |
| CORE-SW1 GigabitEthernet1/0/8 | `10.255.1.29/30` | DSW-K2 GigabitEthernet1/0/1 | `10.255.1.30/30` |
| CORE-SW2 GigabitEthernet1/0/8 | `10.255.2.29/30` | DSW-K2 GigabitEthernet1/0/2 | `10.255.2.30/30` |

| Thiết bị | Loopback0 |
|---|---|
| CORE-SW1 | `10.255.255.1/32` |
| CORE-SW2 | `10.255.255.2/32` |
| DSW-A1 | `10.255.255.21/32` |
| DSW-A2 | `10.255.255.22/32` |
| DSW-B1 | `10.255.255.23/32` |
| DSW-B2 | `10.255.255.24/32` |
| DSW-C1 | `10.255.255.25/32` |
| DSW-C2 | `10.255.255.26/32` |
| DSW-K1 | `10.255.255.27/32` |
| DSW-K2 | `10.255.255.28/32` |
| ISP-Viettel | `8.8.8.8/32` |
| ISP-VNPT | `1.1.1.1/32` |
| EDGE-R1 | `10.255.255.11/32` |
| EDGE-R2 | `10.255.255.12/32` |

</details>

### Cổng và VLAN

Các cặp Core–Core dùng EtherChannel L3 trên Gi1/0/23–24. Các cặp DSW cùng khối dùng EtherChannel L2 trên Gi1/0/3–4. Uplink DSW–Core dùng cổng định tuyến.

<details>
<summary>Bảng 81 kết nối vật lý</summary>

| Thiết bị A | Cổng A | Thiết bị B | Cổng B | Cấu hình cổng |
|---|---|---|---|---|
| ASW-A1 | `FastEthernet0/1` | PC-BGH | `FastEthernet0` | Access VLAN 10 |
| ASW-A1 | `FastEthernet0/2` | PC-HanhChinh | `FastEthernet0` | Access VLAN 10 |
| ASW-A2 | `FastEthernet0/1` | PC-GV-A | `FastEthernet0` | Access VLAN 20 |
| ASW-A2 | `FastEthernet0/2` | CAM-A | `FastEthernet0` | Access VLAN 60 |
| ASW-B1 | `FastEthernet0/1` | PC-GV-B | `FastEthernet0` | Access VLAN 20 |
| ASW-B1 | `FastEthernet0/2` | LAP-B1 | `GigabitEthernet0` | Access VLAN 70 |
| ASW-B2 | `FastEthernet0/1` | LAP-B2 | `GigabitEthernet0` | Access VLAN 70 |
| ASW-B2 | `FastEthernet0/2` | CAM-B | `FastEthernet0` | Access VLAN 60 |
| ASW-B2 | `FastEthernet0/3` | PC-ThuVien | `FastEthernet0` | Access VLAN 80 |
| ASW-C1 | `FastEthernet0/1` | PC-Lab1 | `FastEthernet0` | Access VLAN 40 |
| ASW-C1 | `FastEthernet0/2` | PC-Lab2 | `FastEthernet0` | Access VLAN 40 |
| ASW-C1 | `FastEthernet0/3` | LAP-C1 | `GigabitEthernet0` | Access VLAN 70 |
| ASW-C2 | `FastEthernet0/1` | PC-Lab3 | `FastEthernet0` | Access VLAN 40 |
| ASW-C2 | `FastEthernet0/2` | PC-Lab4 | `FastEthernet0` | Access VLAN 40 |
| ASW-K1 | `FastEthernet0/1` | LAP-K1 | `GigabitEthernet0` | Access VLAN 70 |
| ASW-K1 | `FastEthernet0/2` | CAM-K1 | `FastEthernet0` | Access VLAN 60 |
| ASW-K2 | `FastEthernet0/1` | LAP-K2 | `GigabitEthernet0` | Access VLAN 70 |
| ASW-K2 | `FastEthernet0/2` | CAM-K2 | `FastEthernet0` | Access VLAN 60 |
| ISP-Viettel | `GigabitEthernet0/0` | EDGE-R1 | `GigabitEthernet0/0` | L3 `203.113.1.0/30` |
| ISP-VNPT | `GigabitEthernet0/0` | EDGE-R2 | `GigabitEthernet0/0` | L3 `113.161.1.0/30` |
| ISP-Viettel | `GigabitEthernet0/1` | ISP-VNPT | `GigabitEthernet0/1` | L3 `100.64.0.0/30` |
| ISP-Viettel | `GigabitEthernet0/2` | BR-R1 | `GigabitEthernet0/0` | L3 `203.113.2.0/30` |
| ISP-VNPT | `GigabitEthernet0/2` | PC-Remote | `FastEthernet0` | L3 `113.161.2.0/30` |
| BR-R1 | `GigabitEthernet0/1` | BR-SW | `GigabitEthernet0/1` | — |
| EDGE-R1 | `GigabitEthernet0/1` | CORE-SW1 | `GigabitEthernet1/0/21` | L3 `10.255.3.0/30` |
| EDGE-R2 | `GigabitEthernet0/1` | CORE-SW2 | `GigabitEthernet1/0/21` | L3 `10.255.3.4/30` |
| EDGE-R1 | `GigabitEthernet0/2` | DMZ-SW | `GigabitEthernet0/1` | — |
| EDGE-R2 | `GigabitEthernet0/2` | DMZ-SW | `GigabitEthernet0/2` | — |
| CORE-SW1 | `GigabitEthernet1/0/23` | CORE-SW2 | `GigabitEthernet1/0/23` | — |
| CORE-SW1 | `GigabitEthernet1/0/24` | CORE-SW2 | `GigabitEthernet1/0/24` | — |
| CORE-SW1 | `GigabitEthernet1/0/1` | DSW-A1 | `GigabitEthernet1/0/1` | L3 `10.255.1.0/30` |
| CORE-SW2 | `GigabitEthernet1/0/1` | DSW-A1 | `GigabitEthernet1/0/2` | L3 `10.255.2.0/30` |
| CORE-SW1 | `GigabitEthernet1/0/2` | DSW-A2 | `GigabitEthernet1/0/1` | L3 `10.255.1.4/30` |
| CORE-SW2 | `GigabitEthernet1/0/2` | DSW-A2 | `GigabitEthernet1/0/2` | L3 `10.255.2.4/30` |
| DSW-A1 | `GigabitEthernet1/0/3` | DSW-A2 | `GigabitEthernet1/0/3` | Trunk |
| DSW-A1 | `GigabitEthernet1/0/4` | DSW-A2 | `GigabitEthernet1/0/4` | Trunk |
| DSW-A1 | `GigabitEthernet1/0/5` | ASW-A1 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 10,20,60,99 |
| DSW-A1 | `GigabitEthernet1/0/6` | ASW-A2 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 10,20,60,99 |
| DSW-A2 | `GigabitEthernet1/0/5` | ASW-A1 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 10,20,60,99 |
| DSW-A2 | `GigabitEthernet1/0/6` | ASW-A2 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 10,20,60,99 |
| CORE-SW1 | `GigabitEthernet1/0/3` | DSW-B1 | `GigabitEthernet1/0/1` | L3 `10.255.1.8/30` |
| CORE-SW2 | `GigabitEthernet1/0/3` | DSW-B1 | `GigabitEthernet1/0/2` | L3 `10.255.2.8/30` |
| CORE-SW1 | `GigabitEthernet1/0/4` | DSW-B2 | `GigabitEthernet1/0/1` | L3 `10.255.1.12/30` |
| CORE-SW2 | `GigabitEthernet1/0/4` | DSW-B2 | `GigabitEthernet1/0/2` | L3 `10.255.2.12/30` |
| DSW-B1 | `GigabitEthernet1/0/3` | DSW-B2 | `GigabitEthernet1/0/3` | Trunk |
| DSW-B1 | `GigabitEthernet1/0/4` | DSW-B2 | `GigabitEthernet1/0/4` | Trunk |
| DSW-B1 | `GigabitEthernet1/0/5` | ASW-B1 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 20,60,70,80,99 |
| DSW-B1 | `GigabitEthernet1/0/6` | ASW-B2 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 20,60,70,80,99 |
| DSW-B2 | `GigabitEthernet1/0/5` | ASW-B1 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 20,60,70,80,99 |
| DSW-B2 | `GigabitEthernet1/0/6` | ASW-B2 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 20,60,70,80,99 |
| CORE-SW1 | `GigabitEthernet1/0/5` | DSW-C1 | `GigabitEthernet1/0/1` | L3 `10.255.1.16/30` |
| CORE-SW2 | `GigabitEthernet1/0/5` | DSW-C1 | `GigabitEthernet1/0/2` | L3 `10.255.2.16/30` |
| CORE-SW1 | `GigabitEthernet1/0/6` | DSW-C2 | `GigabitEthernet1/0/1` | L3 `10.255.1.20/30` |
| CORE-SW2 | `GigabitEthernet1/0/6` | DSW-C2 | `GigabitEthernet1/0/2` | L3 `10.255.2.20/30` |
| DSW-C1 | `GigabitEthernet1/0/3` | DSW-C2 | `GigabitEthernet1/0/3` | Trunk |
| DSW-C1 | `GigabitEthernet1/0/4` | DSW-C2 | `GigabitEthernet1/0/4` | Trunk |
| DSW-C1 | `GigabitEthernet1/0/5` | ASW-C1 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 40,70,99 |
| DSW-C1 | `GigabitEthernet1/0/6` | ASW-C2 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 40,70,99 |
| DSW-C2 | `GigabitEthernet1/0/5` | ASW-C1 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 40,70,99 |
| DSW-C2 | `GigabitEthernet1/0/6` | ASW-C2 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 40,70,99 |
| CORE-SW1 | `GigabitEthernet1/0/7` | DSW-K1 | `GigabitEthernet1/0/1` | L3 `10.255.1.24/30` |
| CORE-SW2 | `GigabitEthernet1/0/7` | DSW-K1 | `GigabitEthernet1/0/2` | L3 `10.255.2.24/30` |
| CORE-SW1 | `GigabitEthernet1/0/8` | DSW-K2 | `GigabitEthernet1/0/1` | L3 `10.255.1.28/30` |
| CORE-SW2 | `GigabitEthernet1/0/8` | DSW-K2 | `GigabitEthernet1/0/2` | L3 `10.255.2.28/30` |
| DSW-K1 | `GigabitEthernet1/0/3` | DSW-K2 | `GigabitEthernet1/0/3` | Trunk |
| DSW-K1 | `GigabitEthernet1/0/4` | DSW-K2 | `GigabitEthernet1/0/4` | Trunk |
| DSW-K1 | `GigabitEthernet1/0/5` | ASW-K1 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 60,70,99 |
| DSW-K1 | `GigabitEthernet1/0/6` | ASW-K2 | `GigabitEthernet0/1` | Trunk, native 999, cho phép 60,70,99 |
| DSW-K2 | `GigabitEthernet1/0/5` | ASW-K1 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 60,70,99 |
| DSW-K2 | `GigabitEthernet1/0/6` | ASW-K2 | `GigabitEthernet0/2` | Trunk, native 999, cho phép 60,70,99 |
| DSW-A1 | `GigabitEthernet1/0/7` | SRV-SW | `GigabitEthernet0/1` | Trunk, native 999, cho phép 30,50,99 |
| DSW-A2 | `GigabitEthernet1/0/7` | SRV-SW | `GigabitEthernet0/2` | Trunk, native 999, cho phép 30,50,99 |
| SRV-SW | `FastEthernet0/1` | DHCP-DNS | `FastEthernet0` | Access VLAN 50 |
| SRV-SW | `FastEthernet0/2` | RADIUS | `FastEthernet0` | Access VLAN 50 |
| SRV-SW | `FastEthernet0/3` | PORTAL-NB | `FastEthernet0` | Access VLAN 50 |
| SRV-SW | `FastEthernet0/4` | WLC1 | `GigabitEthernet1` | Trunk, native 50, cho phép 30,50 |
| DMZ-SW | `FastEthernet0/1` | WEB | `FastEthernet0` | — |
| DMZ-SW | `FastEthernet0/2` | LMS | `FastEthernet0` | — |
| DMZ-SW | `FastEthernet0/3` | E-LIB | `FastEthernet0` | — |
| BR-SW | `FastEthernet0/1` | PC-BR1 | `FastEthernet0` | — |
| BR-SW | `FastEthernet0/2` | PC-BR2 | `FastEthernet0` | — |

</details>

---

## Phân công và triển khai

Nguồn số liệu duy nhất: Sheet **"Quy hoạch VLAN & IP – Mạng Campus (Đề tài 1)"** (tab *Subnet*, *IP thiết bị*, *Cổng & VLAN*). Thấy sai IP/VLAN thì sửa trên Sheet trước, rồi sửa cấu hình.

| Người | Phạm vi | Thiết bị cấu hình bằng CLI | Cấu hình bằng giao diện |
|---|---|---|---|
| **Khánh** | Bên trong campus: VLAN, trunk, EtherChannel L2, gateway HSRP, DHCP, Wi-Fi, bảo mật lớp 2, ACL ở Distribution | 8 DSW (phần L2 + SVI), 8 ASW, SRV-SW | DHCP-DNS, RADIUS, PORTAL-NB, WLC1, PC, camera, smartphone |
| **Khang** | Định tuyến và biên: Core, OSPF, Dual-ISP, NAT, DMZ, VPN, ACL ở biên; **giữ file master** | CORE-SW1/2, 8 DSW (phần uplink + OSPF), EDGE-R1/2, ISP-Viettel/VNPT, BR-R1, BR-SW | WEB, LMS, E-LIB, PC-Remote, PC-BR1/2 |

---

## 0. Quy tắc làm song song (cả hai đọc trước)

Packet Tracer không gộp được hai file `.pkt`, nên cấu hình CLI được lưu thành **file text**, không làm thẳng trên file master.

1. Mỗi người làm trên **bản copy** của `Campus-Topology.pkt` để thử.
2. Mỗi khối lệnh trong tài liệu này là một file text trong thư mục chung (Drive/Git):
```
configs/
  DSW-A1.L2.txt        ← Khánh
  DSW-A1.routing.txt   ← Khang
  ASW-A1.txt           ← Khánh
  CORE-SW1.txt         ← Khang
  ...
```

3. DSW là thiết bị duy nhất hai người cùng đụng: Khánh dán file `.L2`, Khang dán file `.routing`. Hai file không trùng lệnh nào, dán theo thứ tự nào cũng được.
4. Cần phần của người kia để test thì dán **cả hai** file vào bản copy của mình.
5. Cấu hình bằng giao diện (Server, WLC, PC, smartphone) không dán được, phải làm trên file master. **Mỗi lần chỉ một người mở file master**; xong thì lưu thành `Campus-Topology_ngayX_ten.pkt` và báo người kia.
6. Cách dán: vào tab CLI → `enable` → `configure terminal` → dán **từng khối nhỏ** (giữa hai dòng `!`), đợi chạy xong mới dán khối tiếp. Dán xong chạy `show running-config` đối chiếu.
7. Mọi khối lệnh đã có sẵn phần cấu hình chung: `enable secret <ENABLE_SECRET>`, `no ip domain-lookup`, banner, `logging synchronous`.

---

## 1. Khánh – Campus nội bộ

### 1.1 Distribution: VLAN, EtherChannel, trunk, SVI + HSRP, ACL (8 DSW)

- DSW số 1 của mỗi cặp là HSRP **Active** (priority 110) và STP **root primary**; DSW số 2 là Standby/secondary.
- Gateway của mỗi VLAN là địa chỉ ảo `.1`; DSW số 1 dùng `.2`, DSW số 2 dùng `.3`.
- `ip helper-address` chỉ đặt trên VLAN có máy nhận DHCP (không đặt cho VLAN 50, 99).
- ACL `CAM-IN` có dòng cho phép `224.0.0.102 udp 1985`: thiếu dòng này HSRP hello giữa hai DSW bị chặn và cả hai cùng thành Active.
- Nếu IOS báo lỗi ở `switchport mode trunk`, thêm `switchport trunk encapsulation dot1q` ngay trước dòng đó.

#### Khối A – Hiệu bộ

**DSW-A1** → `configs/DSW-A1.L2.txt`

```
hostname DSW-A1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 10
 name BGH-HanhChinh
 exit
vlan 20
 name GiangVien
 exit
vlan 30
 name SinhVien-WiFi
 exit
vlan 50
 name MayChu
 exit
vlan 60
 name Camera
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,50,60,99 root primary
!
! --- EtherChannel L2 sang DSW-A2 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,50,60,99
 exit
!
! --- Trunk xuống ASW-A1 (G1/0/5), ASW-A2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,60,99
 no shutdown
 exit
!
! --- Trunk xuống SRV-SW ---
interface GigabitEthernet1/0/7
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,50,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan10
 ip address 10.1.10.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 10 ip 10.1.10.1
 standby 10 priority 110
 standby 10 preempt
 no shutdown
 exit
interface Vlan20
 ip address 10.1.20.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 20 ip 10.1.20.1
 standby 20 priority 110
 standby 20 preempt
 no shutdown
 exit
interface Vlan30
 ip address 10.20.0.2 255.255.224.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 30 ip 10.20.0.1
 standby 30 priority 110
 standby 30 preempt
 no shutdown
 exit
interface Vlan50
 ip address 10.1.50.2 255.255.255.0
 standby version 2
 standby 50 ip 10.1.50.1
 standby 50 priority 110
 standby 50 preempt
 no shutdown
 exit
interface Vlan60
 ip address 10.1.60.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.1.60.1
 standby 60 priority 110
 standby 60 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.1.99.2 255.255.255.0
 standby version 2
 standby 99 ip 10.1.99.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Wi-Fi SV không vào BGH-HC và mạng quản trị ---
ip access-list extended SV-WIFI-IN
 permit udp any any eq bootps
 deny ip 10.20.0.0 0.0.31.255 10.1.10.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.1.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.2.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.3.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.4.99.0 0.0.0.255
 permit ip any any
 exit
interface Vlan30
 ip access-group SV-WIFI-IN in
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.1.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

**DSW-A2** → `configs/DSW-A2.L2.txt`

```
hostname DSW-A2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 10
 name BGH-HanhChinh
 exit
vlan 20
 name GiangVien
 exit
vlan 30
 name SinhVien-WiFi
 exit
vlan 50
 name MayChu
 exit
vlan 60
 name Camera
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,50,60,99 root secondary
!
! --- EtherChannel L2 sang DSW-A1 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,50,60,99
 exit
!
! --- Trunk xuống ASW-A1 (G1/0/5), ASW-A2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,60,99
 no shutdown
 exit
!
! --- Trunk xuống SRV-SW ---
interface GigabitEthernet1/0/7
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,50,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan10
 ip address 10.1.10.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 10 ip 10.1.10.1
 standby 10 priority 100
 standby 10 preempt
 no shutdown
 exit
interface Vlan20
 ip address 10.1.20.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 20 ip 10.1.20.1
 standby 20 priority 100
 standby 20 preempt
 no shutdown
 exit
interface Vlan30
 ip address 10.20.0.3 255.255.224.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 30 ip 10.20.0.1
 standby 30 priority 100
 standby 30 preempt
 no shutdown
 exit
interface Vlan50
 ip address 10.1.50.3 255.255.255.0
 standby version 2
 standby 50 ip 10.1.50.1
 standby 50 priority 100
 standby 50 preempt
 no shutdown
 exit
interface Vlan60
 ip address 10.1.60.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.1.60.1
 standby 60 priority 100
 standby 60 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.1.99.3 255.255.255.0
 standby version 2
 standby 99 ip 10.1.99.1
 standby 99 priority 100
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Wi-Fi SV không vào BGH-HC và mạng quản trị ---
ip access-list extended SV-WIFI-IN
 permit udp any any eq bootps
 deny ip 10.20.0.0 0.0.31.255 10.1.10.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.1.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.2.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.3.99.0 0.0.0.255
 deny ip 10.20.0.0 0.0.31.255 10.4.99.0 0.0.0.255
 permit ip any any
 exit
interface Vlan30
 ip access-group SV-WIFI-IN in
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.1.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

#### Khối B – Giảng đường + Thư viện

**DSW-B1** → `configs/DSW-B1.L2.txt`

```
hostname DSW-B1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 80
 name ThuVien
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 20,60,70,80,99 root primary
!
! --- EtherChannel L2 sang DSW-B2 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 exit
!
! --- Trunk xuống ASW-B1 (G1/0/5), ASW-B2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan20
 ip address 10.2.20.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 20 ip 10.2.20.1
 standby 20 priority 110
 standby 20 preempt
 no shutdown
 exit
interface Vlan60
 ip address 10.2.60.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.2.60.1
 standby 60 priority 110
 standby 60 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.2.70.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.2.70.1
 standby 70 priority 110
 standby 70 preempt
 no shutdown
 exit
interface Vlan80
 ip address 10.2.80.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 80 ip 10.2.80.1
 standby 80 priority 110
 standby 80 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.2.99.2 255.255.255.0
 standby version 2
 standby 99 ip 10.2.99.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.2.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

**DSW-B2** → `configs/DSW-B2.L2.txt`

```
hostname DSW-B2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 80
 name ThuVien
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 20,60,70,80,99 root secondary
!
! --- EtherChannel L2 sang DSW-B1 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 exit
!
! --- Trunk xuống ASW-B1 (G1/0/5), ASW-B2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan20
 ip address 10.2.20.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 20 ip 10.2.20.1
 standby 20 priority 100
 standby 20 preempt
 no shutdown
 exit
interface Vlan60
 ip address 10.2.60.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.2.60.1
 standby 60 priority 100
 standby 60 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.2.70.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.2.70.1
 standby 70 priority 100
 standby 70 preempt
 no shutdown
 exit
interface Vlan80
 ip address 10.2.80.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 80 ip 10.2.80.1
 standby 80 priority 100
 standby 80 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.2.99.3 255.255.255.0
 standby version 2
 standby 99 ip 10.2.99.1
 standby 99 priority 100
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.2.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

#### Khối C – Thực hành

**DSW-C1** → `configs/DSW-C1.L2.txt`

```
hostname DSW-C1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 40
 name PhongThucHanh
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 40,70,99 root primary
!
! --- EtherChannel L2 sang DSW-C2 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 exit
!
! --- Trunk xuống ASW-C1 (G1/0/5), ASW-C2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan40
 ip address 10.3.40.2 255.255.254.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 40 ip 10.3.40.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.3.70.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.3.70.1
 standby 70 priority 110
 standby 70 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.3.99.2 255.255.255.0
 standby version 2
 standby 99 ip 10.3.99.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
 exit
end
write memory
```

**DSW-C2** → `configs/DSW-C2.L2.txt`

```
hostname DSW-C2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 40
 name PhongThucHanh
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 40,70,99 root secondary
!
! --- EtherChannel L2 sang DSW-C1 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 exit
!
! --- Trunk xuống ASW-C1 (G1/0/5), ASW-C2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan40
 ip address 10.3.40.3 255.255.254.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 40 ip 10.3.40.1
 standby 40 priority 100
 standby 40 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.3.70.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.3.70.1
 standby 70 priority 100
 standby 70 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.3.99.3 255.255.255.0
 standby version 2
 standby 99 ip 10.3.99.1
 standby 99 priority 100
 standby 99 preempt
 no shutdown
 exit
end
write memory
```

#### Ký túc xá

**DSW-K1** → `configs/DSW-K1.L2.txt`

```
hostname DSW-K1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 60,70,99 root primary
!
! --- EtherChannel L2 sang DSW-K2 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 exit
!
! --- Trunk xuống ASW-K1 (G1/0/5), ASW-K2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan60
 ip address 10.4.60.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.4.60.1
 standby 60 priority 110
 standby 60 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.4.70.2 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.4.70.1
 standby 70 priority 110
 standby 70 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.4.99.2 255.255.255.0
 standby version 2
 standby 99 ip 10.4.99.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.4.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

**DSW-K2** → `configs/DSW-K2.L2.txt`

```
hostname DSW-K2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
vtp mode transparent
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
spanning-tree mode rapid-pvst
spanning-tree vlan 60,70,99 root secondary
!
! --- EtherChannel L2 sang DSW-K1 ---
interface range GigabitEthernet1/0/3-4
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 exit
!
! --- Trunk xuống ASW-K1 (G1/0/5), ASW-K2 (G1/0/6) ---
interface range GigabitEthernet1/0/5-6
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 no shutdown
 exit
!
! --- SVI + HSRP ---
interface Vlan60
 ip address 10.4.60.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 60 ip 10.4.60.1
 standby 60 priority 100
 standby 60 preempt
 no shutdown
 exit
interface Vlan70
 ip address 10.4.70.3 255.255.255.0
 ip helper-address 10.1.50.5
 standby version 2
 standby 70 ip 10.4.70.1
 standby 70 priority 100
 standby 70 preempt
 no shutdown
 exit
interface Vlan99
 ip address 10.4.99.3 255.255.255.0
 standby version 2
 standby 99 ip 10.4.99.1
 standby 99 priority 100
 standby 99 preempt
 no shutdown
 exit
!
! --- ACL: Camera chỉ tới vùng máy chủ ---
ip access-list extended CAM-IN
 permit udp any any eq bootps
 permit udp any host 224.0.0.102 eq 1985
 permit ip 10.4.60.0 0.0.0.255 10.1.50.0 0.0.0.255
 deny ip any any
 exit
interface Vlan60
 ip access-group CAM-IN in
 exit
end
write memory
```

### 1.2 Access: trunk, access port, Port Security, DHCP Snooping (8 ASW)

- `no ip dhcp snooping information option` là **bắt buộc**: thiếu dòng này switch chèn option 82 và DSW bỏ gói DHCP relay, PC không nhận được IP.
- Khối DAI ở cuối mỗi switch: nếu Packet Tracer không nhận lệnh thì bỏ qua, ghi vào báo cáo ở mức thiết kế.

**ASW-A1** → `configs/ASW-A1.txt`

```
hostname ASW-A1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 10
 name BGH-HanhChinh
 exit
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,60,99
 no shutdown
 exit
!
! --- PC-BGH, PC-HanhChinh ---
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.1.99.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.1.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 10
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 10
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-A2** → `configs/ASW-A2.txt`

```
hostname ASW-A2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 10
 name BGH-HanhChinh
 exit
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,60,99
 no shutdown
 exit
!
! --- PC-GV-A ---
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- CAM-A ---
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.1.99.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.1.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 20,60
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 20,60
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-B1** → `configs/ASW-B1.txt`

```
hostname ASW-B1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 80
 name ThuVien
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 no shutdown
 exit
!
! --- PC-GV-B ---
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- LAP-B1 ---
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.2.99.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.2.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 20,70
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 20,70
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-B2** → `configs/ASW-B2.txt`

```
hostname ASW-B2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 20
 name GiangVien
 exit
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 80
 name ThuVien
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,60,70,80,99
 no shutdown
 exit
!
! --- LAP-B2 ---
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- CAM-B ---
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- PC-ThuVien ---
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 80
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/4-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.2.99.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.2.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 60,70,80
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 60,70,80
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-C1** → `configs/ASW-C1.txt`

```
hostname ASW-C1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 40
 name PhongThucHanh
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 no shutdown
 exit
!
! --- PC-Lab1, PC-Lab2 ---
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- LAP-C1 ---
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/4-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.3.99.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.3.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 40,70
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 40,70
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-C2** → `configs/ASW-C2.txt`

```
hostname ASW-C2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 40
 name PhongThucHanh
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 40,70,99
 no shutdown
 exit
!
! --- PC-Lab3, PC-Lab4 ---
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.3.99.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.3.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 40
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 40
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-K1** → `configs/ASW-K1.txt`

```
hostname ASW-K1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 no shutdown
 exit
!
! --- LAP-K1 ---
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- CAM-K1 ---
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.4.99.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.4.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 60,70
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 60,70
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

**ASW-K2** → `configs/ASW-K2.txt`

```
hostname ASW-K2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 60
 name Camera
 exit
vlan 70
 name AccessPoint
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW (G0/1 → DSW-x1, G0/2 → DSW-x2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 60,70,99
 no shutdown
 exit
!
! --- LAP-K2 ---
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- CAM-K2 ---
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
! --- IP quản trị ---
interface Vlan99
 ip address 10.4.99.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.4.99.1
!
! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 60,70
no ip dhcp snooping information option
interface range GigabitEthernet0/1-2
 ip dhcp snooping trust
 exit
!
! --- DAI (thử; nếu PT báo lỗi thì bỏ khối này) ---
ip arp inspection vlan 60,70
interface range GigabitEthernet0/1-2
 ip arp inspection trust
 exit
end
write memory
```

### 1.3 SRV-SW

**SRV-SW** → `configs/SRV-SW.txt`

```
hostname SRV-SW
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 no ip address
 shutdown
 exit
!
vtp mode transparent
vlan 30
 name SinhVien-WiFi
 exit
vlan 50
 name MayChu
 exit
vlan 99
 name QuanTri
 exit
vlan 999
 name BlackHole
 exit
!
! --- Uplink lên DSW-A1 (G0/1), DSW-A2 (G0/2) ---
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,50,99
 no shutdown
 exit
!
! --- DHCP-DNS, RADIUS, PORTAL-NB ---
interface range FastEthernet0/1-3
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown
 exit
!
! --- WLC1: trunk, management untagged trong VLAN 50 ---
interface FastEthernet0/4
 switchport mode trunk
 switchport trunk native vlan 50
 switchport trunk allowed vlan 30,50
 no shutdown
 exit
!
! --- Cổng không dùng ---
interface range FastEthernet0/5-24
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
!
interface Vlan99
 ip address 10.1.99.13 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.1.99.1
end
write memory
```

### 1.4 Server DHCP-DNS (làm trên file master)

1. Desktop → IP Configuration: Static, IP `10.1.50.5`, mask `255.255.255.0`, gateway `10.1.50.1`, DNS `10.1.50.5`.
2. Services → DHCP → bật **On**. Sửa `serverPool`: gateway `10.1.50.1`, DNS `10.1.50.5`, Start IP `10.1.50.200`, mask `255.255.255.0`, Max Users `10` → **Save**.
3. Thêm 12 pool, mỗi pool điền đủ các ô rồi bấm **Add**:

| Pool Name | Default Gateway | DNS | Start IP | Subnet Mask | Max Users | WLC Address |
|---|---|---|---|---|---|---|
| A-BGH | 10.1.10.1 | 10.1.50.5 | 10.1.10.11 | 255.255.255.0 | 244 | — |
| A-GV | 10.1.20.1 | 10.1.50.5 | 10.1.20.11 | 255.255.255.0 | 244 | — |
| A-CAM | 10.1.60.1 | 10.1.50.5 | 10.1.60.11 | 255.255.255.0 | 244 | — |
| SV-WIFI | 10.20.0.1 | 10.1.50.5 | 10.20.0.11 | 255.255.224.0 | 8180 | — |
| B-GV | 10.2.20.1 | 10.1.50.5 | 10.2.20.11 | 255.255.255.0 | 244 | — |
| B-CAM | 10.2.60.1 | 10.1.50.5 | 10.2.60.11 | 255.255.255.0 | 244 | — |
| B-AP | 10.2.70.1 | 10.1.50.5 | 10.2.70.11 | 255.255.255.0 | 244 | 10.1.50.10 |
| B-THUVIEN | 10.2.80.1 | 10.1.50.5 | 10.2.80.11 | 255.255.255.0 | 244 | — |
| C-LAB | 10.3.40.1 | 10.1.50.5 | 10.3.40.11 | 255.255.254.0 | 500 | — |
| C-AP | 10.3.70.1 | 10.1.50.5 | 10.3.70.11 | 255.255.255.0 | 244 | 10.1.50.10 |
| K-CAM | 10.4.60.1 | 10.1.50.5 | 10.4.60.11 | 255.255.255.0 | 244 | — |
| K-AP | 10.4.70.1 | 10.1.50.5 | 10.4.70.11 | 255.255.255.0 | 244 | 10.1.50.10 |

4. Services → DNS → bật **On**, thêm 3 bản ghi loại A:

| Name | Address |
|---|---|
| www.campus.edu.vn | 10.100.0.10 |
| lms.campus.edu.vn | 10.100.0.11 |
| elib.campus.edu.vn | 10.100.0.12 |

### 1.5 RADIUS và PORTAL-NB (làm trên file master)

**RADIUS:** IP `10.1.50.6/24`, gateway `10.1.50.1`, DNS `10.1.50.5`. Services → AAA → **On**:
- Network Configuration: Client Name `WLC1`, Client IP `10.1.50.10`, Secret `<RADIUS_SHARED_SECRET>`, Server Type `Radius` → **Add**.
- User Setup: `sv001` / `<SV001_PASSWORD>`, `sv002` / `<SV002_PASSWORD>`, `sv003` / `<SV003_PASSWORD>` → **Add**.

**PORTAL-NB:** IP `10.1.50.7/24`, gateway `10.1.50.1`, DNS `10.1.50.5`. Services → HTTP **On**, sửa `index.html` thành trang "Cổng thông tin nội bộ".

### 1.6 Wi-Fi: WLC, AP, smartphone (làm trên file master, **sau khi Khang xong định tuyến**)

1. **WLC1** → tab Config → Management: IP `10.1.50.10`, mask `255.255.255.0`, gateway `10.1.50.1`.
2. Trên **PC-BGH** (đã nhận DHCP): Desktop → Web Browser → `https://10.1.50.10` → tạo tài khoản admin (`admin` / `<WLC_ADMIN_PASSWORD>`) → đăng nhập.
3. **Controller → Interfaces → New:** Interface Name `vlan30`, VLAN Id `30` → Apply. Điền IP `10.20.0.10`, Netmask `255.255.224.0`, Gateway `10.20.0.1`, Primary DHCP Server `10.1.50.5`, Port Number `1` → Apply.
4. **Security → AAA → RADIUS → Authentication → New:** Server IP `10.1.50.6`, Shared Secret `<RADIUS_SHARED_SECRET>` → Apply.
5. **WLANs → Create New:** Profile và SSID `UIT-SV` → Apply.
   - Tab General: Status **Enabled**, Interface `vlan30`.
   - Tab Security → Layer 2: **WPA+WPA2**, WPA2 Policy + AES, Auth Key Mgmt **802.1X**.
   - Tab Security → AAA Servers: chọn server `10.1.50.6` → Apply.
6. Đợi 1–2 phút, vào **Wireless → All APs** → phải thấy 5 AP (LAP-B1, B2, C1, K1, K2). AP nào chưa lên: kiểm tra AP đã nhận IP VLAN 70 chưa (di chuột lên AP).
7. **SV-Phone1/2/3** → Config → Wireless0: SSID `UIT-SV`, Authentication **WPA2-Enterprise**, User ID `sv001` (`sv002`, `sv003`), Password tương ứng, IP Configuration **DHCP** → phải nhận `10.20.x.x`.

> Captive Portal: Packet Tracer không hỗ trợ web-auth đầy đủ → demo bằng WPA2-Enterprise qua RADIUS, báo cáo trình bày Captive Portal ở mức thiết kế.

### 1.7 Thiết bị đầu cuối (làm trên file master)

- Tất cả PC, PC-Lab, PC-ThuVien: Desktop → IP Configuration → **DHCP**.
- CAM-A, CAM-B, CAM-K1, CAM-K2: Config → FastEthernet0 → **DHCP**.
- AP (LAP-…) không cần cấu hình: tự lấy IP VLAN 70 và tìm WLC.

### 1.8 Tự kiểm tra

Trước khi gửi file text hoặc báo xong phần giao diện, Khánh tự chạy các bước ở **mục 6.1** tương ứng với ngày đó.


---

## 2. Khang – Định tuyến, biên, VPN

### 2.1 Core

- Core không đặt ACL (đề yêu cầu). Toàn bộ link là cổng định tuyến /30 chạy OSPF area 0.

**CORE-SW1** → `configs/CORE-SW1.txt`

```
hostname CORE-SW1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
! --- EtherChannel L3 sang CORE-SW2 ---
interface range GigabitEthernet1/0/23-24
 no switchport
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 no switchport
 ip address 10.255.0.1 255.255.255.252
 exit
!
! --- Link xuống 8 DSW ---
interface GigabitEthernet1/0/1
 description -> DSW-A1
 no switchport
 ip address 10.255.1.1 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> DSW-A2
 no switchport
 ip address 10.255.1.5 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/3
 description -> DSW-B1
 no switchport
 ip address 10.255.1.9 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/4
 description -> DSW-B2
 no switchport
 ip address 10.255.1.13 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/5
 description -> DSW-C1
 no switchport
 ip address 10.255.1.17 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/6
 description -> DSW-C2
 no switchport
 ip address 10.255.1.21 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/7
 description -> DSW-K1
 no switchport
 ip address 10.255.1.25 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/8
 description -> DSW-K2
 no switchport
 ip address 10.255.1.29 255.255.255.252
 no shutdown
 exit
!
! --- Link lên EDGE-R1 ---
interface GigabitEthernet1/0/21
 description -> EDGE-R1
 no switchport
 ip address 10.255.3.2 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.1 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.1
 network 10.255.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**CORE-SW2** → `configs/CORE-SW2.txt`

```
hostname CORE-SW2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
ip routing
!
! --- EtherChannel L3 sang CORE-SW1 ---
interface range GigabitEthernet1/0/23-24
 no switchport
 channel-group 1 mode active
 no shutdown
 exit
interface Port-channel1
 no switchport
 ip address 10.255.0.2 255.255.255.252
 exit
!
! --- Link xuống 8 DSW ---
interface GigabitEthernet1/0/1
 description -> DSW-A1
 no switchport
 ip address 10.255.2.1 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> DSW-A2
 no switchport
 ip address 10.255.2.5 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/3
 description -> DSW-B1
 no switchport
 ip address 10.255.2.9 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/4
 description -> DSW-B2
 no switchport
 ip address 10.255.2.13 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/5
 description -> DSW-C1
 no switchport
 ip address 10.255.2.17 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/6
 description -> DSW-C2
 no switchport
 ip address 10.255.2.21 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/7
 description -> DSW-K1
 no switchport
 ip address 10.255.2.25 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/8
 description -> DSW-K2
 no switchport
 ip address 10.255.2.29 255.255.255.252
 no shutdown
 exit
!
! --- Link lên EDGE-R2 ---
interface GigabitEthernet1/0/21
 description -> EDGE-R2
 no switchport
 ip address 10.255.3.6 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.2 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.2
 network 10.255.0.0 0.0.255.255 area 0
 exit
end
write memory
```

### 2.2 Phần định tuyến của 8 DSW

- Dán **sau** file `.L2` của Khánh hoặc trước cũng được, hai file không trùng lệnh.
- `passive-interface default` để OSPF không chạy trên các SVI; chỉ G1/0/1-2 lên Core mới có neighbor.

**DSW-A1** → `configs/DSW-A1.routing.txt`

```
! DSW-A1 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.2 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.2 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.21 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.21
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.1.0.0 0.0.255.255 area 0
 network 10.20.0.0 0.0.31.255 area 0
 exit
end
write memory
```

**DSW-A2** → `configs/DSW-A2.routing.txt`

```
! DSW-A2 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.6 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.6 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.22 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.22
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.1.0.0 0.0.255.255 area 0
 network 10.20.0.0 0.0.31.255 area 0
 exit
end
write memory
```

**DSW-B1** → `configs/DSW-B1.routing.txt`

```
! DSW-B1 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.10 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.10 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.23 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.23
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.2.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**DSW-B2** → `configs/DSW-B2.routing.txt`

```
! DSW-B2 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.14 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.14 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.24 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.24
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.2.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**DSW-C1** → `configs/DSW-C1.routing.txt`

```
! DSW-C1 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.18 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.18 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.25 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.25
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.3.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**DSW-C2** → `configs/DSW-C2.routing.txt`

```
! DSW-C2 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.22 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.22 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.26 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.26
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.3.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**DSW-K1** → `configs/DSW-K1.routing.txt`

```
! DSW-K1 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.26 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.26 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.27 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.27
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.4.0.0 0.0.255.255 area 0
 exit
end
write memory
```

**DSW-K2** → `configs/DSW-K2.routing.txt`

```
! DSW-K2 – phần định tuyến
interface GigabitEthernet1/0/1
 description -> CORE-SW1
 no switchport
 ip address 10.255.1.30 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet1/0/2
 description -> CORE-SW2
 no switchport
 ip address 10.255.2.30 255.255.255.252
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.28 255.255.255.255
 exit
!
router ospf 1
 router-id 10.255.255.28
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2
 network 10.255.0.0 0.0.255.255 area 0
 network 10.4.0.0 0.0.255.255 area 0
 exit
end
write memory
```

### 2.3 ISP giả lập

**ISP-Viettel** → `configs/ISP-Viettel.txt`

```
hostname ISP-Viettel
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface GigabitEthernet0/0
 description -> EDGE-R1
 ip address 203.113.1.1 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/1
 description -> ISP-VNPT
 ip address 100.64.0.1 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/2
 description -> BR-R1 (Co so 2)
 ip address 203.113.2.1 255.255.255.252
 no shutdown
 exit
interface Loopback0
 description Internet gia lap
 ip address 8.8.8.8 255.255.255.255
 exit
!
ip route 203.113.100.0 255.255.255.248 203.113.1.2
ip route 113.161.0.0 255.255.0.0 100.64.0.2
ip route 1.1.1.1 255.255.255.255 100.64.0.2
end
write memory
```

**ISP-VNPT** → `configs/ISP-VNPT.txt`

```
hostname ISP-VNPT
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface GigabitEthernet0/0
 description -> EDGE-R2
 ip address 113.161.1.1 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/1
 description -> ISP-Viettel
 ip address 100.64.0.2 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/2
 description -> PC-Remote
 ip address 113.161.2.1 255.255.255.252
 no shutdown
 exit
interface Loopback0
 description Internet gia lap
 ip address 1.1.1.1 255.255.255.255
 exit
!
ip route 113.161.100.0 255.255.255.248 113.161.1.2
ip route 203.113.0.0 255.255.0.0 100.64.0.1
ip route 8.8.8.8 255.255.255.255 100.64.0.1
end
write memory
```

### 2.4 Router biên: Dual-ISP, NAT, HSRP DMZ

- **Cân bằng tải:** hai edge cùng quảng bá default route vào OSPF. CORE-SW1 đi ra EDGE-R1, CORE-SW2 đi ra EDGE-R2, và mỗi DSW có **2 đường default ngang nhau** (qua hai Core), nên lưu lượng được chia đều ra hai ISP.
- **Failover:** cổng G0/0 của EDGE-R1 đứt → static default mất → R1 ngừng quảng bá → toàn bộ đi qua VNPT. HSRP DMZ chuyển `.1` sang R2 nhờ `track`.
- Nếu `standby 100 track ...` báo lỗi thì bỏ dòng đó; demo failover DMZ bằng cách tắt hẳn EDGE-R1.
- Route `10.50.10.0` (Cơ sở 2) trên R1 và `10.60.0.0` (VPN pool) trên R2 được `redistribute` để lưu lượng VPN luôn quay về đúng router có đường hầm.

**EDGE-R1** → `configs/EDGE-R1.txt`

```
hostname EDGE-R1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface GigabitEthernet0/0
 description WAN Viettel
 ip address 203.113.1.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
interface GigabitEthernet0/1
 description -> CORE-SW1
 ip address 10.255.3.1 255.255.255.252
 ip nat inside
 no shutdown
 exit
interface GigabitEthernet0/2
 description DMZ
 ip address 10.100.0.2 255.255.255.0
 ip nat inside
 standby version 2
 standby 100 ip 10.100.0.1
 standby 100 priority 110
 standby 100 preempt
 standby 100 track GigabitEthernet0/0 20
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.11 255.255.255.255
 exit
!
ip route 0.0.0.0 0.0.0.0 203.113.1.1
ip route 10.50.10.0 255.255.255.0 203.113.1.1
!
router ospf 1
 router-id 10.255.255.11
 passive-interface GigabitEthernet0/2
 network 10.255.3.0 0.0.0.3 area 0
 network 10.100.0.0 0.0.0.255 area 0
 network 10.255.255.11 0.0.0.0 area 0
 default-information originate
 redistribute static subnets
 exit
!
! --- PAT nội bộ, miễn NAT cho lưu lượng VPN ---
access-list 101 deny ip 10.0.0.0 0.255.255.255 10.50.10.0 0.0.0.255
access-list 101 deny ip 10.0.0.0 0.255.255.255 10.60.0.0 0.0.0.255
access-list 101 permit ip 10.0.0.0 0.255.255.255 any
ip nat inside source list 101 interface GigabitEthernet0/0 overload
!
! --- NAT tĩnh DMZ ---
ip nat inside source static 10.100.0.10 203.113.100.2
ip nat inside source static 10.100.0.11 203.113.100.3
ip nat inside source static 10.100.0.12 203.113.100.4
end
write memory
```

**EDGE-R2** → `configs/EDGE-R2.txt`

```
hostname EDGE-R2
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface GigabitEthernet0/0
 description WAN VNPT
 ip address 113.161.1.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
interface GigabitEthernet0/1
 description -> CORE-SW2
 ip address 10.255.3.5 255.255.255.252
 ip nat inside
 no shutdown
 exit
interface GigabitEthernet0/2
 description DMZ
 ip address 10.100.0.3 255.255.255.0
 ip nat inside
 standby version 2
 standby 100 ip 10.100.0.1
 standby 100 priority 100
 standby 100 preempt
 standby 100 track GigabitEthernet0/0 20
 no shutdown
 exit
interface Loopback0
 ip address 10.255.255.12 255.255.255.255
 exit
!
ip route 0.0.0.0 0.0.0.0 113.161.1.1
ip route 10.60.0.0 255.255.255.0 113.161.1.1
!
router ospf 1
 router-id 10.255.255.12
 passive-interface GigabitEthernet0/2
 network 10.255.3.4 0.0.0.3 area 0
 network 10.100.0.0 0.0.0.255 area 0
 network 10.255.255.12 0.0.0.0 area 0
 default-information originate
 redistribute static subnets
 exit
!
! --- PAT nội bộ, miễn NAT cho lưu lượng VPN ---
access-list 101 deny ip 10.0.0.0 0.255.255.255 10.50.10.0 0.0.0.255
access-list 101 deny ip 10.0.0.0 0.255.255.255 10.60.0.0 0.0.0.255
access-list 101 permit ip 10.0.0.0 0.255.255.255 any
ip nat inside source list 101 interface GigabitEthernet0/0 overload
!
! --- NAT tĩnh DMZ ---
ip nat inside source static 10.100.0.10 113.161.100.2
ip nat inside source static 10.100.0.11 113.161.100.3
ip nat inside source static 10.100.0.12 113.161.100.4
end
write memory
```

**Server DMZ** (làm trên file master): Desktop → IP Configuration → Static, gateway `10.100.0.1`, DNS `10.1.50.5`; Services → HTTP **On**, sửa `index.html` mỗi server một tiêu đề khác nhau.

| Server | IP | Mask |
|---|---|---|
| WEB | 10.100.0.10 | 255.255.255.0 |
| LMS | 10.100.0.11 | 255.255.255.0 |
| E-LIB | 10.100.0.12 | 255.255.255.0 |

### 2.5 Cơ sở 2: BR-R1, BR-SW và VPN Site-to-Site về EDGE-R1

**Bước 1 – bật gói bảo mật** trên **cả EDGE-R1 và BR-R1** (mỗi router một lần):

```
license boot module c2900 technology-package securityk9
! gõ yes khi được hỏi
end
write memory
reload
```

Sau khi reload, `show version` phải thấy `securityk9` ở trạng thái `Active`.

**Bước 2 – EDGE-R1** → `configs/EDGE-R1.vpn.txt`

```
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 5
 exit
crypto isakmp key <SITE_TO_SITE_PSK> address 203.113.2.2
crypto ipsec transform-set TS esp-aes 256 esp-sha-hmac
 exit
access-list 110 permit ip 10.0.0.0 0.255.255.255 10.50.10.0 0.0.0.255
crypto map CMAP 10 ipsec-isakmp
 set peer 203.113.2.2
 set transform-set TS
 match address 110
 exit
interface GigabitEthernet0/0
 crypto map CMAP
 exit
end
write memory
```

**Bước 3 – BR-R1** → `configs/BR-R1.txt`

```
hostname BR-R1
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface GigabitEthernet0/0
 description WAN Viettel
 ip address 203.113.2.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
interface GigabitEthernet0/1
 description LAN Co so 2
 ip address 10.50.10.1 255.255.255.0
 ip nat inside
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 203.113.2.1
!
! --- DHCP cho Cơ sở 2 ---
ip dhcp excluded-address 10.50.10.1 10.50.10.10
ip dhcp pool COSO2
 network 10.50.10.0 255.255.255.0
 default-router 10.50.10.1
 dns-server 10.1.50.5
 exit
!
! --- PAT, miễn NAT cho lưu lượng về campus ---
access-list 101 deny ip 10.50.10.0 0.0.0.255 10.0.0.0 0.255.255.255
access-list 101 permit ip 10.50.10.0 0.0.0.255 any
ip nat inside source list 101 interface GigabitEthernet0/0 overload
!
! --- IPsec về EDGE-R1 ---
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 5
 exit
crypto isakmp key <SITE_TO_SITE_PSK> address 203.113.1.2
crypto ipsec transform-set TS esp-aes 256 esp-sha-hmac
 exit
access-list 110 permit ip 10.50.10.0 0.0.0.255 10.0.0.0 0.255.255.255
crypto map CMAP 10 ipsec-isakmp
 set peer 203.113.1.2
 set transform-set TS
 match address 110
 exit
interface GigabitEthernet0/0
 crypto map CMAP
 exit
end
write memory
```

**Bước 4 – BR-SW** → `configs/BR-SW.txt`

```
hostname BR-SW
no ip domain-lookup
enable secret <ENABLE_SECRET>
service password-encryption
banner motd #Truy cap trai phep bi cam#
line console 0
 logging synchronous
 exec-timeout 0 0
 exit
interface Vlan1
 ip address 10.50.10.2 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.50.10.1
end
write memory
```

**Bước 5 – PC-BR1, PC-BR2:** IP Configuration → **DHCP** → phải nhận `10.50.10.x`.

> Tunnel chỉ lên khi có lưu lượng: ping từ PC-BR1 tới `10.1.50.5` vài lần (lần đầu có thể timeout).

### 2.6 Remote Access VPN (Easy VPN) trên EDGE-R2 cho PC-Remote

**Bước 1** – chạy lệnh `license boot ...` như 2.5 trên **EDGE-R2**, reload.

**Bước 2 – EDGE-R2** → `configs/EDGE-R2.vpn.txt`

```
aaa new-model
aaa authentication login default local
aaa authentication login VPN-AUTH local
aaa authorization network VPN-GRP local
username vpnuser secret <VPN_USER_PASSWORD>
!
crypto isakmp policy 20
 encr aes 256
 hash sha
 authentication pre-share
 group 2
 exit
ip local pool VPN-POOL 10.60.0.10 10.60.0.110
crypto isakmp client configuration group REMOTE
 key <REMOTE_GROUP_KEY>
 pool VPN-POOL
 exit
crypto ipsec transform-set TS2 esp-aes 256 esp-sha-hmac
 exit
crypto dynamic-map DMAP 10
 set transform-set TS2
 reverse-route
 exit
crypto map RMAP client authentication list VPN-AUTH
crypto map RMAP isakmp authorization list VPN-GRP
crypto map RMAP client configuration address respond
crypto map RMAP 10 ipsec-isakmp dynamic DMAP
interface GigabitEthernet0/0
 crypto map RMAP
 exit
end
write memory
```

> Sau `aaa new-model`, mở lại console của EDGE-R2 phải đăng nhập bằng `vpnuser` / `<VPN_USER_PASSWORD>`.

**Bước 3 – PC-Remote:** IP Configuration → Static: IP `113.161.2.2`, mask `255.255.255.252`, gateway `113.161.2.1`, DNS `10.1.50.5`. Desktop → **VPN**: GroupName `REMOTE`, Group Key `<REMOTE_GROUP_KEY>`, Host IP `113.161.1.2`, Username `vpnuser`, Password `<VPN_USER_PASSWORD>` → Connect → phải nhận IP `10.60.0.x`.

### 2.7 ACL ở biên (làm sau cùng, khi mọi thứ khác đã chạy)

Áp **inbound trên G0/0**. Làm xong test lại ngay VPN và web DMZ; hỏng thì `no ip access-group OUTSIDE-IN in` để gỡ.

**EDGE-R1** → `configs/EDGE-R1.acl.txt`

```
ip access-list extended OUTSIDE-IN
 permit tcp any host 203.113.100.2 eq www
 permit tcp any host 203.113.100.3 eq www
 permit tcp any host 203.113.100.4 eq www
 permit udp host 203.113.2.2 any eq isakmp
 permit esp host 203.113.2.2 any
 permit tcp any any established
 permit icmp any any echo-reply
 deny ip any any
 exit
interface GigabitEthernet0/0
 ip access-group OUTSIDE-IN in
 exit
end
write memory
```

**EDGE-R2** → `configs/EDGE-R2.acl.txt`

```
ip access-list extended OUTSIDE-IN
 permit tcp any host 113.161.100.2 eq www
 permit tcp any host 113.161.100.3 eq www
 permit tcp any host 113.161.100.4 eq www
 permit udp any any eq isakmp
 permit udp any any eq 4500
 permit esp any any
 permit tcp any any established
 permit icmp any any echo-reply
 deny ip any any
 exit
interface GigabitEthernet0/0
 ip access-group OUTSIDE-IN in
 exit
end
write memory
```

### 2.8 Tự kiểm tra

Trước khi gửi file text hoặc báo xong phần giao diện, Khang tự chạy các bước ở **mục 6.2** tương ứng với ngày đó.


---

## 3. Lịch làm việc và điểm ghép

Khang giữ file master. Cuối mỗi ngày: Khánh gửi file text, Khang dán vào master; phần giao diện thì người phụ trách tự mở master làm theo lượt.

| Ngày | Khánh | Khang | Test chung cuối ngày |
|---|---|---|---|
| **2** | 1.1, 1.2, 1.3 (file text) | 2.1, 2.2, 2.3 (file text) | Khang dán hết vào master. Đặt IP tĩnh tạm cho PC-BGH và PC-Lab1, ping chéo khối |
| **3** | 1.4 DHCP, 1.5 RADIUS/Portal, 1.7 PC nhận DHCP (trên master) | 2.4 Edge + server DMZ (trên master) | PC nhận DHCP, ping `8.8.8.8`, PC-Remote mở web DMZ |
| **4** | 1.6 Wi-Fi | 2.5, 2.6 VPN; 2.7 nếu kịp | Smartphone vào `UIT-SV`; PC-BR1 và PC-Remote vào được `10.1.50.7` |
| **5** | Chạy hết bảng 1.8, chụp ảnh | Chạy hết bảng 2.8, chụp ảnh | Cùng chạy mục 4, sửa lỗi, chốt file |

---

## 4. Kịch bản kiểm tra tổng (ngày 5, cả hai cùng làm)

1. PC-Lab1 nhận DHCP → mở `http://lms.campus.edu.vn`.
2. SV-Phone1 kết nối `UIT-SV` (RADIUS) → mở `http://www.campus.edu.vn`; ping PC-BGH bị chặn.
3. Tắt DSW-B1 → PC-GV-B vẫn ping được `10.1.50.5`.
4. Tắt G0/0 của EDGE-R1 → PC-BGH vẫn ping được `8.8.8.8`.
5. PC-BR1 (Cơ sở 2) và PC-Remote (VPN) mở `http://10.1.50.7`.

---

## 5. Lỗi Packet Tracer hay gặp

| Triệu chứng | Nguyên nhân / cách sửa |
|---|---|
| PC không nhận DHCP | Thiếu `ip helper-address` trên SVI, thiếu pool trên server, hoặc thiếu `no ip dhcp snooping information option` trên ASW |
| HSRP cả hai cùng Active | Port-channel giữa hai DSW chưa mang VLAN đó, hoặc ACL chặn `224.0.0.102` |
| OSPF không lên | Cổng chưa `no switchport` hoặc còn bị `passive-interface` |
| Không có lệnh `crypto` | Chưa chạy `license boot ... securityk9` và `reload` |
| AP không join WLC | Pool VLAN 70 thiếu WLC Address, hoặc chưa route được tới `10.1.50.10` |
| Port-channel `(SD)` | Hai đầu khác chế độ (trunk vs routed) hoặc khác allowed VLAN |
| Cổng PC bị err-disabled | Port Security bắt MAC lạ: `shutdown` rồi `no shutdown` trên cổng đó |
| Dán lệnh bị mất dòng | Dán khối nhỏ hơn, đợi CLI chạy xong mới dán tiếp |

---

## 6. Hướng dẫn kiểm tra công việc

Dùng cho người kiểm tra, hoặc để Khánh và Khang kiểm tra chéo phần của nhau. Kiểm tra trên **file master sau khi ghép**, không kiểm tra trên bản copy riêng.

**Cách làm:**
1. Chạy từng bước theo đúng thứ tự: bước sau dựa vào bước trước. Bước nào sai thì dừng, báo người phụ trách, chưa kiểm tra tiếp phần phụ thuộc.
2. Mỗi bước có cột **Kết quả đúng**. Khớp thì ghi ✅ vào bảng nghiệm thu ở mục 6.3, không khớp thì ghi ❌ kèm ảnh chụp.
3. Ảnh chụp đặt tên `<người>_<thiết bị>_<lệnh>.png`, ví dụ `Khanh_DSW-A1_standby.png`. Ảnh này dùng lại cho báo cáo.
4. Bước có ghi **(ảnh)** là bắt buộc chụp để đưa vào báo cáo.

### 6.1 Kiểm tra phần của Khánh

#### A. Distribution – sau ngày 2

Chạy cho **cả 8 DSW**; bảng dưới lấy DSW-A1 làm ví dụ.

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | DSW-A1 | `show vlan brief` | Có VLAN 10, 20, 30, 50, 60, 99, 999 trạng thái `active`, đúng tên | File `.L2` chưa dán hết khối `vlan` |
| 2 | DSW-A1 | `show etherchannel summary` **(ảnh)** | Dòng `1  Po1(SU)  LACP  Gig1/0/3(P) Gig1/0/4(P)` | `(SD)`: DSW-A2 chưa cấu hình hoặc khác allowed VLAN. Cổng có cờ `(I)`/`(s)`: một đầu chưa `channel-group` |
| 3 | DSW-A1 | `show interfaces trunk` | `Po1`, `Gig1/0/5`, `Gig1/0/6`, `Gig1/0/7` ở trạng thái `trunking`, native `999`; allowed đúng tab *Cổng & VLAN* | Cổng thiếu: chưa `no shutdown` hoặc ASW đầu kia chưa cấu hình |
| 4 | DSW-A1 | `show ip interface brief` | Mọi `Vlan..` là `up up` | `down down`: chưa có cổng nào mang VLAN đó lên (kiểm tra trunk/ASW) |
| 5 | DSW-A1, DSW-A2 | `show standby brief` **(ảnh)** | DSW-A1: mọi dòng `Active`, Pri `110`, Standby là IP `.3`. DSW-A2: mọi dòng `Standby`, Pri `100` | Cả hai `Active`: Po1 không mang VLAN đó, hoặc ACL chặn hello HSRP |
| 6 | DSW-A1 | `show spanning-tree vlan 10` | Có dòng `This bridge is the root` | Thiếu `spanning-tree vlan ... root primary` |
| 7 | DSW-A1 | `show access-lists` | Có `SV-WIFI-IN` (7 dòng) và `CAM-IN` (4 dòng) | File `.L2` chưa dán khối ACL |

Mẫu output đúng của `show standby brief` trên DSW-A1:

```
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        10   110 P Active   local           10.1.10.3       10.1.10.1
Vl20        20   110 P Active   local           10.1.20.3       10.1.20.1
...
```

#### B. Access – sau ngày 2

Chạy cho **cả 8 ASW và SRV-SW**; bảng dưới lấy ASW-B2 làm ví dụ.

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | ASW-B2 | `show vlan brief` **(ảnh)** | `Fa0/1` ở VLAN 70, `Fa0/2` ở VLAN 60, `Fa0/3` ở VLAN 80, `Fa0/4–24` ở VLAN 999 | Đối chiếu tab *Cổng & VLAN* |
| 2 | ASW-B2 | `show interfaces trunk` | `Gig0/1`, `Gig0/2` trunking, native 999, allowed `20,60,70,80,99` | Một đầu native khác 999 → log báo `native VLAN mismatch` |
| 3 | ASW-B2 | `show interfaces status` | Cổng không dùng hiện `disabled` | Thiếu khối "Cổng không dùng" |
| 4 | ASW-B2 | `show spanning-tree vlan 70` | `Gi0/1` là `Root FWD`, `Gi0/2` là `Altn BLK` (đúng thiết kế: chặn 1 đường lên DSW-B2) | Cả hai `FWD` hoặc root nằm ở ASW: DSW-B1 thiếu `root primary` |
| 5 | ASW-B2 | `show port-security` | 3 cổng Fa0/1–3 có `MaxSecureAddr 2`, `Action Restrict` | Thiếu khối port-security ở cổng đó |
| 6 | ASW-B2 | `show ip dhcp snooping` | Snooping bật cho VLAN 60, 70, 80; `Gi0/1`, `Gi0/2` là `yes` (trusted) | Uplink chưa trust → PC không nhận DHCP |
| 7 | ASW-B2 | `show running-config \| include information` | Có dòng `no ip dhcp snooping information option` | Thiếu dòng này → DHCP qua relay hỏng |
| 8 | ASW-B2 | `show ip interface brief \| include Vlan` | `Vlan1` là `administratively down`, `Vlan99` là `up up` với IP `10.2.99.12` | — |

#### C. DHCP và thiết bị đầu cuối – sau ngày 3

Điều kiện: phần 6.2-A của Khang đã đạt (định tuyến chạy). Mở Desktop → IP Configuration của từng máy, bấm DHCP rồi đối chiếu:

| Thiết bị | IP phải thuộc dải | Gateway | DNS |
|---|---|---|---|
| PC-BGH, PC-HanhChinh | 10.1.10.11 – .254 | 10.1.10.1 | 10.1.50.5 |
| PC-GV-A | 10.1.20.11 – .254 | 10.1.20.1 | 10.1.50.5 |
| PC-GV-B | 10.2.20.11 – .254 | 10.2.20.1 | 10.1.50.5 |
| PC-ThuVien | 10.2.80.11 – .254 | 10.2.80.1 | 10.1.50.5 |
| PC-Lab1 … PC-Lab4 | 10.3.40.11 – 10.3.41.254 (mask /23) | 10.3.40.1 | 10.1.50.5 |
| CAM-A / CAM-B / CAM-K1, K2 | 10.1.60.x / 10.2.60.x / 10.4.60.x | .1 cùng dải | 10.1.50.5 |
| LAP-B1, B2 / LAP-C1 / LAP-K1, K2 (rê chuột lên AP) | 10.2.70.x / 10.3.70.x / 10.4.70.x | .1 cùng dải | 10.1.50.5 |

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | Một PC bất kỳ **(ảnh)** | IP Configuration → DHCP | Hiện `DHCP request successful` và IP đúng bảng trên | `APIPA 169.254.x.x`: xem mục 5, dòng "PC không nhận DHCP" |
| 2 | ASW-C1 | `show ip dhcp snooping binding` | Có 2 dòng MAC/IP của PC-Lab1, PC-Lab2 ở VLAN 40 | Trống: PC chưa xin DHCP sau khi bật snooping |
| 3 | PC-Lab1 | Command Prompt: `ping lms.campus.edu.vn` | Phân giải ra `10.100.0.11` và có reply | Không phân giải: thiếu bản ghi DNS; phân giải được nhưng không reply: chờ 6.2-C |

#### D. Wi-Fi – sau ngày 4

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | PC-BGH | Web Browser `https://10.1.50.10` → **Monitor** **(ảnh)** | Access Points: 5, Clients: 3 | AP thiếu: AP chưa có IP VLAN 70 hoặc pool AP thiếu WLC Address |
| 2 | WLC1 (web) | **Wireless → All APs** | Đủ LAP-B1, B2, C1, K1, K2 | — |
| 3 | SV-Phone1 **(ảnh)** | Desktop → IP Configuration | IP `10.20.x.x`, mask `255.255.224.0`, gateway `10.20.0.1` | Không liên kết: sai SSID/mật khẩu hoặc RADIUS sai secret; liên kết được nhưng không có IP: interface `vlan30` trên WLC sai DHCP server |
| 4 | SV-Phone1 | Đổi password sai → kết nối lại | **Không** kết nối được (chứng minh RADIUS đang xác thực) | Vẫn vào được: WLAN đang để Open/PSK, chưa phải 802.1X |
| 5 | SV-Phone1 | `ping 10.1.50.5` | Có reply | — |

#### E. ACL và dự phòng – sau ngày 4

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | SV-Phone1 **(ảnh)** | `ping 10.1.10.11` (dải BGH-HC) | `Destination host unreachable` hoặc timeout (bị chặn) | Có reply: ACL chưa áp vào `Vlan30` ở **cả hai** DSW-A |
| 2 | CAM-B | `ping` IP của PC-GV-B | Bị chặn | Có reply: ACL `CAM-IN` chưa áp trên DSW-B1/B2 |
| 3 | CAM-B | `ping 10.1.50.5` | Có reply | Bị chặn: sai dải nguồn trong `CAM-IN` |
| 4 | DSW-B1, DSW-B2 | `show standby brief` | Vẫn một Active, một Standby ở VLAN 60 sau khi có ACL | Cả hai Active: thiếu dòng `permit udp any host 224.0.0.102 eq 1985` |
| 5 | DSW-A1 | `show access-lists` | Các dòng `deny` của `SV-WIFI-IN` có `(x match(es))` tăng | Không tăng: ACL áp sai interface hoặc sai chiều |
| 6 | PC-GV-B **(ảnh)** | `ping -t 10.1.50.5`, rồi tắt nguồn DSW-B1 (tab Physical) | Mất 1–3 gói rồi chạy tiếp; DSW-B2 `show standby brief` thành `Active` | Mất hẳn: HSRP hoặc OSPF trên DSW-B2 chưa chạy |
| 7 | DSW-B1 | Bật nguồn lại, đợi ~1 phút, `show standby brief` | DSW-B1 lấy lại `Active` (nhờ `preempt`) | — |

### 6.2 Kiểm tra phần của Khang

#### A. Core và OSPF – sau ngày 2

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | CORE-SW1 | `show ip interface brief` | G1/0/1–8, G1/0/21, Port-channel1, Loopback0 đều `up up` với đúng IP tab *IP thiết bị* | `down`: thiếu `no shutdown` hoặc đầu DSW chưa `no switchport` |
| 2 | CORE-SW1 | `show etherchannel summary` | `1  Po1(RU)  LACP  Gig1/0/23(P) Gig1/0/24(P)` | `(SD)`: CORE-SW2 chưa cấu hình Po1 |
| 3 | CORE-SW1 **(ảnh)** | `show ip ospf neighbor` | **10 neighbor** trạng thái `FULL`: 8 DSW (`10.255.255.21–28`), CORE-SW2 (`.2`), EDGE-R1 (`.11`, có sau ngày 3) | Thiếu DSW nào thì kiểm tra file `.routing` của DSW đó |
| 4 | CORE-SW2 | `show ip ospf neighbor` | Tương tự: 8 DSW, CORE-SW1, EDGE-R2 | — |
| 5 | DSW-C1 **(ảnh)** | `show ip route ospf` | Có `10.1.10.0/24`, `10.2.20.0/24`, `10.4.60.0/24`, `10.20.0.0/19`… mỗi mạng **2 đường** (qua CORE-SW1 và CORE-SW2) | Chỉ 1 đường: một uplink chưa có neighbor |
| 6 | DSW-A1 | `ping 10.255.255.28 source loopback0` | Thành công (A1 tới K2 qua core) | — |
| 7 | PC-BGH | IP tĩnh tạm `10.1.10.50/24`, gw `10.1.10.1`, ping PC-Lab1 (IP tĩnh tạm `10.3.40.50/23`) | Có reply → định tuyến giữa các khối chạy | Xong bước này trả hai PC về DHCP |

Mẫu output đúng của `show ip ospf neighbor` trên CORE-SW1 (rút gọn):

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.255.21     1   FULL/DR         00:00:35    10.255.1.2      GigabitEthernet1/0/1
10.255.255.22     1   FULL/DR         00:00:36    10.255.1.6      GigabitEthernet1/0/2
...
10.255.255.2      1   FULL/BDR        00:00:33    10.255.0.2      Port-channel1
```

#### B. ISP giả lập – sau ngày 2

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | ISP-Viettel | `ping 1.1.1.1 source loopback0` | Thành công | Thiếu route chéo giữa hai ISP |
| 2 | ISP-VNPT | `show ip route static` | Có `113.161.100.0/29`, `203.113.0.0/16`, `8.8.8.8/32` | — |

#### C. Router biên, NAT, DMZ – sau ngày 3

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | DSW-A1 **(ảnh)** | `show ip route \| include 0.0.0.0` | **Hai** dòng `O*E2 0.0.0.0/0`: qua `10.255.1.1` (CORE-SW1 → EDGE-R1) và qua `10.255.2.1` (CORE-SW2 → EDGE-R2) | Một dòng: edge còn lại thiếu `default-information originate` hoặc static default |
| 2 | CORE-SW1 | `show ip route \| include 0.0.0.0` | Một dòng `O*E2` qua `10.255.3.1` (EDGE-R1). Đúng thiết kế: Core chọn edge gần nhất, chia tải nằm ở DSW | — |
| 3 | PC-BGH **(ảnh)** | `ping 8.8.8.8` và `ping 1.1.1.1` | Cả hai có reply | — |
| 4 | EDGE-R1 **(ảnh)** | `show ip nat translations` | Có dòng `icmp 203.113.1.2:x  10.1.10.x:x` (PAT) và 3 dòng NAT tĩnh `203.113.100.2–4 → 10.100.0.10–12` | Trống: cổng thiếu `ip nat inside/outside` hoặc ACL 101 sai |
| 5 | EDGE-R1, EDGE-R2 | `show standby brief` | R1 `Active` group 100, R2 `Standby`, VIP `10.100.0.1` | — |
| 6 | PC-Remote **(ảnh)** | Web Browser `http://203.113.100.2`, `.3`, `.4` | Mở được 3 trang WEB, LMS, E-LIB | Không mở: thiếu route `203.113.100.0/29` trên ISP-Viettel, hoặc HTTP trên server chưa On |
| 7 | PC-BGH | `tracert 8.8.8.8` | Đi qua `10.1.10.2` → `10.255.x.1` → `10.255.3.x` → ISP | — |
| 8 | EDGE-R1 **(ảnh)** | `shutdown` G0/0, đợi ~40 giây, trên PC-BGH `tracert 8.8.8.8` | Vẫn tới được 8.8.8.8, đường đi qua `10.255.3.5` (EDGE-R2). EDGE-R2 thành HSRP `Active` | Không ra được: R2 chưa quảng bá default. **Nhớ `no shutdown` lại sau khi test** |

#### D. VPN Site-to-Site – sau ngày 4

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | EDGE-R1, BR-R1 | `show version` | Bảng License: `securityk9` là `Permanent` / `Active` | Chưa reload sau lệnh `license boot` |
| 2 | PC-BR1 | IP Configuration → DHCP | IP `10.50.10.11` trở lên, gateway `10.50.10.1`, DNS `10.1.50.5` | — |
| 3 | PC-BR1 **(ảnh)** | `ping 10.1.50.5` (4–5 lần) | Lần đầu có thể timeout, các lần sau có reply | Toàn timeout: xem `show crypto isakmp sa` |
| 4 | EDGE-R1 **(ảnh)** | `show crypto isakmp sa` | Dòng `203.113.1.2  203.113.2.2  QM_IDLE ... ACTIVE` | Trống: sai key hoặc sai peer; `MM_NO_STATE`: policy hai đầu khác nhau |
| 5 | EDGE-R1 **(ảnh)** | `show crypto ipsec sa` | `#pkts encaps` và `#pkts decaps` khác 0 và tăng sau mỗi lần ping | Bằng 0: ACL 110 hai đầu không đối xứng |
| 6 | PC-BR1 | `ping 8.8.8.8` | Vẫn có reply (Cơ sở 2 ra Internet qua NAT, không qua VPN) | — |

#### E. Remote Access VPN – sau ngày 4

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | PC-Remote **(ảnh)** | Desktop → VPN → Connect | Báo `VPN is connected` | Lỗi xác thực: sai Group/Key/user; không phản hồi: EDGE-R2 chưa license |
| 2 | PC-Remote | Command Prompt: `ipconfig` | Có thêm IP `10.60.0.x` (tunnel) | — |
| 3 | PC-Remote **(ảnh)** | `ping 10.1.50.7` và Web Browser `http://10.1.50.7` | Có reply và mở được trang Portal nội bộ | Không reply: thiếu `redistribute static subnets` trên EDGE-R2, hoặc ACL 101 chưa miễn NAT `10.60.0.0` |
| 4 | EDGE-R2 | `show crypto isakmp sa` | Có SA tới `113.161.2.2` | — |

#### F. ACL biên – sau cùng

| # | Chạy trên | Lệnh / thao tác | Kết quả đúng | Sai thì xem |
|---|---|---|---|---|
| 1 | EDGE-R1 | `show access-lists OUTSIDE-IN` | Các dòng `permit` có `match` tăng sau khi chạy lại các bước C, D | — |
| 2 | Chạy lại | 6.2-C bước PC-Remote mở web, 6.2-D bước ping, 6.2-E bước VPN | Vẫn đạt hết | Hỏng mục nào thì gỡ ACL bằng `no ip access-group OUTSIDE-IN in`, báo lại |
| 3 | PC-Remote **(ảnh)** | `ping 203.113.1.2` | **Bị chặn** (Internet không ping được router biên) | Có reply: ACL chưa áp vào G0/0 |

### 6.3 Bảng nghiệm thu

Copy bảng này vào nhóm chat, điền sau mỗi lần kiểm tra.

| Mốc | Phần | Người làm | Kết quả | Ghi chú / ảnh lỗi |
|---|---|---|---|---|
| Ngày 2 | 6.1-A Distribution (8 DSW) | Khánh | ⬜ | |
| Ngày 2 | 6.1-B Access (8 ASW + SRV-SW) | Khánh | ⬜ | |
| Ngày 2 | 6.2-A Core và OSPF | Khang | ⬜ | |
| Ngày 2 | 6.2-B ISP giả lập | Khang | ⬜ | |
| Ngày 3 | 6.1-C DHCP và thiết bị đầu cuối | Khánh | ⬜ | |
| Ngày 3 | 6.2-C Biên, NAT, DMZ, failover ISP | Khang | ⬜ | |
| Ngày 4 | 6.1-D Wi-Fi | Khánh | ⬜ | |
| Ngày 4 | 6.1-E ACL và HSRP failover | Khánh | ⬜ | |
| Ngày 4 | 6.2-D VPN Site-to-Site | Khang | ⬜ | |
| Ngày 4 | 6.2-E Remote Access VPN | Khang | ⬜ | |
| Ngày 5 | 6.2-F ACL biên | Khang | ⬜ | |
| Ngày 5 | Mục 4 – kịch bản tổng | Cả hai | ⬜ | |

**Khi một mục ❌:** gửi người phụ trách ảnh chụp + nguyên văn thông báo lỗi (nếu có). Người đó sửa trong file text, gửi lại, dán đè lên master rồi chạy lại đúng bước đó.
