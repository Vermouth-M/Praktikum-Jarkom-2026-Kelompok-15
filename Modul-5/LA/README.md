# Implementasi Jaringan Enterprise HQ–Branch dengan VRRP, ISC-DHCP, FortiGate, GRE Tunnel, dan OSPF
### Modul 5: VLAN, Trunk, OSPF Multi-Vendor

Repository ini berisi dokumentasi dan laporan hasil pengerjaan Tugas Modul 5 Praktikum Jaringan Komputer 2026 mengenai implementasi jaringan Enterprise Multi-Vendor skala HQ-Branch.

---

## 1. Deskripsi Singkat & Topologi

Topologi ini mensimulasikan jaringan enterprise yang menghubungkan 2 lokasi utama, yaitu **Kantor Pusat (HQ) di Jakarta** dan **Kantor Cabang (Branch) di Surabaya**. Kedua site tersebut dihubungkan melalui jaringan **ISP** menggunakan teknologi **GRE Tunnel** dan routing dinamis **OSPF over GRE** agar dapat saling berkomunikasi secara aman dan dinamis melintasi infrastruktur publik.

### Perangkat yang Digunakan:
- **Cisco Vios Router** & **Cisco Switch**
- **MikroTik RouterOS** (Internal & ISP)
- **Fortinet FortiGate** (Edge Firewall & GRE Endpoint)
- **Ubuntu Server** (Centralized ISC-DHCP & Nginx Web Server)
- **Tinycore Linux** & **VPCS** (Client Nodes)

### Gambar Topologi Jaringan
<img src="tumod p5/topologi-p5.jpeg" alt="Topologi Jaringan Enterprise" width="800">

---

## 2. Tabel Addressing

### Sisi Jakarta / HQ
#### 2.1 VLAN Jakarta
| VLAN | Nama VLAN | Network | Gateway Virtual | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| 10 | FINANCE | 192.168.10.0/24 | 192.168.10.1 | DHCP dari Ubuntu Server Jakarta |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | DHCP dari Ubuntu Server Jakarta |
| 60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1 | VLAN server Ubuntu Jakarta |

#### 2.2 IP Address Cisco Router Jakarta
| Interface | VLAN / Link | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| Gi0/1.10 | VLAN 10 | 192.168.10.2/24 | IP fisik Cisco untuk VLAN 10 |
| Gi0/1.20 | VLAN 20 | 192.168.20.2/24 | IP fisik Cisco untuk VLAN 20 |
| Gi0/1.60 | VLAN 60 | 192.168.60.2/24 | IP fisik Cisco untuk VLAN 60 |
| Gi0/0 | Link ke FortiGate JKT | 10.10.100.2/30 | Transit Cisco JKT ke FortiGate JKT |

#### 2.3 IP Address MikroTik Router Jakarta
| Interface | VLAN / Link | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| vlan10-finance | VLAN 10 | 192.168.10.3/24 | IP fisik MikroTik untuk VLAN 10 |
| vlan20-it | VLAN 20 | 192.168.20.3/24 | IP fisik MikroTik untuk VLAN 20 |
| vlan60-server | VLAN 60 | 192.168.60.3/24 | IP fisik MikroTik untuk VLAN 60 |
| ether1 | Link ke FortiGate JKT | 10.10.101.2/30 | Transit MikroTik JKT ke FortiGate JKT |

#### 2.4 VRRP Jakarta
| VLAN | Virtual IP | Master | Backup | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| 10 | 192.168.10.1 | Cisco Router Jakarta | MikroTik Router Jakarta | Gateway virtual VLAN 10 |
| 20 | 192.168.20.1 | MikroTik Router Jakarta | Cisco Router Jakarta | Gateway virtual VLAN 20 |
| 60 | 192.168.60.1 | Cisco Router Jakarta | MikroTik Router Jakarta | Gateway virtual VLAN 60 |

#### 2.5 Ubuntu Server Jakarta
| Perangkat | VLAN | IP Address | Gateway | Service |
| :--- | :--- | :--- | :--- | :--- |
| Ubuntu Server | 60 | 192.168.60.10/24 | 192.168.60.1 | ISC-DHCP Server & Nginx Web Server |

#### 2.6 DHCP Pool Jakarta
| VLAN | Network | Range DHCP | Gateway | DHCP Server |
| :--- | :--- | :--- | :--- | :--- |
| 10 | 192.168.10.0/24 | 192.168.10.100 - 192.168.10.200 | 192.168.10.1 | Ubuntu Server Jakarta |
| 20 | 192.168.20.0/24 | 192.168.20.100 - 192.168.20.200 | 192.168.20.1 | Ubuntu Server Jakarta |

#### 2.7 FortiGate Jakarta
| Interface | Terhubung ke | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| port1 | Cisco Router Jakarta | 10.10.100.1/30 | Link ke Cisco Jakarta |
| port2 | MikroTik Router JKT | 10.10.101.1/30 | Link ke MikroTik Jakarta |
| port3 | MikroTik ISP | 10.0.12.2/30 | Link WAN ke ISP |
| GRE-JKT-SBY | FortiGate Surabaya | 172.16.0.1/32 | IP GRE Tunnel Jakarta |

---

### Sisi ISP
#### 3.1 MikroTik ISP
| Interface | Terhubung ke | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| ether2 | FortiGate Jakarta | 10.0.12.1/30 | Link ISP ke Jakarta |
| ether3 | FortiGate Surabaya | 10.0.13.1/30 | Link ISP ke Surabaya |
| ether1 | Cloud NAT / Internet | DHCP | Akses internet simulasi |

#### 3.2 Link WAN ISP
| Link | Network | Sisi A | IP Sisi A | Sisi B | IP Sisi B |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Jakarta ↔ ISP | 10.0.12.0/30 | MikroTik ISP | 10.0.12.1 | FortiGate JKT | 10.0.12.2 |
| ISP ↔ Surabaya | 10.0.13.0/30 | MikroTik ISP | 10.0.13.1 | FortiGate SBY | 10.0.13.2 |

---

### Sisi Surabaya / Branch
#### 4.1 VLAN Surabaya
| VLAN | Nama VLAN | Network | Gateway | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| 30 | SALES | 192.168.30.0/24 | 192.168.30.1 | DHCP dari MikroTik Surabaya |
| 40 | OPERATIONS | 192.168.40.0/24 | 192.168.40.1 | IP static manual |

#### 4.2 IP Address MikroTik Router Surabaya
| Interface | VLAN / Link | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| vlan30-sales | VLAN 30 | 192.168.30.1/24 | Gateway VLAN 30 |
| vlan40-operations| VLAN 40 | 192.168.40.1/24 | Gateway VLAN 40 |
| ether1 | Link ke FortiGate SBY | 10.10.200.2/30 | Transit MikroTik SBY ke FortiGate SBY |

#### 4.3 DHCP Pool Surabaya
| VLAN | Network | Range DHCP | Gateway | DHCP Server |
| :--- | :--- | :--- | :--- | :--- |
| 30 | 192.168.30.0/24 | 192.168.30.100 - 192.168.30.200 | 192.168.30.1 | MikroTik Surabaya |
| 40 | 192.168.40.0/24 | Static manual | 192.168.40.1 | Tidak menggunakan DHCP |

#### 4.4 IP Client Surabaya
| Client | VLAN | IP Address | Gateway | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| PC Sales | 30 | DHCP | 192.168.30.1 | Mendapat IP dari MikroTik SBY |
| PC Operations | 40 | 192.168.40.10/24 | 192.168.40.1 | IP static manual |
| PC Ops Tinycore | 40 | 192.168.40.20/24 | 192.168.40.1 | IP static manual |

#### 4.5 FortiGate Surabaya
| Interface | Terhubung ke | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| port1 | MikroTik ISP | 10.0.13.2/30 | Link WAN ke ISP |
| port2 | MikroTik Surabaya | 10.10.200.1/30 | Link ke internal Surabaya |
| GRE-SBY-JKT | FortiGate Jakarta | 172.16.0.2/32 | IP GRE Tunnel Surabaya |

---

### GRE Tunnel & OSPF Network Ads

#### GRE Tunnel Configuration
- **GRE-JKT-SBY**: Local `10.0.12.2` $\rightarrow$ Remote `10.0.13.2` | Tunnel IP: `172.16.0.1/32`
- **GRE-SBY-JKT**: Local `10.0.13.2` $\rightarrow$ Remote `10.0.12.2` | Tunnel IP: `172.16.0.2/32`

#### OSPF Advertised Network
- **Sisi Jakarta**: `192.168.10.0/24`, `192.168.20.0/24`, `192.168.60.0/24`, `172.16.0.1/32`
- **Sisi Surabaya**: `192.168.30.0/24`, `192.168.40.0/24`, `172.16.0.2/32`

---

## 3. Hasil Pembagian Tugas & Dokumentasi Modul

### Tugas Modul 1 — Konfigurasi Cisco Switch Jakarta
- **Konfigurasi:** Pembuatan VLAN 10, 20, 60. Pengaturan Access Port ke client dan Ubuntu Server, serta setup Trunking link membawa VLAN 10, 20, dan 60 menuju Cisco & MikroTik Router.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 1/WhatsApp Image 2026-06-14 at 14.16.47.jpeg" alt="Konfigurasi Cisco Switch Jakarta" width="800">

### Tugas Modul 2 — Konfigurasi Cisco Router Jakarta
- **Konfigurasi:** Subinterface VLAN 10, 20, 60, IP fisik, VRRP setup (Master untuk VLAN 10 & 60), DHCP Relay ke Ubuntu Server, dan transit link default-route ke FortiGate.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 2/WhatsApp Image 2026-06-14 at 14.19.49.jpeg" alt="Konfigurasi Cisco Router 1" width="800">
<img src="tumod p5/no 2/WhatsApp Image 2026-06-14 at 14.19.49 (1).jpeg" alt="Konfigurasi Cisco Router 2" width="800">
<img src="tumod p5/no 2/WhatsApp Image 2026-06-14 at 14.19.49 (2).jpeg" alt="Konfigurasi Cisco Router 3" width="800">

### Tugas Modul 3 — Konfigurasi MikroTik Router Jakarta
- **Konfigurasi:** Pembuatan VLAN interface 10, 20, 60, IP fisik, VRRP setup (Master untuk VLAN 20), DHCP Relay, dan default route ke FortiGate Jakarta (`10.10.101.1`).
- **Bukti Pengumpulan:**

<img src="tumod p5/no 3/WhatsApp Image 2026-06-14 at 14.23.24.jpeg" alt="Konfigurasi MikroTik JKT 1" width="800">
<img src="tumod p5/no 3/WhatsApp Image 2026-06-14 at 14.23.24 (1).jpeg" alt="Konfigurasi MikroTik JKT 2" width="800">
<img src="tumod p5/no 3/WhatsApp Image 2026-06-14 at 14.23.24 (2).jpeg" alt="Konfigurasi MikroTik JKT 3" width="800">
<img src="tumod p5/no 3/WhatsApp Image 2026-06-14 at 14.23.24 (3).jpeg" alt="Konfigurasi MikroTik JKT 4" width="800">
<img src="tumod p5/no 3/WhatsApp Image 2026-06-14 at 14.23.24 2.jpeg" alt="Konfigurasi MikroTik JKT 5" width="800">

### Tugas Modul 4 — Konfigurasi Ubuntu Server Jakarta
- **Konfigurasi:** Alokasi IP Static VLAN 60 via VRRP Gateway, instalasi `isc-dhcp-server` dan `nginx`. Konfigurasi scopes pool IP untuk VLAN 10 & VLAN 20 di `/etc/dhcp/dhcpd.conf`.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 4/WhatsApp Image 2026-06-14 at 14.31.49.jpeg" alt="Konfigurasi Ubuntu Server 1" width="800">
<img src="tumod p5/no 4/WhatsApp Image 2026-06-14 at 14.31.49 (1).jpeg" alt="Konfigurasi Ubuntu Server 2" width="800">
<img src="tumod p5/no 4/WhatsApp Image 2026-06-14 at 14.31.49 (2).jpeg" alt="Konfigurasi Ubuntu Server 3" width="800">
<img src="tumod p5/no 4/WhatsApp Image 2026-06-14 at 14.31.49 (3).jpeg" alt="Konfigurasi Ubuntu Server 4" width="800">

### Tugas Modul 5 — Konfigurasi FortiGate Jakarta
- **Konfigurasi:** Alokasi IP interfaces port1, port2, port3, default-routing ke ISP, Static-routing internal network JKT, Firewall Policy + NAT Outbound, GRE Tunnel, dan dynamic OSPF over GRE dengan static redistribution.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50.jpeg" alt="Konfigurasi FortiGate JKT 1" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (1).jpeg" alt="Konfigurasi FortiGate JKT 2" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (2).jpeg" alt="Konfigurasi FortiGate JKT 3" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (3).jpeg" alt="Konfigurasi FortiGate JKT 4" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (4).jpeg" alt="Konfigurasi FortiGate JKT 5" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (5).jpeg" alt="Konfigurasi FortiGate JKT 6" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (6).jpeg" alt="Konfigurasi FortiGate JKT 7" width="800">
<img src="tumod p5/no 5/WhatsApp Image 2026-06-14 at 16.29.50 (7).jpeg" alt="Konfigurasi FortiGate JKT 8" width="800">

### Tugas Modul 6 — Konfigurasi MikroTik ISP
- **Konfigurasi:** Alokasi WAN Link IP, integrasi internet NAT Cloud PNETLab pada ether1, setup NAT Masquerade out-interface ether1.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 6/WhatsApp Image 2026-06-14 at 16.31.24.jpeg" alt="Konfigurasi MikroTik ISP 1" width="800">
<img src="tumod p5/no 6/WhatsApp Image 2026-06-14 at 16.31.24 (1).jpeg" alt="Konfigurasi MikroTik ISP 2" width="800">

### Tugas Modul 7 — Konfigurasi Switch dan MikroTik Surabaya
- **Konfigurasi:** Inisiasi VLAN 30 & 40 pada Switch SBY, port access, trunking, sub-vlan interface di MikroTik SBY, DHCP pool server lokal untuk VLAN 30, dan default route ke FortiGate Surabaya.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 7/WhatsApp Image 2026-06-14 at 16.34.27.jpeg" alt="Konfigurasi Switch dan MikroTik SBY 1" width="800">
<img src="tumod p5/no 7/WhatsApp Image 2026-06-14 at 16.34.27 (1).jpeg" alt="Konfigurasi Switch dan MikroTik SBY 2" width="800">
<img src="tumod p5/no 7/WhatsApp Image 2026-06-14 at 16.34.27 (2).jpeg" alt="Konfigurasi Switch dan MikroTik SBY 3" width="800">

### Tugas Modul 8 — Konfigurasi FortiGate Surabaya
- **Konfigurasi:** Konfigurasi interface port1 (WAN) dan port2 (Internal Transit), setup default gateway ke ISP, static route internal Surabaya, Outbound NAT Policy, GRE Tunnel Endpoint, dan OSPF Over GRE.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 8/WhatsApp Image 2026-06-14 at 16.39.22.jpeg" alt="Konfigurasi FortiGate SBY 1" width="800">
<img src="tumod p5/no 8/WhatsApp Image 2026-06-14 at 16.39.22 (1).jpeg" alt="Konfigurasi FortiGate SBY 2" width="800">
<img src="tumod p5/no 8/WhatsApp Image 2026-06-14 at 16.39.22 (2).jpeg" alt="Konfigurasi FortiGate SBY 3" width="800">
<img src="tumod p5/no 8/WhatsApp Image 2026-06-14 at 16.39.22 (3).jpeg" alt="Konfigurasi FortiGate SBY 4" width="800">
<img src="tumod p5/no 8/WhatsApp Image 2026-06-14 at 16.39.22 (4).jpeg" alt="Konfigurasi FortiGate SBY 5" width="800">

### Tugas Modul 9 — Konfigurasi GRE Tunnel dan OSPF over GRE
- **Konfigurasi:** Point-to-Point virtual link enkapsulasi GRE antar-FortiGate, penyesuaian MTU/MSS, mapping area OSPF backbone, dan inject static route via OSPF redistribution.
- **Bukti Pengumpulan:**

<img src="tumod p5/no 9/WhatsApp Image 2026-06-15 at 14.42.56.jpeg" alt="Konfigurasi GRE & OSPF 1" width="800">
<img src="tumod p5/no 9/WhatsApp Image 2026-06-15 at 14.42.56 (1).jpeg" alt="Konfigurasi GRE & OSPF 2" width="800">
<img src="tumod p5/no 9/WhatsApp Image 2026-06-15 at 14.42.57.jpeg" alt="Konfigurasi GRE & OSPF 3" width="800">

---

## 4. Tugas Modul 10 — Pengujian Akhir & Analisis

### Hasil Pengujian Konektivitas & Layanan:
1. **DHCP Client JKT (VLAN 10)**: Berhasil mendapatkan alokasi dynamic IP dari Centralized Ubuntu Server melalui DHCP Relay.
2. **DHCP Client SBY (VLAN 30)**: Berhasil mendapatkan alokasi dynamic IP dari Local MikroTik Surabaya.
3. **Ping Internet dari Jakarta & Surabaya**: Aman / Reachable.
4. **Ping Antar Site (Cross-Site)**: Sukses terkoneksi melintasi GRE Tunnel.
5. **Akses Web Server**: Client Branch Surabaya berhasil mengakses Nginx Web Server Jakarta.
6. **Routing Table OSPF**: Seluruh subnet enterprise tersinkronisasi secara dinamis.

#### Bukti Dokumentasi Pengujian:
<img src="tumod p5/WhatsApp Image 2026-06-15 at 20.23.35.jpeg" alt="Hasil Pengujian Akhir 1" width="800">
<img src="tumod p5/WhatsApp Image 2026-06-15 at 20.23.36.jpeg" alt="Hasil Pengujian Akhir 2" width="800">
<img src="tumod p5/WhatsApp Image 2026-06-15 at 20.23.36 (1).jpeg" alt="Hasil Pengujian Akhir 3" width="800">
<img src="tumod p5/WhatsApp Image 2026-06-15 at 20.23.37.jpeg" alt="Hasil Pengujian Akhir 4" width="800">
<img src="tumod p5/WhatsApp Image 2026-06-15 at 20.23.37 (1).jpeg" alt="Hasil Pengujian Akhir 5" width="800">

### Analisis Jalur Lalu Lintas Data (Traffic Path Analysis)
- **Traffic Client JKT ke Internet**: Client $\rightarrow$ Virtual IP Gateway VRRP (Cisco/MikroTik) $\rightarrow$ FortiGate Jakarta (proses Source NAT) $\rightarrow$ MikroTik ISP $\rightarrow$ Cloud Internet.
- **Traffic Client SBY ke Internet**: Client $\rightarrow$ MikroTik Surabaya $\rightarrow$ FortiGate Surabaya (proses Source NAT) $\rightarrow$ MikroTik ISP $\rightarrow$ Cloud Internet.
- **Traffic Client SBY ke Web Server JKT**: Client Surabaya $\rightarrow$ MikroTik Surabaya $\rightarrow$ Edge FortiGate Surabaya $\rightarrow$ Dienkapsulasi ke **GRE Tunnel** melewati ISP $\rightarrow$ Didekapsulasi oleh FortiGate Jakarta $\rightarrow$ VRRP Gateway Jakarta $\rightarrow$ Ubuntu Server (VLAN 60).
- **Failover Analisis (VRRP)**: Apabila link utama pada VRRP Master (misal Cisco Router untuk VLAN 10) terputus, Backup Router (MikroTik) secara otomatis mengambil alih fungsi Virtual IP Gateway dalam hitungan detik (*miliseconds*), sehingga menjamin redundansi tinggi (*high availability*) pada sisi infrastruktur LAN HQ Jakarta.
