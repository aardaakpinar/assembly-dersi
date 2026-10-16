[Ana sayfa / İçindekiler](../README.md)

# Bölüm 19: Kayan noktalı sayılar ve SIMD

### Tam sayılar ile kayan noktalı sayılar farklı dünyadır

Genel amaçlı register'lar (`rax` vb.) tamsayılar içindir. Ondalıklı sayılar (`3.14`) için işlemcinin ayrı register'ları ve komutları vardır: **`xmm0`–`xmm15`** (128 bit).

- **Tek duyarlıklı (float, 32 bit):** komutlar `...ss` ile biter (**s**calar **s**ingle)
- **Çift duyarlıklı (double, 64 bit):** komutlar `...sd` ile biter (**s**calar **d**ouble)

### Temel komutlar

```nasm
movsd  xmm0, [deger]       ; double yükle
movsd  [sonuc], xmm0       ; double yaz
movss  xmm0, [f]           ; float yükle

addsd  xmm0, xmm1          ; xmm0 += xmm1
subsd  xmm0, xmm1
mulsd  xmm0, xmm1
divsd  xmm0, xmm1
sqrtsd xmm0, xmm1          ; xmm0 = karekök(xmm1)

cvtsi2sd  xmm0, rax        ; tamsayı → double
cvttsd2si rax, xmm0        ; double → tamsayı (kesme)
comisd    xmm0, xmm1       ; karşılaştır (bayrakları ayarlar, ja/jb/je ile kullan)
```

Veri tanımlama:

```nasm
pi      dq 3.14159265358979     ; double
yarıcap dq 5.0
oran    dd 1.5                  ; float
```

### Örnek: çember alanı

Çıktıyı yazdırmak için `printf` kullanacağız (kendi float yazdırma kodumuzu yazmak ayrı bir konu). Bu yüzden program `main` ile başlar.

```nasm
default rel
extern printf

section .data
    fmt     db "Alan = %.2f", 10, 0
    pi      dq 3.14159265358979
    yaricap dq 5.0

section .text
    global main

main:
    push rbp
    mov rbp, rsp

    movsd xmm0, [yaricap]
    mulsd xmm0, xmm0            ; r * r
    mulsd xmm0, [pi]            ; * pi

    lea rdi, [fmt]
    mov eax, 1                  ; 1 adet xmm register'ı kullanıldı (vararg kuralı)
    call printf wrt ..plt

    xor eax, eax
    pop rbp
    ret
```

```bash
nasm -f elf64 alan.asm -o alan.o && gcc alan.o -o alan && ./alan
# Alan = 78.54
```

Notlar:

- Kayan nokta **argümanlar `xmm0`'dan başlar**, tamsayı argümanlardan bağımsız sayılır (`printf("%.2f", x)` için format `rdi`'de, `x` `xmm0`'dadır).
- `printf` gibi variadic fonksiyonlarda `al` = kullanılan xmm register sayısı olmalıdır.

### SIMD: tek komutla çok veri

SIMD (Single Instruction, Multiple Data) bir komutun birden çok veriyi aynı anda işlemesidir. 128 bitlik bir `xmm` register'ı:

- 16 adet 8 bitlik değer, veya
- 8 adet 16 bitlik, veya
- 4 adet 32 bitlik (int veya float), veya
- 2 adet 64 bitlik (int veya double)

tutabilir.

**Örnek: 4 adet 32 bitlik tamsayıyı aynı anda toplamak:**

```nasm
section .data
    a dd 1, 2, 3, 4
    b dd 10, 20, 30, 40

section .bss
    c resd 4

section .text
    global _start

_start:
    movdqu xmm0, [a]            ; 4 tamsayıyı yükle (hizalama gerektirmeyen)
    movdqu xmm1, [b]
    paddd  xmm0, xmm1           ; dört toplamayı birden yap: {11, 22, 33, 44}
    movdqu [c], xmm0

    mov edi, [c + 12]           ; c[3] = 44
    mov eax, 60
    syscall
```

**Önemli SIMD komut aileleri:**

| Önek / Komut           | Anlam                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- |
| `paddb/w/d/q`          | paketli tamsayı toplama (bayt/word/dword/qword)                                                             |
| `psubd`, `pmulld`      | çıkarma, çarpma                                                                                             |
| `addps` / `addpd`      | 4 float / 2 double paketli toplama                                                                          |
| `pxor`, `pand`, `por`  | paketli mantıksal işlemler                                                                                  |
| `pcmpeqb` + `pmovmskb` | 16 baytı aynı anda karşılaştırıp sonucu bit maskesine çevirme (hızlı `strlen`, `memchr` bu şekilde yazılır) |

SSE'nin devamı **AVX / AVX2** (256 bit, `ymm` register'ları) ve **AVX-512** (512 bit, `zmm`) vardır. Aynı mantık, daha geniş register'lar. AVX komutları `v` öneki alır (`vaddpd ymm0, ymm1, ymm2`, üç operandlı).

### Alıştırmalar

1. Bir dairenin çevresini (2πr) hesaplayıp yazdır.
2. İki `double`'ın ortalamasını hesapla.
3. 4 elemanlı iki `float` dizisini `addps` ile topla.
4. Bir tamsayıyı `cvtsi2sd` ile double'a çevirip karekökünü `sqrtsd` ile bul, `printf` ile yazdır.

---

[← Bölüm 18: Dosyalar ve komut satırı argümanları](18-dosyalar-ve-komut-satiri-argumanlari.md) · [Bölüm 20: Projeler ve çözümler →](20-projeler-ve-cozumler.md)
