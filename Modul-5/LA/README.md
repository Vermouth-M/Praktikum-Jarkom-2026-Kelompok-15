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
