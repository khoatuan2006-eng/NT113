# 📑 BIÊN BẢN BÀN GIAO & KẾ HOẠCH TRIỂN KHAI TIẾP THEO (DÀNH CHO KHANG)

> **Dự án:** NT113 – Đề tài 1: Thiết kế và triển khai mạng Campus  
> **Người bàn giao:** Khánh (Phụ trách L2, Campus nội bộ, DHCP, Wi-Fi)  
> **Người nhận bàn giao:** Khang (Phụ trách L3 Core, OSPF, Biên, Dual-ISP, NAT, DMZ, VPN, Giữ file Master)  
> **Nguồn số liệu duy nhất:** Sheet *"Quy hoạch VLAN & IP – Mạng Campus (Đề tài 1)"*  

---

## I. TỔNG KẾT PHẦN VIỆC KHÁNH ĐÃ BÀN GIAO

Khánh đã hoàn tất **17 file cấu hình CLI** trong thư mục `configs/`:
- **8 DSW (Phần L2 + SVI):** `DSW-A1.L2.txt`, `DSW-A2.L2.txt`, `DSW-B1.L2.txt`, `DSW-B2.L2.txt`, `DSW-C1.L2.txt`, `DSW-C2.L2.txt`, `DSW-K1.L2.txt`, `DSW-K2.L2.txt`
  - Đã xong: VTP Transparent, VLAN, STP Rapid-PVST (DSW-1 = root primary, DSW-2 = root secondary).
  - EtherChannel L2 `Port-channel1` giữa cặp DSW cùng khối (LACP Active, Gi1/0/3-4).
  - Trunk xuống các ASW và SRV-SW (Native VLAN 999).
  - SVI + HSRP v2 (DSW-1 Active pri 110, DSW-2 Standby pri 100, preempt).
  - `ip helper-address 10.1.50.5` trên các VLAN cấp DHCP.
  - ACL `CAM-IN` (đã mở UDP 1985 cho HSRP) và `SV-WIFI-IN`.
- **8 ASW:** `ASW-A1.txt` đến `ASW-K2.txt`
  - Cổng Access phân bổ chuẩn theo Sheet quy hoạch.
  - Port Security (max 2, sticky, restrict), BPDU Guard, PortFast.
  - DHCP Snooping kích hoạt đúng VLAN, uplink trusted, **đã có lệnh bắt buộc `no ip dhcp snooping information option`**.
  - Các cổng không dùng đưa vào VLAN 999 và `shutdown`.
- **1 Server Switch (SRV-SW):** `SRV-SW.txt`
  - Access VLAN 50 cho DHCP-DNS, RADIUS, PORTAL-NB.
  - Trunk sang WLC1 trên Fa0/4 với **Native VLAN 50** (untagged management).

---

## II. DANH MỤC CÁC PHẦN VIỆC KHANG CẦN HOÀN THIỆN TIẾP THEO

Khang giữ vai trò **nắm giữ file master**, điều phối và ghép nối hệ thống theo 5 giai đoạn sau:

```
┌────────────────────────────────────────────────────────────────────────┐
│ Giai đoạn 1: Ghép Core Switches & Phần Định tuyến của 8 DSW (OSPF)    │
├────────────────────────────────────────────────────────────────────────┤
│ Giai đoạn 2: Cấu hình Dual-ISP, Router biên (EDGE-R1/R2) & Server DMZ │
├────────────────────────────────────────────────────────────────────────┤
│ Giai đoạn 3: Triển khai VPN Site-to-Site (Cơ sở 2) & Remote Access VPN │
├────────────────────────────────────────────────────────────────────────┤
│ Giai đoạn 4: Áp dụng ACL ở biên (Security) & Bàn giao lượt cho Khánh  │
├────────────────────────────────────────────────────────────────────────┤
│ Giai đoạn 5: Đồng kiểm tra & Chạy kịch bản nghiệm thu tổng thể        │
└────────────────────────────────────────────────────────────────────────┘
```

---

### GIAI ĐOẠN 1: CORE SWITCHES & ĐỊNH TUYẾN 8 DSW (Mục 2.1 & 2.2)

#### 1. Cấu hình 2 Core Switch (`CORE-SW1` và `CORE-SW2`)
*Tham khảo cấu hình chi tiết tại Mục 2.1 README.md*
- **L3 EtherChannel:** Cổng Gi1/0/23-24 gom thành `Port-channel1`, chạy `no switchport`, IP `10.255.0.1/30` (CORE-SW1) và `10.255.0.2/30` (CORE-SW2).
- **Cổng đấu nối xuống 8 DSW (Gi1/0/1 - Gi1/0/8):** Chuyển sang routed port (`no switchport`), đặt IP `/30` tương ứng theo bảng:
  - Gi1/0/1 → DSW-A1 (`10.255.1.0/30` trên CORE-1, `10.255.2.0/30` trên CORE-2)
  - Gi1/0/2 → DSW-A2 (`10.255.1.4/30` trên CORE-1, `10.255.2.4/30` trên CORE-2)
  - Gi1/0/3 → DSW-B1 (`10.255.1.8/30` trên CORE-1, `10.255.2.8/30` trên CORE-2)
  - Gi1/0/4 → DSW-B2 (`10.255.1.12/30` trên CORE-1, `10.255.2.12/30` trên CORE-2)
  - Gi1/0/5 → DSW-C1 (`10.255.1.16/30` trên CORE-1, `10.255.2.16/30` trên CORE-2)
  - Gi1/0/6 → DSW-C2 (`10.255.1.20/30` trên CORE-1, `10.255.2.20/30` trên CORE-2)
  - Gi1/0/7 → DSW-K1 (`10.255.1.24/30` trên CORE-1, `10.255.2.24/30` trên CORE-2)
  - Gi1/0/8 → DSW-K2 (`10.255.1.28/30` trên CORE-1, `10.255.2.28/30` trên CORE-2)
- **Cổng uplink lên Edge Router:**
  - CORE-SW1 Gi1/0/21 nối EDGE-R1 Gi0/1 (`10.255.3.2/30`).
  - CORE-SW2 Gi1/0/21 nối EDGE-R2 Gi0/1 (`10.255.3.6/30`).
- **OSPF Process:** `router ospf 1`, router-id `10.255.255.1` (Core1) và `.2` (Core2), network `10.255.0.0 0.0.255.255 area 0`.

#### 2. Dán phần định tuyến vào 8 DSW (Các file `.routing.txt`)
*Dán vào sau cấu hình L2 của Khánh trên cùng thiết bị:*
- `DSW-A1.routing.txt`, `DSW-A2.routing.txt`
- `DSW-B1.routing.txt`, `DSW-B2.routing.txt`
- `DSW-C1.routing.txt`, `DSW-C2.routing.txt`
- `DSW-K1.routing.txt`, `DSW-K2.routing.txt`
- **Nguyên lý bắt buộc:**
  - Gi1/0/1 và Gi1/0/2 chuyển sang routed port (`no switchport`) đặt IP `/30`.
  - Loopback0 đặt IP `/32`.
  - `passive-interface default` và `no passive-interface Gi1/0/1-2`.
  - Quảng bá dải SVI của khối mình vào OSPF Area 0.

> **Kiểm tra tiêu chí đạt Giai đoạn 1:**
> - `CORE-SW1# show ip ospf neighbor` -> Phải thấy **10 neighbor trạng thái FULL**.
> - Ping từ `DSW-A1` tới Loopback0 của `DSW-K2` thành công qua mạng Core.

---

### GIAI ĐOẠN 2: DUAL-ISP, EDGE ROUTERS & SERVER DMZ (Mục 2.3 & 2.4)

#### 1. Cấu hình 2 ISP giả lập
- **ISP-Viettel:** Gi0/0 nối EDGE-R1 (`203.113.1.1/30`), Gi0/1 nối ISP-VNPT (`100.64.0.1/30`), Gi0/2 nối BR-R1 Cơ sở 2 (`203.113.2.1/30`), Loopback0: `8.8.8.8/32`.
- **ISP-VNPT:** Gi0/0 nối EDGE-R2 (`113.161.1.1/30`), Gi0/1 nối ISP-Viettel (`100.64.0.2/30`), Gi0/2 nối PC-Remote (`113.161.2.1/30`), Loopback0: `1.1.1.1/32`.
- Thiết lập định tuyến static qua lại giữa 2 ISP.

#### 2. Cấu hình 2 Router biên (`EDGE-R1` & `EDGE-R2`)
- **Định tuyến OSPF & Default Route:**
  - `ip route 0.0.0.0 0.0.0.0 <ISP_IP>`
  - `router ospf 1`: `default-information originate` (đẩy default-route vào campus để chia tải giữa 2 ISP) và `redistribute static subnets`.
- **HSRP DMZ (Group 100):**
  - VIP: `10.100.0.1` trên Gi0/2.
  - EDGE-R1: Priority 110, Preempt, `standby 100 track GigabitEthernet0/0 20` (Active).
  - EDGE-R2: Priority 100, Preempt (Standby).
- **NAT / PAT:**
  - ACL 101: Cho phép mạng `10.0.0.0/8` ra ngoài Internet, **nhưng deny dải Cơ sở 2 (`10.50.10.0/24`) và VPN Remote (`10.60.0.0/24`) để không bị NAT mất gói VPN**.
  - NAT Overload trên Gi0/0.
  - Static NAT cho 3 Server DMZ:
    - EDGE-R1: Map `10.100.0.10-12` sang `203.113.100.2-4`
    - EDGE-R2: Map `10.100.0.10-12` sang `113.161.100.2-4`

#### 3. Cấu hình giao diện 3 Server vùng DMZ (Làm trên file Master)
Truy cập tab **Desktop** -> **IP Configuration** (Static) và tab **Services** -> **HTTP**:
- **WEB:** IP `10.100.0.10`, Mask `255.255.255.0`, Gateway `10.100.0.1`, DNS `10.1.50.5`. HTTP On, sửa trang chủ: *"Trang Chu Web Truong"*.
- **LMS:** IP `10.100.0.11`, Mask `255.255.255.0`, Gateway `10.100.0.1`, DNS `10.1.50.5`. HTTP On, sửa trang chủ: *"He Thong Hoc Tap Truc Tuyen LMS"*.
- **E-LIB:** IP `10.100.0.12`, Mask `255.255.255.0`, Gateway `10.100.0.1`, DNS `10.1.50.5`. HTTP On, sửa trang chủ: *"Thu Vien Dien Tu E-Library"*.

---

### GIAI ĐOẠN 3: TRIỂN KHAI VPN (SITE-TO-SITE & REMOTE ACCESS) (Mục 2.5 & 2.6)

#### 1. VPN Site-to-Site giữa Campus (EDGE-R1) và Cơ sở 2 (BR-R1)
> [!IMPORTANT]
> Bắt buộc chạy lệnh bật bản quyền crypto trên cả `EDGE-R1` và `BR-R1`, sau đó **reload router**:
> ```text
> license boot module c2900 technology-package securityk9
> write memory
> reload
> ```

- **Trên EDGE-R1 (`configs/EDGE-R1.vpn.txt`):**
  - ISAKMP Policy 10 (AES 256, SHA, Group 5, PSK `<SITE_TO_SITE_PSK>`).
  - IPsec Transform-set `TS` (`esp-aes 256 esp-sha-hmac`).
  - ACL 110: `permit ip 10.0.0.0 0.255.255.255 10.50.10.0 0.0.0.255`.
  - Crypto map `CMAP` gán vào Gi0/0.
- **Trên Router Cơ sở 2 (`BR-R1`):**
  - Gi0/0 nối Viettel (`203.113.2.2/30`), Gi0/1 nối LAN Cơ sở 2 (`10.50.10.1/24`).
  - Pool DHCP `COSO2` cấp dải `10.50.10.11 - .254`.
  - Cấu hình IPsec đối xứng về peer `203.113.1.2`.
- **Trên Switch `BR-SW`:** Cấu hình VLAN 1 IP `10.50.10.2/24`, default gateway `10.50.10.1`.
- **PC-BR1, PC-BR2:** Đặt chế độ DHCP để nhận IP `10.50.10.x`.
- **Kích hoạt Tunnel:** Đứng từ PC-BR1 ping liên tục 4-5 lần về `10.1.50.5`. Kiểm tra `show crypto isakmp sa` báo `QM_IDLE ACTIVE`.

#### 2. Remote Access VPN (Easy VPN) trên EDGE-R2 cho PC-Remote
- Chạy lệnh `license boot module c2900 technology-package securityk9` và reload `EDGE-R2`.
- Cấu hình Easy VPN Server: AAA local, local pool `VPN-POOL 10.60.0.10 10.60.0.110`, group `REMOTE` (Key `<REMOTE_GROUP_KEY>`), user `vpnuser` / `<VPN_USER_PASSWORD>`.
- **Trên PC-Remote:**
  - Đặt IP tĩnh: `113.161.2.2/30`, GW `113.161.2.1`, DNS `10.1.50.5`.
  - Mở công cụ **VPN**: Điền thông số Group, Key, Host IP `113.161.1.2`, đăng nhập với `vpnuser` -> Kết nối thành công nhận IP `10.60.0.x`.

---

### GIAI ĐOẠN 4: BẢO MẬT BIÊN (ACL) & PHỐI HỢP CÙNG KHÁNH (Mục 2.7)

1. **Áp ACL `OUTSIDE-IN` trên cổng Gi0/0 của EDGE-R1 và EDGE-R2:**
   - Chỉ cho phép: HTTP vào Web DMZ, IPsec ISAKMP (UDP 500/4500), ESP, traffic TCP established và echo-reply.
   - Deny toàn bộ truy cập lạ từ ngoài Internet.
2. **Bàn giao file Master tạm thời cho Khánh:**
   - Bàn giao để Khánh hoàn tất giao diện: Server DHCP-DNS (12 pools), RADIUS, WLC1, AP, Smartphone kết nối Wi-Fi `UIT-SV`.

---

### GIAI ĐOẠN 5: BẢNG CHECKLIST NGHIỆM THU CHUNG (Mục 4 & Mục 6)

| # | Hạng mục kiểm tra | Thao tác / Thiết bị | Tiêu chuẩn ĐẠT (Passed) |
|---|---|---|---|
| 1 | **OSPF Toàn mạng** | `CORE-SW1# show ip ospf neighbor` | Đủ 10 neighbor trạng thái `FULL` |
| 2 | **Cân bằng tải Dual-ISP** | `DSW-A1# show ip route 0.0.0.0` | Thấy **2 đường O*E2 default route** song song qua 2 Core |
| 3 | **Internet & NAT** | PC bất kỳ trong campus | Ping `8.8.8.8` và `1.1.1.1` có reply |
| 4 | **Truy cập Server DMZ** | PC-Remote (ngoài Internet) | Mở trình duyệt Web IP `203.113.100.2`, `.3`, `.4` đều ra web |
| 5 | **DNS phân giải tên miền** | PC nội bộ campus | Mở `http://lms.campus.edu.vn` và `http://www.campus.edu.vn` thành công |
| 6 | **Dự phòng HSRP Core/Edge** | Shutdown Gi0/0 của EDGE-R1 | Mạng campus vẫn ping `8.8.8.8` bình thường qua EDGE-R2 |
| 7 | **VPN Site-to-Site** | PC-BR1 (Cơ sở 2) | Ping thông máy chủ `10.1.50.5` trong campus |
| 8 | **Remote Access VPN** | PC-Remote sau khi connect VPN | Mở được trang web nội bộ `http://10.1.50.7` (PORTAL-NB) |
| 9 | **An ninh ACL Biên** | PC-Remote | Ping IP WAN router biên `203.113.1.2` bị chặn hoàn toàn |

---

> 💡 **Lưu ý đặc biệt cho Khang khi làm việc với file Master:**
> - Luôn lưu dự phòng file với định dạng `Campus-Topology_backup_<gio>.pkt` trước khi dán khối cấu hình lớn.
> - Sau khi dán CLI, luôn gõ `write memory` hoặc `copy run start` để thiết bị lưu cấu hình NVRAM trong Packet Tracer.
