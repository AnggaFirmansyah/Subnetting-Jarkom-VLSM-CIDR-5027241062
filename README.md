
Nama : Angga Firmansyah
NRP : 5027241062
Prefix IP : 10.102.0.0

# Topologi

<img width="617" height="662" alt="image" src="https://github.com/user-attachments/assets/a90ed7b2-6ba9-4961-a927-43e19879f1b3" />

# VLSM

| Ruang / Unit       | Jumlah Host Aktif |       Prefix IP | Subnet Mask     | IP Broadcast | First IP address | Last IP address |
| ------------------ | ----------------: | --------------: | --------------- | ------------ | ---------------- | --------------- |
| Sekretariat        |               380 |   10.102.0.0/23 | 255.255.254.0   | 10.102.1.255 | 10.102.0.1       | 10.102.1.254    |
| Kurikulum          |               220 |   10.102.2.0/24 | 255.255.255.0   | 10.102.2.255 | 10.102.2.1       | 10.102.2.254    |
| Guru & Tendik      |                95 |   10.102.3.0/25 | 255.255.255.128 | 10.102.3.127 | 10.102.3.1       | 10.102.3.126    |
| Sarana & Prasarana |                45 | 10.102.3.128/26 | 255.255.255.192 | 10.102.3.191 | 10.102.3.129     | 10.102.3.190    |
| Pengawas (HQ)      |                18 | 10.102.3.192/27 | 255.255.255.224 | 10.102.3.223 | 10.102.3.193     | 10.102.3.222    |
| Pengawas (Cabang)  |                18 | 10.102.3.224/27 | 255.255.255.224 | 10.102.3.255 | 10.102.3.225     | 10.102.3.254    |
| Server & Admin     |                 6 |   10.102.4.0/29 | 255.255.255.248 | 10.102.4.7   | 10.102.4.1       | 10.102.4.6      |
| Link Antar-Router  |                 2 |   10.102.4.8/30 | 255.255.255.252 | 10.102.4.11  | 10.102.4.9       | 10.102.4.10     |

# CIDR

| Lokasi         | Subnet Digabung                                                               |   Supernet (CIDR) | Mask            | Range Alamat                | Gateway (saran) |
| -------------- | ----------------------------------------------------------------------------- | ----------------: | --------------- | --------------------------- | --------------- |
| Kantor Pusat   | 10.102.0.0/23, 10.102.2.0/24, 10.102.3.0/25, 10.102.3.128/26, 10.102.3.192/27 | **10.102.0.0/22** | 255.255.252.0   | 10.102.0.0 – 10.102.3.255   | 10.102.0.1      |
| Kantor Cabang  | 10.102.3.224/27                                                               |   10.102.3.224/27 | 255.255.255.224 | 10.102.3.224 – 10.102.3.255 | 10.102.3.225    |
| Server & Admin | 10.102.4.0/29                                                                 |     10.102.4.0/29 | 255.255.255.248 | 10.102.4.0 – 10.102.4.7     | 10.102.4.1      |
| Link WAN       | 10.102.4.8/30                                                                 |     10.102.4.8/30 | 255.255.255.252 | 10.102.4.8 – 10.102.4.11    | 10.102.4.9      |


# Router

```bash
enable
configure terminal
hostname R-HQ

! Physical Fa0/0 will be trunked to SW-HQ
interface FastEthernet0/0
 no ip address
 no shutdown
exit

! Sekretariat VLAN 10
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.102.0.1 255.255.254.0
 no shutdown
exit

! Kurikulum VLAN 20
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.102.2.1 255.255.255.0
 no shutdown
exit

! Guru & Tendik VLAN 30
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.102.3.1 255.255.255.128
 no shutdown
exit

! Sarpras VLAN 40
interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.102.3.129 255.255.255.192
 no shutdown
exit

! Pengawas HQ VLAN 50
interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 10.102.3.193 255.255.255.224
 no shutdown
exit

! Server & Admin VLAN 60 (or use separate switch)
interface FastEthernet0/0.60
 encapsulation dot1Q 60
 ip address 10.102.4.1 255.255.255.248
 no shutdown
exit

! WAN link to branch on Fa0/1
interface FastEthernet0/1
 description Link-to-R-Cabang
 ip address 10.102.4.9 255.255.255.252
 no shutdown
exit

! Static route to the branch subnet (Pengawas Cabang)
ip route 10.102.3.224 255.255.255.224 10.102.4.10
end
write memory
```
```
enable
configure terminal
hostname R-Cabang

! LAN at branch (Pengawas Cabang)
interface FastEthernet0/0
 description To-SW-Cabang
 ip address 10.102.3.225 255.255.255.224
 no shutdown
exit

! WAN link to HQ
interface FastEthernet0/1
 description Link-to-R-HQ
 ip address 10.102.4.10 255.255.255.252
 no shutdown
exit

! Static route back to HQ networks (you can add single summary)
ip route 10.102.0.0 255.255.252.0 10.102.4.9
end
write memory
```

# ICPC

```
| Perangkat             | IP Address   | Subnet Mask     | Default Gateway | Keterangan      |
| --------------------- | ------------ | --------------- | --------------- | --------------- |
| PC0                   | 10.102.0.10  | 255.255.254.0   | 10.102.0.1      | Sekretariat     |
| PC1                   | 10.102.2.10  | 255.255.255.0   | 10.102.2.1      | Kurikulum       |
| PC2                   | 10.102.3.10  | 255.255.255.128 | 10.102.3.1      | Guru & Tendik   |
| PC3                   | 10.102.3.130 | 255.255.255.192 | 10.102.3.129    | Sarpras         |
| PC4 (Pengawas HQ)     | 10.102.3.194 | 255.255.255.224 | 10.102.3.193    | Pengawas HQ     |
| PC5 (Pengawas Cabang) | 10.102.3.226 | 255.255.255.224 | 10.102.3.225    | Pengawas Cabang |
| Server0               | 10.102.4.2   | 255.255.255.248 | 10.102.4.1      | Server & Admin  |
```
