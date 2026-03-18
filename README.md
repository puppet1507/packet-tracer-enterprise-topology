# Enterprise Network Infrastructure – Cisco Packet Tracer

## Giới thiệu

Dự án mô phỏng hạ tầng mạng doanh nghiệp hoàn chỉnh trên Cisco Packet Tracer, bao gồm một **trụ sở chính** (tòa nhà A & B) và một **chi nhánh** (tòa nhà C). Mô hình áp dụng kiến trúc 3 lớp chuẩn, tích hợp bảo mật, wireless, VPN và các cơ chế dự phòng.

---

## Kiến trúc tổng quan

Hệ thống được thiết kế theo mô hình **Core – Distribution – Access**:

- **Core Layer**: Hoạt động thuần Layer 3, làm backbone kết nối các Distribution Switch. Bật OSPF và default route lên tường lửa.
- **Distribution Layer**: Xử lý Inter-VLAN Routing thông qua SVI, cấu hình HSRP để tạo Virtual IP làm gateway dự phòng cho từng VLAN. Kết nối Core bằng routed port, kết nối Access bằng trunk.
- **Access Layer**: Kết nối thiết bị đầu cuối, cổng trunk lên Distribution, cổng access theo từng VLAN.

> Trụ sở có 2 Core Switch và 2 cặp Distribution Switch. Chi nhánh có 1 Core và 1 cặp Distribution Switch.

---

## Phân hoạch VLAN

### Trụ sở (Tòa A & B)

| VLAN | Tên           | Subnet              |
|------|---------------|---------------------|
| 10   | Server        | 192.168.10.0/24     |
| 30   | IT            | 192.168.30.0/24     |
| 40   | IT-Wifi       | 192.168.40.0/24     |
| 50   | Staff         | 192.168.50.0/24     |
| 60   | Staff-Wifi    | 192.168.60.0/24     |
| 70   | Printer       | 192.168.70.0/24     |
| 80   | Guest         | 192.168.80.0/24     |
| 19   | Management A  | 192.168.19.0/24     |
| 99   | Management B  | 192.168.99.0/24     |

### Chi nhánh (Tòa C)

| VLAN | Tên        | Subnet              |
|------|------------|---------------------|
| 20   | IT-Branch  | 192.168.20.0/24     |
| 100  | Server     | 192.168.100.0/24    |
| 199  | Management | 192.168.199.0/24    |

---

## Các tính năng đã triển khai

### Định tuyến & Dự phòng
- **Inter-VLAN Routing** tại Distribution Layer thông qua SVI
- **OSPF** chạy xuyên suốt từ Distribution → Core → ASA
- **HSRP** tại Distribution Switch, tạo Virtual IP làm default gateway cho từng VLAN
- **Port Channel + Trunk** giữa hai Distribution Switch trong cùng một cặp

### DHCP
- Cấu hình `ip helper-address` trên các SVI tại Distribution Switch
- DHCP Server tập trung, tạo pool riêng cho từng VLAN

### Wireless
- **WLC (Wireless LAN Controller)** đặt tại tòa A, VLAN 19
- **Access Point** dùng trunk với native VLAN 99 để nhận IP
- Hỗ trợ nhiều SSID ánh xạ nhiều VLAN (IT-Wifi, Staff-Wifi, Guest)
- Dùng **FlexConnect (Local Switching)** do lỗi Central Switching trên Packet Tracer

### Bảo mật – Tường lửa (ASA)
Mỗi site dùng 2 ASA:
- **ASA1**: Kết nối Internet, cấu hình NAT (PAT) cho traffic nội bộ ra ngoài, static NAT cho Web Server và External DNS. Bật OSPF và ACL trên interface outside.
- **ASA2**: Xử lý VPN IPSec site-to-site với chi nhánh. Không NAT (tách riêng do giới hạn của ASA trên Packet Tracer).

### ACL
- Áp dụng ACL tại Distribution Switch
- VLAN Guest: chỉ được phép DHCP, DNS và truy cập Internet; chặn toàn bộ kết nối đến mạng nội bộ
  
  <img width="498" height="169" alt="Screenshot 2026-03-19 051909" src="https://github.com/user-attachments/assets/e037c785-9e94-4546-b005-5559d857aa89" />



### VPN
- **IPSec Site-to-Site VPN** giữa trụ sở và chi nhánh qua ASA2
- Traffic VPN đi theo default route riêng từ Core → ASA2

### Internet Simulation
- Giả lập Internet với 1 PC và 1 DNS Server (8.8.8.8)
  
  <img width="874" height="432" alt="Screenshot 2026-03-19 062735" src="https://github.com/user-attachments/assets/fefdba31-0543-4558-9563-93fff7f1a08d" />


- DMZ chứa Web Server và External DNS Server
  
  <img width="873" height="444" alt="Screenshot 2026-03-19 062704" src="https://github.com/user-attachments/assets/a0819a52-50e6-48d8-9313-2eda167b3abb" />



---

## Topology

  <img width="1424" height="699" alt="Screenshot 2026-03-19 063005" src="https://github.com/user-attachments/assets/d2d48065-c893-4714-9e6c-e88923675f47" />



---

## Kiểm thử

- Tracert từ VLAN 30 (IT) đến các VLAN khác trong trụ sở.
  
  <img width="490" height="293" alt="Screenshot 2026-03-19 053832" src="https://github.com/user-attachments/assets/41dd8fa1-273d-4972-bac6-721a299c7d11" />


- Tracert từ VLAN 30 đến VLAN 20 (IT-Branch) qua VPN
  
  <img width="492" height="216" alt="Screenshot 2026-03-19 054625" src="https://github.com/user-attachments/assets/51726e1e-e804-422b-8b39-380c75e948f6" />


- ACL Guest – chặn truy cập nội bộ
  
  <img width="631" height="624" alt="Screenshot 2026-03-19 061027" src="https://github.com/user-attachments/assets/75c54ebc-1d96-41e1-9992-44a79ec0c3e7" />


- PC Public truy cập Web Server qua NAT: PC gửi DNS query đến 8.8.8.8, được forward đến External DNS Server để resolve và cache lại. PC dùng IP nhận được để kết nối. (External DNS Server phải dùng bản ghi Nat IP).
  
  <img width="869" height="336" alt="Screenshot 2026-03-19 062516" src="https://github.com/user-attachments/assets/b5b775a8-8d5a-4f55-878f-6e50d00386e6" />



---

## Troubleshooting

**1. WLC – Central Switching bị lỗi trên Packet Tracer**  
Central Switching khiến việc mapping VLAN tại WLC bị sai. Giải pháp: chuyển sang **FlexConnect (Local Switching)** để AP tự xử lý mapping VLAN.

**2. ASA1 – Dynamic NAT không hoạt động ngay**  
Cần chạy một lệnh static NAT trước, sau đó mới chạy lại dynamic NAT thì mới hoạt động.

**3. Routing loop khi kết hợp VPN và OSPF**  
Default route từ Core trỏ đến ASA VPN khiến gói tin bị loop ngược lại. Giải pháp: thêm default route trực tiếp trên ASA VPN. Lưu ý: `redistribute static subnets` trên ASA trong Packet Tracer không hoạt động.

---
