# 📋 Report – Phần việc của Khánh (Campus nội bộ)

> [!IMPORTANT]
> Nguồn số liệu duy nhất: Sheet **"Quy hoạch VLAN & IP – Mạng Campus (Đề tài 1)"** (tab *Subnet*, *IP thiết bị*, *Cổng & VLAN*). Mọi IP/VLAN trong file config đã được đối chiếu với README.md là bản trích từ Sheet đó.

---

## ✅ Trạng thái hoàn thành

| Mục | Nội dung | File | Trạng thái |
|---|---|---|---|
| 1.1 | DSW-A1 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-A1.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-A2 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-A2.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-B1 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-B1.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-B2 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-B2.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-C1 L2: VLAN, EtherChannel, trunk, SVI+HSRP | `DSW-C1.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-C2 L2: VLAN, EtherChannel, trunk, SVI+HSRP | `DSW-C2.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-K1 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-K1.L2.txt` | ✅ Đã tạo |
| 1.1 | DSW-K2 L2: VLAN, EtherChannel, trunk, SVI+HSRP, ACL | `DSW-K2.L2.txt` | ✅ Đã tạo |
| 1.2 | ASW-A1: trunk, access, port-security, DHCP snooping, DAI | `ASW-A1.txt` | ✅ Đã tạo |
| 1.2 | ASW-A2: trunk, access, port-security, DHCP snooping, DAI | `ASW-A2.txt` | ✅ Đã tạo |
| 1.2 | ASW-B1: trunk, access, port-security, DHCP snooping, DAI | `ASW-B1.txt` | ✅ Đã tạo |
| 1.2 | ASW-B2: trunk, access, port-security, DHCP snooping, DAI | `ASW-B2.txt` | ✅ Đã tạo |
| 1.2 | ASW-C1: trunk, access, port-security, DHCP snooping, DAI | `ASW-C1.txt` | ✅ Đã tạo |
| 1.2 | ASW-C2: trunk, access, port-security, DHCP snooping, DAI | `ASW-C2.txt` | ✅ Đã tạo |
| 1.2 | ASW-K1: trunk, access, port-security, DHCP snooping, DAI | `ASW-K1.txt` | ✅ Đã tạo |
| 1.2 | ASW-K2: trunk, access, port-security, DHCP snooping, DAI | `ASW-K2.txt` | ✅ Đã tạo |
| 1.3 | SRV-SW: trunk lên DSW-A1/A2, access VLAN 50, trunk WLC | `SRV-SW.txt` | ✅ Đã tạo |
| 1.4 | DHCP-DNS (giao diện) | File master | Đã config trên Packet Tracer |
| 1.5 | RADIUS, PORTAL-NB (giao diện) | File master | Đã config trên Packet Tracer |
| 1.6 | WLC1, AP, smartphone (giao diện) | File master | ⏳ Sau khi Khang xong định tuyến |
| 1.7 | PC, Camera (giao diện DHCP) | File master | ⏳ Cần làm trên Packet Tracer |

---

## 📁 Danh sách file CLI đã tạo

```
configs/
  DSW-A1.L2.txt    ← Khánh ✅
  DSW-A2.L2.txt    ← Khánh ✅
  DSW-B1.L2.txt    ← Khánh ✅
  DSW-B2.L2.txt    ← Khánh ✅
  DSW-C1.L2.txt    ← Khánh ✅
  DSW-C2.L2.txt    ← Khánh ✅
  DSW-K1.L2.txt    ← Khánh ✅
  DSW-K2.L2.txt    ← Khánh ✅
  ASW-A1.txt       ← Khánh ✅
  ASW-A2.txt       ← Khánh ✅
  ASW-B1.txt       ← Khánh ✅
  ASW-B2.txt       ← Khánh ✅
  ASW-C1.txt       ← Khánh ✅
  ASW-C2.txt       ← Khánh ✅
  ASW-K1.txt       ← Khánh ✅
  ASW-K2.txt       ← Khánh ✅
  SRV-SW.txt       ← Khánh ✅
```

---

## 🔍 Chi tiết cấu hình đã thực hiện

### 1.1 Distribution (8 DSW) – L2 + SVI + HSRP + ACL

#### Quy tắc chung
- **VTP**: `transparent` trên mọi switch (không dùng server/client)
- **STP**: `rapid-pvst`, DSW số 1 = `root primary`, DSW số 2 = `root secondary`
- **EtherChannel**: Gi1/0/3–4, LACP mode `active` ↔ `active`, trunk native 999
- **HSRP v2**: DSW số 1 = Active (priority 110), DSW số 2 = Standby (priority 100), preempt bật ở cả hai
- **ip helper-address 10.1.50.5** chỉ trên VLAN có DHCP (không đặt cho VLAN 50 và 99)

#### Khối A (DSW-A1 / DSW-A2)

| Yếu tố | Giá trị |
|---|---|
| VLAN | 10, 20, 30, 50, 60, 99, 999 |
| EtherChannel Po1 | Gi1/0/3-4, allowed 10,20,30,50,60,99, native 999 |
| Trunk → ASW-A1/A2 | Gi1/0/5-6, allowed 10,20,60,99, native 999 |
| Trunk → SRV-SW | Gi1/0/7, allowed 30,50,99, native 999 |
| VLAN 10 SVI | DSW-A1: `10.1.10.2/24`, DSW-A2: `10.1.10.3/24`, HSRP VIP `10.1.10.1` |
| VLAN 20 SVI | DSW-A1: `10.1.20.2/24`, DSW-A2: `10.1.20.3/24`, HSRP VIP `10.1.20.1` |
| VLAN 30 SVI | DSW-A1: `10.20.0.2/19`, DSW-A2: `10.20.0.3/19`, HSRP VIP `10.20.0.1` |
| VLAN 50 SVI | DSW-A1: `10.1.50.2/24`, DSW-A2: `10.1.50.3/24`, HSRP VIP `10.1.50.1` |
| VLAN 60 SVI | DSW-A1: `10.1.60.2/24`, DSW-A2: `10.1.60.3/24`, HSRP VIP `10.1.60.1` |
| VLAN 99 SVI | DSW-A1: `10.1.99.2/24`, DSW-A2: `10.1.99.3/24`, HSRP VIP `10.1.99.1` |
| ACL SV-WIFI-IN | Trên Vlan30 (in): chặn VLAN30 vào dải 10.1.10.0, 10.x.99.0 |
| ACL CAM-IN | Trên Vlan60 (in): chặn cam vào nơi ngoài 10.1.50.0; cho phép HSRP 224.0.0.102:1985 |

#### Khối B (DSW-B1 / DSW-B2)

| Yếu tố | Giá trị |
|---|---|
| VLAN | 20, 60, 70, 80, 99, 999 |
| EtherChannel Po1 | Gi1/0/3-4, allowed 20,60,70,80,99, native 999 |
| Trunk → ASW-B1/B2 | Gi1/0/5-6, allowed 20,60,70,80,99, native 999 |
| VLAN 20 SVI | DSW-B1: `10.2.20.2/24`, DSW-B2: `10.2.20.3/24`, VIP `10.2.20.1` |
| VLAN 60 SVI | DSW-B1: `10.2.60.2/24`, DSW-B2: `10.2.60.3/24`, VIP `10.2.60.1` |
| VLAN 70 SVI | DSW-B1: `10.2.70.2/24`, DSW-B2: `10.2.70.3/24`, VIP `10.2.70.1` |
| VLAN 80 SVI | DSW-B1: `10.2.80.2/24`, DSW-B2: `10.2.80.3/24`, VIP `10.2.80.1` |
| VLAN 99 SVI | DSW-B1: `10.2.99.2/24`, DSW-B2: `10.2.99.3/24`, VIP `10.2.99.1` |
| ACL CAM-IN | Trên Vlan60 (in): cam khối B chỉ vào 10.1.50.0 |

#### Khối C (DSW-C1 / DSW-C2)

| Yếu tố | Giá trị |
|---|---|
| VLAN | 40, 70, 99, 999 |
| EtherChannel Po1 | Gi1/0/3-4, allowed 40,70,99, native 999 |
| Trunk → ASW-C1/C2 | Gi1/0/5-6, allowed 40,70,99, native 999 |
| VLAN 40 SVI | DSW-C1: `10.3.40.2/23`, DSW-C2: `10.3.40.3/23`, VIP `10.3.40.1` |
| VLAN 70 SVI | DSW-C1: `10.3.70.2/24`, DSW-C2: `10.3.70.3/24`, VIP `10.3.70.1` |
| VLAN 99 SVI | DSW-C1: `10.3.99.2/24`, DSW-C2: `10.3.99.3/24`, VIP `10.3.99.1` |

#### Khối K (DSW-K1 / DSW-K2)

| Yếu tố | Giá trị |
|---|---|
| VLAN | 60, 70, 99, 999 |
| EtherChannel Po1 | Gi1/0/3-4, allowed 60,70,99, native 999 |
| Trunk → ASW-K1/K2 | Gi1/0/5-6, allowed 60,70,99, native 999 |
| VLAN 60 SVI | DSW-K1: `10.4.60.2/24`, DSW-K2: `10.4.60.3/24`, VIP `10.4.60.1` |
| VLAN 70 SVI | DSW-K1: `10.4.70.2/24`, DSW-K2: `10.4.70.3/24`, VIP `10.4.70.1` |
| VLAN 99 SVI | DSW-K1: `10.4.99.2/24`, DSW-K2: `10.4.99.3/24`, VIP `10.4.99.1` |
| ACL CAM-IN | Trên Vlan60 (in): cam khối K chỉ vào 10.1.50.0 |

---

### 1.2 Access (8 ASW)

Mỗi ASW áp dụng:
- **VTP transparent**, khai báo VLAN phù hợp khối
- **Uplink G0/1-2**: trunk, native 999, allowed VLAN theo khối
- **Cổng PC/Camera/AP**: access mode, portfast, bpduguard, port-security (max 2, sticky, restrict)
- **Cổng không dùng (Fa0/x–24)**: access VLAN 999, shutdown
- **VLAN 99**: IP quản trị, `ip default-gateway`
- **DHCP Snooping**: bật theo VLAN có DHCP, uplink trusted, **`no ip dhcp snooping information option`** (bắt buộc!)
- **DAI**: thử; nếu Packet Tracer báo lỗi thì bỏ qua

| ASW | Fa0/1 | Fa0/2 | Fa0/3 | IP quản trị |
|---|---|---|---|---|
| ASW-A1 | VLAN 10 (PC-BGH) | VLAN 10 (PC-HanhChinh) | — | `10.1.99.11/24` |
| ASW-A2 | VLAN 20 (PC-GV-A) | VLAN 60 (CAM-A) | — | `10.1.99.12/24` |
| ASW-B1 | VLAN 20 (PC-GV-B) | VLAN 70 (LAP-B1) | — | `10.2.99.11/24` |
| ASW-B2 | VLAN 70 (LAP-B2) | VLAN 60 (CAM-B) | VLAN 80 (PC-ThuVien) | `10.2.99.12/24` |
| ASW-C1 | VLAN 40 (PC-Lab1) | VLAN 40 (PC-Lab2) | VLAN 70 (LAP-C1) | `10.3.99.11/24` |
| ASW-C2 | VLAN 40 (PC-Lab3) | VLAN 40 (PC-Lab4) | — | `10.3.99.12/24` |
| ASW-K1 | VLAN 70 (LAP-K1) | VLAN 60 (CAM-K1) | — | `10.4.99.11/24` |
| ASW-K2 | VLAN 70 (LAP-K2) | VLAN 60 (CAM-K2) | — | `10.4.99.12/24` |

---

### 1.3 SRV-SW

| Yếu tố | Giá trị |
|---|---|
| Uplink G0/1 | Trunk → DSW-A1, native 999, allowed 30,50,99 |
| Uplink G0/2 | Trunk → DSW-A2, native 999, allowed 30,50,99 |
| Fa0/1 | Access VLAN 50 → DHCP-DNS |
| Fa0/2 | Access VLAN 50 → RADIUS |
| Fa0/3 | Access VLAN 50 → PORTAL-NB |
| Fa0/4 | Trunk → WLC1, **native VLAN 50**, allowed 30,50 |
| IP quản trị | `10.1.99.13/24`, GW `10.1.99.1` |

> [!NOTE]
> Fa0/4 (cổng WLC1) dùng **native VLAN 50** (ngoại lệ duy nhất, các trunk còn lại native 999). Điều này để WLC nhận được IP quản lý `10.1.50.10` không gắn tag.

---

## 🖥️ Hướng dẫn thao tác giao diện (cần làm trên file master)

### 1.4 Server DHCP-DNS (`10.1.50.5/24`)

1. **Desktop → IP Configuration**: Static, IP `10.1.50.5`, mask `255.255.255.0`, gateway `10.1.50.1`, DNS `10.1.50.5`
2. **Services → DHCP → On**. Sửa `serverPool` → save
3. Thêm **12 pool DHCP**:

| Pool | Gateway | Start IP | Mask | Max | WLC |
|---|---|---|---|---|---|
| A-BGH | 10.1.10.1 | 10.1.10.11 | 255.255.255.0 | 244 | — |
| A-GV | 10.1.20.1 | 10.1.20.11 | 255.255.255.0 | 244 | — |
| A-CAM | 10.1.60.1 | 10.1.60.11 | 255.255.255.0 | 244 | — |
| SV-WIFI | 10.20.0.1 | 10.20.0.11 | 255.255.224.0 | 8180 | — |
| B-GV | 10.2.20.1 | 10.2.20.11 | 255.255.255.0 | 244 | — |
| B-CAM | 10.2.60.1 | 10.2.60.11 | 255.255.255.0 | 244 | — |
| B-AP | 10.2.70.1 | 10.2.70.11 | 255.255.255.0 | 244 | **10.1.50.10** |
| B-THUVIEN | 10.2.80.1 | 10.2.80.11 | 255.255.255.0 | 244 | — |
| C-LAB | 10.3.40.1 | 10.3.40.11 | 255.255.254.0 | 500 | — |
| C-AP | 10.3.70.1 | 10.3.70.11 | 255.255.255.0 | 244 | **10.1.50.10** |
| K-CAM | 10.4.60.1 | 10.4.60.11 | 255.255.255.0 | 244 | — |
| K-AP | 10.4.70.1 | 10.4.70.11 | 255.255.255.0 | 244 | **10.1.50.10** |

4. **Services → DNS → On**, thêm 3 record A:

| Name | Address |
|---|---|
| www.campus.edu.vn | 10.100.0.10 |
| lms.campus.edu.vn | 10.100.0.11 |
| elib.campus.edu.vn | 10.100.0.12 |

---

### 1.5 RADIUS (`10.1.50.6/24`)

- IP: `10.1.50.6`, mask `255.255.255.0`, GW `10.1.50.1`, DNS `10.1.50.5`
- **Services → AAA → On**
- Network Config: `WLC1`, IP `10.1.50.10`, Secret `<RADIUS_SHARED_SECRET>`, Type `Radius` → Add
- User Setup: `sv001/<SV001_PASSWORD>`, `sv002/<SV002_PASSWORD>`, `sv003/<SV003_PASSWORD>` → Add

### 1.5 PORTAL-NB (`10.1.50.7/24`)

- IP: `10.1.50.7`, mask `255.255.255.0`, GW `10.1.50.1`, DNS `10.1.50.5`
- **Services → HTTP On**, sửa `index.html` thành "Cổng thông tin nội bộ"

---

### 1.6 WLC1 (sau khi Khang xong định tuyến)

1. **Config → Management**: IP `10.1.50.10`, mask `255.255.255.0`, GW `10.1.50.1`
2. Từ PC-BGH: `https://10.1.50.10` → tạo admin
3. **Controller → Interfaces → New**: `vlan30`, VLAN `30`, IP `10.20.0.10/19`, GW `10.20.0.1`, DHCP `10.1.50.5`, Port `1`
4. **Security → AAA → RADIUS**: Server `10.1.50.6`, Secret `<RADIUS_SHARED_SECRET>`
5. **WLANs → Create**: SSID `UIT-SV`, Interface `vlan30`, WPA2+AES, 802.1X, RADIUS `10.1.50.6`
6. Chờ 5 AP join: LAP-B1, LAP-B2, LAP-C1, LAP-K1, LAP-K2

### 1.7 Thiết bị đầu cuối

- PC-BGH, PC-HanhChinh, PC-GV-A/B, PC-ThuVien, PC-Lab1–4: **DHCP**
- CAM-A, CAM-B, CAM-K1, CAM-K2: Config → Fa0 → **DHCP**
- AP (LAP-…): tự lấy IP, không cần cấu hình
- SV-Phone1/2/3: SSID `UIT-SV`, WPA2-Enterprise, ID `sv001/2/3`, IP **DHCP**

---

## ⚠️ Lưu ý quan trọng khi dán CLI

1. **`switchport trunk encapsulation dot1q`**: thêm ngay trước `switchport mode trunk` nếu IOS báo lỗi
2. **Dán từng khối nhỏ** (giữa hai `!`), đợi chạy xong mới dán tiếp
3. **Thay placeholder** trước khi dán: `<ENABLE_SECRET>`, `<RADIUS_SHARED_SECRET>`, v.v.
4. Dán xong → `show running-config` đối chiếu
5. Chạy `write memory` hoặc `copy run start` sau mỗi thiết bị

## 🔑 Điểm then chốt tránh lỗi

| Vấn đề | Giải pháp đã tích hợp |
|---|---|
| HSRP cả hai cùng Active | `permit udp any host 224.0.0.102 eq 1985` trong `CAM-IN` ✅ |
| PC không nhận DHCP qua relay | `no ip dhcp snooping information option` trên mọi ASW ✅ |
| AP không join WLC | Pool AP (B/C/K) có `WLC Address 10.1.50.10` ✅ |
| Port-channel (SD) | Hai đầu cùng `mode active`, cùng allowed VLAN ✅ |

---

## 📅 Bảng nghiệm thu (Khánh)

| Mốc | Mục | Nội dung | Kết quả |
|---|---|---|---|
| Ngày 2 | 6.1-A | Distribution 8 DSW | ⬜ |
| Ngày 2 | 6.1-B | Access 8 ASW + SRV-SW | ⬜ |
| Ngày 3 | 6.1-C | DHCP và thiết bị đầu cuối | ⬜ |
| Ngày 4 | 6.1-D | Wi-Fi | ⬜ |
| Ngày 4 | 6.1-E | ACL và HSRP failover | ⬜ |


## Cấu hình CLI phần Khang

Các file dưới đây được tổng hợp từ mục 2 của README và các điều chỉnh đã kiểm tra trong [bản Packet Tracer](../NT113_Khang.pkt). Chúng là lệnh triển khai theo thiết bị. Bản bàn giao và kết quả nghiệm thu nằm trong [biên bản phần Khang](../BAN_GIAO_PHAN_KHANG.md).

| Nhóm | File theo thiết bị |
|---|---|
| Core | [CORE-SW1](CORE-SW1.txt), [CORE-SW2](CORE-SW2.txt) |
| Định tuyến DSW | [A1](DSW-A1.routing.txt), [A2](DSW-A2.routing.txt), [B1](DSW-B1.routing.txt), [B2](DSW-B2.routing.txt), [C1](DSW-C1.routing.txt), [C2](DSW-C2.routing.txt), [K1](DSW-K1.routing.txt), [K2](DSW-K2.routing.txt) |
| ISP | [Viettel](ISP-Viettel.txt), [VNPT](ISP-VNPT.txt) |
| Router biên | [EDGE-R1](EDGE-R1.txt), [EDGE-R2](EDGE-R2.txt) |
| VPN và cơ sở 2 | [EDGE-R1 IPsec](EDGE-R1.vpn.txt), [EDGE-R2 Easy VPN](EDGE-R2.vpn.txt), [BR-R1](BR-R1.txt), [BR-SW](BR-SW.txt) |
| ACL biên | [EDGE-R1](EDGE-R1.acl.txt), [EDGE-R2](EDGE-R2.acl.txt) |

Vào chế độ `configure terminal` trước khi áp từng file. Các file định tuyến DSW áp sau file L2 tương ứng. Bật `securityk9`, lưu và reload EDGE-R1, EDGE-R2 và BR-R1 trước khi áp phần crypto. Trong Packet Tracer, nếu gói đang ở trạng thái `disable`, gỡ trạng thái đó trước khi bật gói. Điền các placeholder VPN bằng thông tin lab trao đổi riêng. Sau khi đã kiểm tra định tuyến và VPN, áp các file ACL biên. Bộ lọc chống giả mạo nguồn của ISP-VNPT nằm trong file cấu hình ISP.

IP và HTTP của các máy chủ DMZ, các bản ghi DNS cùng cấu hình IP/VPN của PC được thao tác trong giao diện Packet Tracer và lưu trong file mô phỏng.
