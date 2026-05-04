# R6 - Network Defense (Firewall)

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073  

## Tujuan

Untuk memahami konsep dasar firewall menggunakan netfilter, serta bagaimana mekanisme filtering packet bekerja dalam sistem operasi Linux.

Percobaan ini bertujuan untuk memahami bagaimana packet dapat dikontrol, difilter, dan dimanipulasi menggunakan berbagai aturan firewall, serta memahami peran penting firewall dalam keamanan jaringan.

## Implementasi

Pada percobaan ini dilakukan analisis terhadap konsep netfilter, iptables, serta berbagai teknik dalam melakukan filtering packet.

Setiap kasus dianalisis untuk memahami bagaimana aturan firewall bekerja, bagaimana packet diproses dalam sistem, serta bagaimana mekanisme keamanan diterapkan untuk melindungi jaringan.

### N7.1

Netfilter adalah framework dalam kernel Linux yang digunakan untuk melakukan packet filtering, network address translation (NAT), dan packet mangling.

Keuntungan:
- Dapat mengontrol traffic jaringan
- Mendukung firewall dan NAT
- Fleksibel dan dapat dikustomisasi
- Terintegrasi langsung dengan kernel sehingga efisien

### N7.2

Lima netfilter hooks untuk IPv4:
- PREROUTING: dipanggil sebelum routing decision
- INPUT: untuk packet yang ditujukan ke host
- FORWARD: untuk packet yang diteruskan
- OUTPUT: untuk packet yang dibuat oleh host
- POSTROUTING: setelah routing decision

### N7.3

1. Packet dari S ke D (di D):
PREROUTING → INPUT

2. Packet dari S ke D (di router R):
PREROUTING → FORWARD → POSTROUTING

3. Packet dibuat di S:
OUTPUT → POSTROUTING

### N7.4

Netfilter hooks berada di kernel space, sehingga untuk mengaksesnya diperlukan kernel module.

Program user biasa tidak dapat langsung berinteraksi dengan kernel untuk alasan keamanan.

### N7.5

filterHook.hook = block;

nf_register_hook(&filterHook);

nf_unregister_hook(&filterHook);

module_init(setUpFilter);

module_exit(removeFilter);

iph = ip_hdr(skb);

if(des_ip==in_aton("10.0.2.5") &&
ntohs(des_port)== 80) {
return NF_DROP;
}

### N7.6

- Membatasi packet masuk: INPUT chain
- Membatasi packet keluar: OUTPUT chain

### N7.7

Ya, netfilter dapat digunakan untuk memodifikasi packet.

Contoh:
- Network Address Translation (NAT)
- Packet mangling
- Load balancing
- Packet logging

### N7.8

Urutan:
F2 → F3 → F1

Penjelasan:
Packet dibuat di host → melalui LOCAL_OUT (F2 dan F3), lalu POST_ROUTING (F1)

### N7.9

False.

Karena packet masih dapat diproses oleh hook lain setelahnya, sehingga belum tentu langsung diterima sepenuhnya.

### N7.10

1. Jika F2 return NF_ACCEPT → F3 tetap dipanggil  
2. Jika F2 return NF_DROP → F3 tidak dipanggil  

### N7.11

- INPUT chain → NF_INET_LOCAL_IN  
- OUTPUT chain → NF_INET_LOCAL_OUT  
- POSTROUTING chain → NF_INET_POST_ROUTING  

### N7.12

SYNPROXY bekerja dengan cara memvalidasi koneksi TCP sebelum diteruskan ke server.

Firewall akan merespon SYN request terlebih dahulu, dan hanya meneruskan koneksi jika handshake valid.

Hal ini mencegah serangan SYN flood karena server tidak langsung menerima request.

### N7.13

Stateful firewall dapat melacak status koneksi.

Contoh:
- Mengizinkan paket balasan dari koneksi yang sudah established
- Memblokir koneksi yang tidak valid

Keuntungan:
- Lebih aman
- Lebih efisien
- Mengurangi rule yang kompleks

### N7.14

ufw bukan firewall sebenarnya, melainkan interface untuk mengelola iptables.

Firewall sebenarnya tetap berjalan di level kernel melalui netfilter.

### N7.15

iptables -A INPUT -s 192.168.10.0/24 -j ACCEPT

### N7.16

iptables -A INPUT -d 10.0.20.5 -p tcp --dport 22 -j DROP  
iptables -A INPUT -d 10.0.20.5 -p tcp --dport 23 -j DROP  
iptables -A INPUT -d 10.0.20.5 -p tcp --dport 80 -j DROP  
iptables -A INPUT -d 10.0.20.5 -p tcp --dport 443 -j DROP  

### N7.17

iptables -t nat -A PREROUTING -p udp --dport 9000 -m statistic --mode random --probability 0.25 -j DNAT --to-destination A:9000  
iptables -t nat -A PREROUTING -p udp --dport 9000 -m statistic --mode random --probability 0.33 -j DNAT --to-destination B:9000  
iptables -t nat -A PREROUTING -p udp --dport 9000 -m statistic --mode random --probability 0.5 -j DNAT --to-destination C:9000  
iptables -t nat -A PREROUTING -p udp --dport 9000 -j DNAT --to-destination D:9000  

### N7.18

Connection pada UDP dan ICMP berarti tracking berdasarkan flow atau request-response.

Meskipun tidak connection-oriented, sistem tetap mencatat state sementara.

### N7.19

- TCP: 431752 detik  
- UDP: 1 detik  
- ICMP: 29 detik  

Kesimpulan: waktu timeout tergantung jenis protokol dan state koneksi.
