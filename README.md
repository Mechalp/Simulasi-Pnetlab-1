![PNETLab](https://img.shields.io/badge/Lab-PNETLab-blue)
![Routing](https://img.shields.io/badge/Routing-Static-success)
![Devices](https://img.shields.io/badge/Device-MikroTik%20%7C%20Cisco-orange)

Lab ini membangun konektivitas **dua jaringan /24** yang berbeda menggunakan **static routing sederhana** antara **MikroTik** dan **Router Cisco**.

✅ Target akhir:  
- **VPC1 bisa ping VPC2**
- **VPC2 bisa ping VPC1**

---

## 🖼️ Topologi Jaringan
![Topologi Jaringan](https://drive.google.com/uc?export=view&id=1zjSLjHKYvckHRVSkEvuYAoWvR9ew0L4L)

---

## 📌 IP Plan

| Device | Interface | IP / Mask | Keterangan |
|---|---|---|---|
| MikroTik | ether1 (ke VPC1) | `192.168.1.1/24` | Gateway LAN1 |
| VPC1 | eth0 | `192.168.1.2/24` | GW: `192.168.1.1` |
| MikroTik | ether2 (ke Cisco) | `10.10.10.1/24` | Link antar-router |
| Cisco | e0/0 (ke MikroTik) | `10.10.10.2/24` | Link antar-router |
| Cisco | e0/1 (ke VPC2) | `192.168.2.1/24` | Gateway LAN2 |
| VPC2 | eth0 | `192.168.2.2/24` | GW: `192.168.2.1` |

---

## 🧠 Konsep Routing (Static)

Agar LAN kiri dan kanan saling kenal, kita cukup tambahkan 1 route di masing-masing router:

- **MikroTik** tahu jalan ke `192.168.2.0/24` lewat **Cisco** (`10.10.10.2`)
- **Cisco** tahu jalan ke `192.168.1.0/24` lewat **MikroTik** (`10.10.10.1`)

---

---

## ⚙️ Konfigurasi

### 1) MikroTik (RouterOS)

📄 File: `configs/mikrotik.rsc`

```rsc
/ip address
add address=192.168.1.1/24 interface=ether1 comment="LAN1 Gateway"
add address=10.10.10.1/24 interface=ether2 comment="Link to Cisco"

/ip route
add dst-address=192.168.2.0/24 gateway=10.10.10.2 comment="Route to LAN2 via Cisco"
```

✅ Apply config:

* **Paste** langsung ke terminal MikroTik

---

### 2) Cisco Router

```cisco
enable
configure terminal

hostname R-CISCO

interface Ethernet0/0
 description LINK_TO_MIKROTIK
 ip address 10.10.10.2 255.255.255.0
 no shutdown

interface Ethernet0/1
 description LAN2_GATEWAY
 ip address 192.168.2.1 255.255.255.0
 no shutdown

ip route 192.168.1.0 255.255.255.0 10.10.10.1

end
do write 
```

---

### 3) VPC Configuration (PNETLab VPC)

#### VPC1

```bash
ip 192.168.1.2 255.255.255.0 192.168.1.1
save
show ip
```

#### VPC2

```bash
ip 192.168.2.2 255.255.255.0 192.168.2.1
save
show ip
```

---

## ✅ Testing / Verifikasi

### Ping dari VPC1 ➜ VPC2

```bash
ping 192.168.2.2
```

### Ping dari VPC2 ➜ VPC1

```bash
ping 192.168.1.2
```

Kalau sudah benar, hasilnya akan **reply** dua arah.

---

## 🛠️ Troubleshooting Cepat

### Cek hop-by-hop

Dari **VPC1**:

1. `ping 192.168.1.1` (gateway MikroTik)
2. `ping 10.10.10.2` (interface Cisco)
3. `ping 192.168.2.2` (VPC2)

Dari **VPC2**:

1. `ping 192.168.2.1` (gateway Cisco)
2. `ping 10.10.10.1` (interface MikroTik)
3. `ping 192.168.1.2` (VPC1)

### Cek routing

* **Cisco**

  * `show ip interface brief`
  * `show ip route`
* **MikroTik**

  * `/ip address print`
  * `/ip route print`

---

## 📜 Lisensi

Bebas dipakai untuk pembelajaran / lab. Kalau mau, tambahkan `LICENSE` (MIT) di repo kamu.

---

✨ Selesai. Selamat ngelab!

```

Kalau kamu mau, aku bisa sekalian buatin juga:
- file `mikrotik.rsc` + `cisco.cfg` versi final (rapi + komentar lengkap),
- dan versi README yang pakai **gambar lokal** (images/topology.png) supaya pasti tampil tanpa bergantung Google Drive.
```
