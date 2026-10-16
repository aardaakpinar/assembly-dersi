[Ana sayfa / İçindekiler](../README.md)

# Bölüm 17: C ile birlikte çalışmak

Gerçek dünyada assembly çoğunlukla C/C++ projelerinin içinde küçük, kritik parçalar olarak yaşar. Bu bölümde iki yönü de göreceksin.

### C'den assembly fonksiyonu çağırmak

C tarafı `rdi, rsi, ...` ile argümanları geçirir (ABI'yi Bölüm 15'te öğrendin), sonucu `rax`'ten alır. Bizim yapmamız gereken sözleşmeye uymak.

**`topla.asm`:**

```nasm
section .text
    global topla

; long topla(long a, long b)
topla:
    lea rax, [rdi + rsi]
    ret
```

**`main.c`:**

```c
#include <stdio.h>

extern long topla(long a, long b);

int main(void) {
    printf("%ld\n", topla(3, 4));
    return 0;
}
```

Derleme (artık `_start` yok, giriş noktası C'nin `main`'i, bağlama işini `gcc` yapar):

```bash
nasm -f elf64 topla.asm -o topla.o
gcc main.c topla.o -o program
./program        # 7
```

### Assembly'den C kütüphanesi çağırmak

Assembly programında `printf` kullanabilirsin. Ama bu sefer program `_start` yerine **`main`** ile başlar (libc başlangıç kodunu çalıştırır, sonra `main`'i çağırır).

```nasm
default rel                      ; RIP-göreli adresleme (PIE için şart)
extern printf

section .data
    fmt db "Sayi: %d, Metin: %s", 10, 0
    metin db "assembly", 0

section .text
    global main

main:
    push rbp                     ; main'e girişte rsp % 16 == 8; push sonrası 16'nın katı
    mov rbp, rsp

    lea rdi, [fmt]               ; 1. arg: format string
    mov esi, 42                  ; 2. arg: %d
    lea rdx, [metin]             ; 3. arg: %s
    xor eax, eax                 ; vararg çağrı: kullanılan vektör register sayısı = 0
    call printf wrt ..plt        ; PLT üzerinden çağır (PIE uyumlu)

    xor eax, eax                 ; return 0
    pop rbp
    ret
```

```bash
nasm -f elf64 selam.asm -o selam.o
gcc selam.o -o selam
./selam          # Sayi: 42, Metin: assembly
```

Dikkat edilecekler:

1. **Hizalama:** `main`'e girişte `rsp % 16 == 8`'dir (dönüş adresi yüzünden). `push rbp` onu 16'nın katı yapar, `call printf` öncesi hizalama doğru olur. Başka fonksiyon çağıracaksan her `call`'dan önce `rsp`'nin 16'nın katı olduğundan emin ol.
2. **`xor eax, eax`:** `printf` gibi değişken argümanlı (variadic) fonksiyonlar, `al`'de kullanılan `xmm` register sayısını bekler. Kayan nokta yoksa 0.
3. **`call printf wrt ..plt`:** Modern `gcc` varsayılan olarak PIE üretir. Bu yazım PIE ile uyumludur.
4. **`default rel`** ve `lea rdi, [fmt]`: Adresleri RIP-göreli üretir. `mov rdi, fmt` PIE'de **bağlama hatası** verir.

### Callee-saved kuralı

Assembly fonksiyonun `rbx`, `r12`–`r15`'i kullanıyorsa **çıkışta eski değerlerini geri koymalı**. C derleyicisi bunlara güvenir:

```nasm
fonksiyon:
    push rbx
    push r12
    ; ... rbx ve r12'yi kullan ...
    pop r12
    pop rbx
    ret
```

Push sayısı hizalamayı etkiler, çağırdığın fonksiyonlara dikkat.

### Derleyicinin ürettiği assembly'yi incele

Öğrenmenin en iyi yollarından biri: aynı işi C ile yaz, gcc'nin ne ürettiğine bak.

```bash
gcc -S -O1 -masm=intel -fno-asynchronous-unwind-tables -fno-pie -o cikti.s kaynak.c
```

Alternatif olarak tarayıcıda **godbolt.org** (Compiler Explorer) kullanabilirsin. C kodunu yaz, yanında assembly'yi canlı gör. Optimizasyon seviyelerini (`-O0`, `-O2`) karşılaştır.

### GCC satır içi assembly (inline asm)

C kodunun içine assembly gömmek mümkün. Sözdizimi tuhaftır, bir örneğe bak:

```c
#include <stdio.h>

int main(void) {
    long a = 10, b = 32, sonuc;
    __asm__ volatile (
        "mov %1, %0\n\t"
        "add %2, %0"
        : "=r"(sonuc)            // çıktı
        : "r"(a), "r"(b)         // girdiler
    );
    printf("%ld\n", sonuc);      // 42
    return 0;
}
```

GCC varsayılan olarak **AT&T sözdizimi** kullanır (kaynak, hedef sırası). Hatalara dikkat. Detaylar için GCC'nin "Extended Asm" belgesine bak.

### Alıştırmalar

1. `long carp(long a, long b)` assembly fonksiyonunu yazıp C'den çağır.
2. Bir C dizisinin toplamını bulan `long dizi_topla(long *dizi, long n)` fonksiyonunu assembly ile yaz.
3. Assembly `main`'den `printf` ile 1–10 arası sayıları yazdır.
4. godbolt.org'da `int topla(int a, int b) { return a + b; }` fonksiyonunu `-O0` ve `-O2` ile derleyip farkı incele.

---

[← Bölüm 16: Makrolar ve çok dosyalı projeler](../02-orta/16-makrolar-ve-cok-dosyali-projeler.md) · [Bölüm 18: Dosyalar ve komut satırı argümanları →](18-dosyalar-ve-komut-satiri-argumanlari.md)
