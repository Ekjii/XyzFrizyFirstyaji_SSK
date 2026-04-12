#R4 - Web Attacks

Nama: Xyz Frizy Firstyaji
NRP: 5024221073

## Tujuan

Untuk memahami konsep SQL Injection serta bagaimana celah keamanan dapat terjadi akibat penggunaan query SQL yang tidak aman.

Percobaan ini bertujuan untuk melihat bagaimana input dari user dapat dimanipulasi untuk mengubah logika query, serta memahami cara pencegahan SQL Injection menggunakan teknik yang lebih aman seperti prepared statement.

## Implementasi

Pada percobaan ini dilakukan analisis terhadap berbagai bentuk query SQL yang rentan terhadap SQL Injection.

Setiap kasus dianalisis untuk melihat apakah terdapat celah injeksi, bagaimana cara eksploitasinya, serta bagaimana mekanisme pencegahannya.

Fokus utama adalah memahami bagaimana input user dapat mengubah struktur query SQL, serta bagaimana praktik keamanan dapat mencegah serangan tersebut.

### W4.1

Ya, program ini masih memiliki kerentanan SQL Injection.

Meskipun menggunakan fungsi SHA2 untuk melakukan hashing, input user tetap dimasukkan langsung ke dalam query SQL tanpa sanitasi.

Seorang attacker dapat menutup tanda kutip dan menambahkan kondisi tambahan, misalnya:

eid: ' OR '1'='1
passwd: ' OR '1'='1

Sehingga query menjadi:

SELECT * FROM employee
WHERE eid=SHA2('' OR '1'='1',256) and password=SHA2('' OR '1'='1',256)

Hal ini dapat menyebabkan query selalu bernilai true dan memungkinkan bypass autentikasi.

Kesimpulan: hashing tidak mencegah SQL Injection jika query tetap dibentuk secara langsung dari input user.

### W4.2

Program ini lebih aman dibandingkan sebelumnya karena proses hashing dilakukan sebelum query dibentuk.

Input user tidak langsung dimasukkan ke dalam query, melainkan sudah diubah menjadi hash terlebih dahulu.

Namun, jika input awal tetap tidak divalidasi dengan baik, masih ada potensi penyalahgunaan tergantung implementasi.

Secara umum, pendekatan ini lebih aman dibandingkan W4.1, tetapi belum sepenuhnya aman tanpa penggunaan prepared statement.

### W4.3

Ya, SQL Injection masih dapat dilakukan.

Line break dalam query tidak mempengaruhi cara SQL parser membaca query.

Attacker tetap dapat menyisipkan payload seperti:

' OR '1'='1

Sehingga kondisi WHERE menjadi selalu true.

Kesimpulan: format penulisan query (termasuk line break) tidak mempengaruhi kerentanan SQL Injection.

### W4.4

Attacker dapat memanipulasi input $name untuk menyisipkan query tambahan.

Contoh payload:

name: abc', 'EID6000', 'pass', 999999) --

Sehingga query menjadi:

INSERT INTO employee (...)
VALUES ('abc', 'EID6000', 'pass', 999999) -- ', 'EID6000', ...

Dengan demikian attacker dapat mengubah nilai salary menjadi lebih tinggi dari 80000.

### W4.5

Attacker dapat menyisipkan payload pada field $name atau $eid untuk mengubah query.

Contoh:

name: attacker', Salary=1 --

Sehingga query menjadi:

UPDATE employee
SET name='attacker', Salary=1 --', password='...'
WHERE ...

Selain itu, attacker juga dapat mengganti password dengan nilai yang diketahui.

Hal ini memungkinkan attacker untuk mengubah data penting sekaligus mengambil alih akun.

### W4.6

Attacker dapat menyisipkan multiple query dengan menggunakan tanda titik koma (;).

Contoh:

eid: ' ; DROP TABLE employee; --

Sehingga query menjadi:

SELECT * FROM employee
WHERE eid='' ; DROP TABLE employee; -- and password='...'

Jika database mengizinkan multiple statements, maka attacker dapat menjalankan perintah berbahaya seperti menghapus tabel.

### W4.7

Ya, jika database mengizinkan multiple statements, maka SQL Injection dapat digunakan untuk menjalankan perintah tambahan.

Namun, pada banyak sistem modern, fitur ini dinonaktifkan secara default untuk mencegah serangan seperti ini.

Kesimpulan: secara teori bisa, tetapi tergantung konfigurasi database.

### W4.8

Tidak, filtering di client side tidak aman.

Hal ini karena attacker dapat dengan mudah melewati filter tersebut menggunakan tools seperti curl atau mengirim request langsung ke server.

Validasi harus dilakukan di server side.

Kesimpulan: client-side filtering tidak cukup untuk mencegah SQL Injection.

### W4.9

Kode ini lebih aman karena menggunakan prepared statement.

Namun, bagian eid masih dimasukkan langsung ke dalam query tanpa parameter binding.

Sehingga masih terdapat potensi SQL Injection pada bagian eid.

Kesimpulan: prepared statement harus digunakan untuk semua input user, bukan hanya sebagian.

### W4.10

Menggunakan prepared statement:

$stmt = $conn->prepare("UPDATE employee SET password=? WHERE eid=? and password=?");
$stmt->bind_param("sss", $newpwd, $eid, $oldpwd);
$stmt->execute();


### W4.11

Meskipun user tidak memiliki akses langsung ke database, mereka dapat mengirim input melalui web application.

Input tersebut akan diproses oleh server dan dimasukkan ke dalam query SQL.

Jika query tidak aman, maka attacker dapat menyisipkan kode SQL melalui input tersebut.

Flow:
User -> Web Application -> SQL Query -> Database

Kesimpulan: web application menjadi perantara yang memungkinkan SQL Injection.

### W4.12

Prepared statement dan execve memiliki konsep yang mirip, yaitu memisahkan data dan command.

Pada prepared statement:
- Query SQL sudah ditentukan terlebih dahulu
- Input user hanya dianggap sebagai data

Pada execve:
- Program dan argumen dipisahkan
- Tidak ada parsing string seperti pada system()

Kesimpulan:
Keduanya mencegah injection dengan cara memisahkan command dan input user.


