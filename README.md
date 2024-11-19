# Praktikum 13 - Static Routing dengan Konfigurasi VLAN

![img](./assets/Screenshot%20from%202024-11-19%2020-31-55.png)

PT Mencari Cinta saat ini memiliki 2 LAN yang masing2 LAN nya memiliki VLAN tersendiri. Pada LAN kiri terdapat VLAN departemen HR dan departemen sales. Pada LAN kanan terdapat VLAN departemen product dan departemen developer.

PT Mencari Cinta melakukan permintaan untuk yang LAN kiri menggunakan DHCP Server untuk layanan DHCP dan yang LAN kanan menggunakan DHCP Router untuk layanan DHCP.

Sebagai network engineer, bantu PT mencari cinta untuk melakukan konfigurasi jaringan supaya masing2 VLAN bisa saling berkomunikasi satu sama lain

Untuk konfigurasi packet tracer awal dapat diunduh pada file [Latihan VLAN 2.pkt](./pkt/latihan_vlan_2.pkt) atau bisa dilihat pada grup whatsapp.

## Konfigurasi VLAN pada LAN 1

Masuk ke CLI Switch LAN 1 lalu ketikkan perintah berikut

```zsh
enable
conf t
vlan 10
name HR
exit

vlan 20
name sales
exit

vlan 30
name DHCP
exit
```

Setelah itu, masukkan VLAN HR ke port Fa2/1, VLAN sales ke port Fa1/1 dan VLAN DHCP ke port Fa3/1

```zsh
int fa2/1
switchport mode access
switchport access vlan 10
ex

int fa1/1
switchport mode access
switchport access vlan 20
ex

int fa3/1
switchport mode access
switchport access vlan 30
ex
```

Lalu kita buat network yang mengarah ke router menjadi trunk dengan perintah berikut

```zsh
int g0/1
switchport mode trunk
```

Setelah itu, kita lakukan konfigurasi inter-VLAN routing. Masuk ke CLI router kiri, lalu ketikkan perintah berikut untuk menyalakan port gig2/0
```zsh
enable
conf t
int gig2/0
no sh
exit
```

Setelah itu, kita buat sub interface untuk VLAN 10, 20 dan 30
```zsh
int gig2/0.10
encapsulation dot1Q 10
ip add 192.168.10.1 255.255.255.0
exit

int gig2/0.20
encapsulation dot1Q 20
ip add 192.168.20.1 255.255.255.0
exit

int gig2/0.30
encapsulation dot1Q 30
ip add 192.168.30.1 255.255.255.0
exit
```

Setelah itu, kita harus pastikan interface G2/0 beserta sub interface yang sudah dibuat berada dalam status up dengan perintah pada global (bukan conf t)

```zsh
sh ip int br
```

![img](./assets/Screenshot%20from%202024-11-19%2020-47-27.png)

Pada server DHCP di LAN 1, lakukan konfigurasi ip secara statis dengan info berikut

Field | Value | 
--- | --- |
IPv4 Address | 192.168.30.2 | 
Subnet Mask | 255.255.255.0 | 
Default Gateway | 192.168.30.1 | 

Lalu masuk ke services DHCP, nyalakan service DHCP dan tambahkan 2 pool dengan detail informasi berikut

- Pool 1 (HR Pool)

    Field | Value | 
    --- | --- |
    Pool Name | HR Pool | 
    Default Gateway | 192.168.10.1 |
    Start IP Address | 192.168.10.2 | 

- Pool 2 (Sales Pool)

    Field | Value | 
    --- | --- |
    Pool Name | Sales Pool | 
    Default Gateway | 192.168.20.1 |
    Start IP Address | 192.168.20.2 | 

Setelah itu, masuk ke router lagi dan masuk ke sub interface VLAN 10

```zsh
int g2/0.10 
ip helper-address 192.168.30.2
ex

int g2/0.20
ip helper-address 192.168.30.2
ex
```

Setalah itu, coba lakuakn request alamat secara DHCP pada VLAN HR dan VLAN Sales

## Konfigurasi VLAN pada LAN 2

Masuk ke CLI Switch LAN 2 lalu ketikkan perintah berikut

```zsh
enable
conf t
vlan 10
name Product
exit

vlan 20
name Developer
exit
```

Setelah itu, masukkan VLAN Product ke port Fa1/1, VLAN Developer ke port Fa2/1

```zsh
int fa1/1
switchport mode access
switchport access vlan 10
ex

int fa2/1
switchport mode access
switchport access vlan 20
ex
```

Lalu kita buat network yang mengarah ke router menjadi trunk dengan perintah berikut

```zsh
int g0/1
switchport mode trunk
```

Setelah itu, kita lakukan konfigurasi inter-VLAN routing. Masuk ke CLI router kanan, lalu ketikkan perintah berikut untuk menyalakan port gig2/0
```zsh
enable
conf t
int gig2/0
no sh
exit
```

Setelah itu, kita buat sub interface untuk VLAN 10 dan 20
```zsh
int gig2/0.10
encapsulation dot1Q 10
ip add 192.68.10.1 255.255.255.0
exit

int gig2/0.20
encapsulation dot1Q 20
ip add 192.68.20.1 255.255.255.0
exit
```

Lalu, kita lakukan konfigurasi DHCP pada router dengan perintah berikut

```zsh
ip dhcp pool ProductPool
network 192.68.10.0 255.255.255.0
default-router 192.68.10.1
dns-server 8.8.8.8
exit

ip dhcp pool DeveloperPool
network 192.68.20.0 255.255.255.0
default-router 192.68.20.1
dns-server 8.8.8.8
exit
```

Untuk mengecek pool DHCP yang telah dibuat, kamu bisa menjalankan perintah berikut pada global (bukan conf t)

```zsh
sh ip dhcp pool
```

Apabila benar, maka harusnya muncul output seperti gambar dibawah ini

![img](./assets/Screenshot%20from%202024-11-19%2021-18-08.png)

Setelah itu, buka PC pada salah satu LAN kanan, lalu seharusnya kalian bisa melihat bahwa ip setiap end device sudah otomatis di set ke DHCP apabila benar.

# Routing antar LAN
#### **1. Konfigurasi Router 0**

Masuk ke Router 0 dan lakukan konfigurasi pada interface Serial0/0:

```zsh
Router0> enable
Router0# configure terminal
Router0(config)# interface Serial0/0
Router0(config-if)# ip address 200.200.10.1 255.255.255.0
Router0(config-if)# no shutdown
Router0(config-if)# exit
```

Setelah konfigurasi IP address selesai, kita perlu melakukan static routing pada Router 0:

```zsh
Router0(config)# ip route 194.68.10.0 255.255.255.0 200.200.10.2
Router0(config)# ip route 192.68.20.0 255.255.255.0 200.200.10.2
```

Pastikan semua konfigurasi telah dilakukan dengan benar dan periksa tabel routing untuk memastikan rute telah ditambahkan:

```zsh
Router0# show ip route
```
![img](./assets/show-iproute0.png)

#### **2. Konfigurasi Router 1**

Selanjutnya, masuk ke Router 1 dan lakukan konfigurasi pada interface Serial0/0:

```zsh
Router1> enable
Router1# configure terminal
Router1(config)# interface Serial0/0
Router1(config-if)# ip address 200.200.10.2 255.255.255.0
Router1(config-if)# no shutdown
Router1(config-if)# exit
```

Setelah mengatur IP address, kita perlu menambahkan static routing pada Router 1:

```zsh
Router1(config)# ip route 192.168.10.0 255.255.255.0 200.200.10.1
Router1(config)# ip route 192.168.20.0 255.255.255.0 200.200.10.1
Router1(config)# ip route 192.168.30.0 255.255.255.0 200.200.10.1
```

Sama seperti sebelumnya, pastikan semua konfigurasi telah dilakukan dengan benar dan periksa tabel routing:

```zsh
Router1# show ip route
```
![img](./assets/show-iproute1.png)

### Penjelasan Konfigurasi

- **Interface Serial**: Interface Serial digunakan untuk menghubungkan kedua router secara langsung.
  
- **IP Addressing**: Setiap router diberikan alamat IP yang unik dalam subnet yang sama untuk memastikan komunikasi yang efektif.

- **Static Routing**: Routing statis ditetapkan untuk mendefinisikan jalur ke jaringan lain secara manual, yang berguna dalam jaringan kecil di mana perubahan topologi jarang terjadi.

### Verifikasi Koneksi

Setelah semua konfigurasi selesai, lakukan verifikasi koneksi antar PC atau perangkat lain yang terhubung ke masing-masing router dengan menggunakan perintah `ping`. Pastikan semua perangkat dapat saling berkomunikasi meskipun berada di jaringan yang berbeda. Contoh: lakukan pengujian dari VLAN HRD (melalui PC2) ke VLAN product (menghubungi PC4 dengan ip 192.68.10.2).

```zsh
PC> ping 192.68.10.2
```
![img](./assets/testing.png)
