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
```

### 3.2 Konfigurasi Fortigate Firewall
```bash
config system interface
    edit "port1"
        set mode static
        set ip 10.10.10.2 255.255.255.252
        set allowaccess ping
    next
    edit "port2"
        set mode static
        set ip 10.20.20.1 255.255.255.252
        set allowaccess ping
    next
    edit "port3"
        set mode static
        set ip 192.168.20.1 255.255.255.0
        set allowaccess ping
    next
end

# Routing (Default Route ke ISP & Static Route ke LAN)
config router static
    edit 1
        set gateway 10.10.10.1
        set device "port1"
    next
    edit 2
        set dst 192.168.10.0 255.255.255.0
        set gateway 10.20.20.2
        set device "port2"
    next
end

# Pembuatan Virtual IP (Port Forwarding Port 80 WAN ke Server DMZ)
config firewall vip
    edit "VIP_DMZ_HTTP"
        set extip 10.10.10.2
        set extintf "port1"
        set mappedip "192.168.20.10"
        set portforward enable
        set protocol tcp
        set extport 80
        set lport 80
    next
end

# Kebijakan Firewall (Firewall Policy)
config firewall policy
    edit 1
        set name "LAN_to_WAN"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
    edit 2
        set name "LAN_to_DMZ"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
    next
    edit 3
        set name "WAN_to_DMZ_HTTP"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "VIP_DMZ_HTTP"
        set action accept
        set schedule "always"
        set service "HTTP"
    next
end

```
### 3.3 Konfigurasi Cisco Router
```bash
# Mengakses mode konfigurasi
Router> enable
Router# configure terminal

# Konfigurasi interface yang terhubung ke FortiGate (port2)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 10.20.20.2 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

# Konfigurasi interface yang terhubung ke Client LAN
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

# Routing ke arah FortiGate
Router(config)# ip route 0.0.0.0 0.0.0.0 10.20.20.1
Router(config)# exit
Router# copy running-config startup-config
```

### 3.4 Konfigurasi Ubuntu Server DMZ
```bash
# Mengubah konten default halaman Nginx
sudo echo "Tumod_4_DMZ_Firewall_15-Kelompok15" > /var/www/html/index.html

# Memastikan service berjalan otomatis
sudo systemctl enable nginx
sudo systemctl restart nginx
```
