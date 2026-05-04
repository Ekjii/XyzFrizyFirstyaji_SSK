# R5 - Network Attacks

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073  

## Tujuan

Untuk memahami konsep dasar packet sniffing dan spoofing, serta bagaimana serangan jaringan dapat dilakukan pada berbagai layer seperti IP dan TCP.

Percobaan ini bertujuan untuk memahami bagaimana packet dapat ditangkap dari jaringan, bagaimana manipulasi packet dapat dilakukan, serta memahami implikasi keamanan dari serangan tersebut.

## Implementasi

Pada percobaan ini dilakukan analisis terhadap berbagai teknik dalam packet sniffing dan spoofing.

Setiap kasus dianalisis untuk memahami cara kerja serangan, penyebab terjadinya, serta dampaknya terhadap sistem jaringan.

Fokus utama adalah memahami bagaimana packet ditransmisikan dalam jaringan, serta bagaimana attacker dapat mengeksploitasi mekanisme tersebut.

### N4.1

Error "NULL: No such device" terjadi karena interface jaringan yang digunakan ("eth1") tidak tersedia pada sistem.

Nama interface jaringan dapat berbeda-beda tergantung konfigurasi sistem, seperti eth0, ens33, atau wlan0.

Jika interface yang diberikan tidak valid, maka pcap_open_live() akan mengembalikan nilai NULL.

Kesimpulan: error terjadi karena penggunaan nama interface yang tidak sesuai dengan sistem.

### N4.2

Program hanya menangkap packet yang masuk atau keluar dari komputer sendiri karena promiscuous mode tidak diaktifkan.

Parameter ketiga pada pcap_open_live bernilai 0, yang berarti non-promiscuous mode.

Dalam mode ini, network interface hanya menerima packet yang ditujukan untuk dirinya sendiri.

Kesimpulan: untuk menangkap semua packet dalam jaringan, promiscuous mode harus diaktifkan.

### N4.3

Fungsi pcap_open_live() memerlukan root privilege karena digunakan untuk mengakses network interface secara langsung.

Jika program dijalankan tanpa privilege yang cukup, maka proses sniffing akan gagal atau terbatas.

Kesimpulan: akses terhadap network interface membutuhkan hak akses khusus.

### N4.4

Untuk menguji apakah pcap_setfilter() memerlukan root privilege, dapat dilakukan percobaan dengan menjalankan program sebagai user biasa dan sebagai root.

Jika fungsi gagal saat dijalankan tanpa root tetapi berhasil saat menggunakan root, maka dapat disimpulkan bahwa fungsi tersebut memerlukan privilege.

Kesimpulan: hasil eksperimen menentukan apakah filter dilakukan di level yang membutuhkan akses khusus.

### N4.5

Terdapat dua pendekatan dalam melakukan filtering packet.

Pendekatan pertama adalah menangkap semua packet, kemudian melakukan filtering di level aplikasi.

Pendekatan kedua adalah menggunakan pcap_setfilter() untuk melakukan filtering langsung di kernel.

Pendekatan pertama lebih fleksibel, tetapi membutuhkan lebih banyak resource.

Pendekatan kedua lebih efisien karena hanya packet yang relevan yang diteruskan ke aplikasi.

Kesimpulan: filtering di kernel lebih efisien dibandingkan filtering di aplikasi.

### N4.6

Wireshark tetap dapat melakukan sniffing meskipun dijalankan sebagai user biasa karena menggunakan proses tambahan dengan privilege lebih tinggi.

Wireshark menjalankan program dumpcap sebagai child process yang memiliki hak akses untuk melakukan packet capture.

Kesimpulan: pemisahan proses memungkinkan aplikasi tetap aman namun tetap memiliki kemampuan sniffing.

### N4.7

Opsi -Z pada tcpdump digunakan untuk menurunkan privilege setelah proses awal selesai.

Program menggunakan root hanya untuk membuka akses ke network interface, kemudian menurunkan privilege untuk meningkatkan keamanan.

Kesimpulan: prinsip least privilege digunakan untuk mengurangi risiko keamanan.

### N4.8

Prinsip least privilege dapat diterapkan dengan memisahkan proses yang membutuhkan privilege tinggi dan proses yang tidak.

Packet capture dapat dilakukan oleh proses kecil dengan privilege tinggi, sedangkan analisis dilakukan oleh proses biasa.

Kesimpulan: pendekatan ini dapat mengurangi attack surface.

### N4.9

Pada Big-Endian, data disimpan dengan byte paling signifikan terlebih dahulu:

0x1000 → AA  
0x1001 → BB  
0x1002 → CC  
0x1003 → DD  

Pada Little-Endian, data disimpan dengan byte paling rendah terlebih dahulu:

0x1000 → DD  
0x1001 → CC  
0x1002 → BB  
0x1003 → AA  

Kesimpulan: perbedaan endian mempengaruhi cara data disimpan di memori.

### N4.10

Jika payload berukuran 255 byte (0x000000FF) dan tidak dikonversi ke host order pada mesin Little-Endian, maka nilai akan terbaca sebagai 0xFF000000.

Hal ini menyebabkan alokasi memori sebesar 4278190080 byte.

Kesimpulan: kesalahan konversi byte order dapat menyebabkan alokasi memori yang sangat besar dan berpotensi menimbulkan crash.

### N4.11

Pada mesin Little-Endian, fungsi htonl() dan ntohl() akan mengubah urutan byte sehingga nilai 255 menjadi 4278190080.

Pada mesin Big-Endian, tidak terjadi perubahan sehingga nilai tetap 255.

Kesimpulan: fungsi konversi byte order menghasilkan output yang berbeda tergantung arsitektur.

### N4.12

Packet spoofing dengan tujuan IP lokal tidak terlihat karena sistem memerlukan ARP untuk menentukan MAC address tujuan.

Jika ARP tidak berhasil atau tidak sesuai, packet tidak akan dikirim ke jaringan.

Sedangkan untuk IP eksternal, packet dikirim ke gateway sehingga tetap dapat diamati.

Kesimpulan: mekanisme ARP mempengaruhi apakah packet spoofing dapat terlihat di jaringan
