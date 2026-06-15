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
![Topologi Jaringan Enterprise](images/topologi-p5.jpeg)

---
