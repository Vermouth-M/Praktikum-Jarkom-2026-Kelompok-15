# LAPORAN TUGAS MODUL 4: DMZ & FIREWALL
**KELOMPOK 15**

## 1. Topologi Jaringan
Berikut adalah rancangan topologi jaringan yang digunakan dalam simulasi PNETLab untuk pemisahan zona WAN (Outside/External), LAN (Inside/Internal), dan DMZ (Demilitarized Zone):

<p align="center">
  <img src="images/topologi%20tm4.jpeg" alt="Topologi Jaringan Kelompok 15">
</p>

---

## 2. Tabel IP Address Kelompok 15

| No. | Perangkat | Interface | IP Address | Gateway | Keterangan |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | **MikroTik ISP** | ether1 | DHCP Client | *DHCP Lab* | Terhubung ke Cloud / Internet |
| | | ether2 | 10.10.10.1/30 | - | Terhubung ke FortiGate port1 |
| | | ether3 | 172.16.100.1/24 | - | Gateway untuk Client-WAN |
| 2 | **FortiGate** | port1 | 10.10.10.2/30 | 10.10.10.1 | Interface WAN (Outside) |
| | | port2 | 10.20.20.1/30 | - | Interface INSIDE ke Cisco |
| | | port3 | 192.168.20.1/24 | - | Interface DMZ |
| 3 | **Cisco Router** | G0/0 | 10.20.20.2/30 | - | Terhubung ke FortiGate port2 |
| | | G0/1 | 192.168.10.1/24 | - | Gateway LAN |
| 4 | **Client LAN** | eth0 | 192.168.10.10/24 | 192.168.10.1 | Client Internal (DNS: 8.8.8.8) |
| 5 | **Client WAN** | eth0 | 172.16.100.10/24 | 172.16.100.1 | Client Luar/Internet (DNS: 8.8.8.8) |
| 6 | **Ubuntu Server DMZ**| eth0 / ens3| 192.168.20.10/24 | 192.168.20.1 | Web Server DMZ (DNS: 8.8.8.8) |

---

## 3. Konfigurasi Perangkat

### 3.1 MikroTik ISP
```bash
# 1. Konfigurasi IP Address pada Interface
/ip address
add address=10.10.10.1/30 interface=ether2
add address=172.16.100.1/24 interface=ether3

# 2. Mengaktifkan DHCP Client pada ether1 ke Cloud
/ip dhcp-client
add interface=ether1 disabled=no

# 3. Konfigurasi NAT Masquerade menuju Internet
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade

# 4. Routing Static ke segmen internal via FortiGate (10.10.10.2)
/ip route
add dst-address=192.168.10.0/24 gateway=10.10.10.2
add dst-address=192.168.20.0/24 gateway=10.10.10.2
