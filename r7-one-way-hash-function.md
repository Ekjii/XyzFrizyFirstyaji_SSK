# R7 - Kriptografi (Hash)

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073  

## Tujuan

Untuk memahami konsep one-way hash function serta bagaimana fungsi hash digunakan dalam keamanan sistem, khususnya dalam penyimpanan password dan verifikasi integritas data.

Percobaan ini bertujuan untuk memahami sifat-sifat hash, kelemahannya, serta bagaimana mekanisme keamanan seperti salt dan multiple hashing dapat meningkatkan keamanan.

## Implementasi

Pada percobaan ini dilakukan analisis terhadap berbagai konsep dalam fungsi hash satu arah.

Setiap kasus dianalisis untuk memahami bagaimana hash bekerja, potensi serangan terhadap hash, serta bagaimana teknik tertentu digunakan untuk meningkatkan keamanan sistem.

### C2.1

Fungsi f(x) = x mod 10000 bukan hash yang baik karena menghasilkan output yang sangat terbatas.

Banyak input berbeda akan menghasilkan output yang sama (collision tinggi), sehingga tidak aman.

Kesimpulan: hash harus memiliki distribusi yang luas dan sulit ditebak.

### C2.2

Angka 2 pada SHA-2 menunjukkan versi algoritma dari keluarga SHA (generasi kedua).

Angka 256 pada SHA-256 menunjukkan panjang output hash, yaitu 256 bit.

### C2.3

PHP: md5(), sha1(), hash()  
SQL: MD5(), SHA1(), SHA2()  
Python: hashlib (md5(), sha1(), sha256())

### C2.4

Setuju.

Salt membuat setiap hash menjadi unik meskipun password sama.

Hal ini mencegah penggunaan rainbow table dan memperlambat brute force attack.

### C2.5

Tidak perlu menyembunyikan salt.

Salt tidak berfungsi sebagai secret, melainkan untuk memastikan keunikan hash.

Menyimpan salt secara plaintext tetap aman selama password di-hash dengan benar.

### C2.6

Tidak setuju.

Jika hash dikirim dari client, maka hash tersebut menjadi “password baru”.

Attacker dapat melakukan replay attack dengan menggunakan hash tersebut tanpa mengetahui password asli.

Kesimpulan: hashing harus dilakukan di server, bukan hanya di client.

### C2.7

Multiple hashing dilakukan untuk memperlambat proses brute force.

Semakin banyak iterasi, semakin lama waktu yang dibutuhkan attacker untuk mencoba password.

Kesimpulan: meningkatkan computational cost untuk attacker.

### C2.8

Sebagai pengacara, dapat berargumen bahwa hash yang dipublikasikan sebelum tahun 2004 tetap valid sebagai bukti timestamp.

Collision pada MD5 memang ada, tetapi tidak mudah untuk membuat collision yang bermakna terhadap karya tertentu secara spesifik.

### C2.9

Hal ini memungkinkan length extension attack.

Dengan mengetahui hash(K || M), attacker dapat menghitung hash(K || M || padding || X) tanpa mengetahui K.

### C2.10

Tidak bisa.

Karena K berada di belakang, maka length extension attack tidak dapat dilakukan.

### C2.11

Ya, diperlukan untuk mengetahui bagaimana padding dilakukan.

Tanpa mengetahui panjang K, sulit untuk membentuk padding yang tepat.

### C2.12

1. Padding terdiri dari bit 1 diikuti dengan nol dan panjang total pesan.

2. String N harus berisi:
- padding dari K:M
- diikuti dengan "extra content"

### C2.13

Bob dapat meminta Alice untuk menghitung hash dari K yang digabung dengan nilai acak (challenge).

Jika hasil hash sesuai, maka Alice terbukti mengetahui K tanpa mengungkapkannya.

### C2.14

Gunakan hash chain.

Password berikutnya adalah hash dari password sebelumnya.

Server hanya menyimpan hash terakhir, dan setiap kali password digunakan, nilai diperbarui.

### C2.15

Gunakan komitmen hash.

Publish hash dari seluruh prediksi di awal.

Setiap hari, buka satu bagian prediksi.

Orang lain dapat memverifikasi dengan mencocokkan hash.

### C2.16

Tidak bisa.

Collision attack tidak memungkinkan untuk mencari pasangan spesifik untuk pesan tertentu.

### C2.17

Ya.

Jika ditemukan satu collision, dapat digunakan untuk membuat collision lain dengan teknik tertentu seperti concatenation.
