[Ana sayfa / İçindekiler](../README.md)

# Bölüm 6: Aritmetik komutlar

### Toplama ve çıkarma

```nasm
add rax, rbx        ; rax = rax + rbx
add rax, 10         ; rax = rax + 10
sub rax, rbx        ; rax = rax - rbx
inc rax             ; rax = rax + 1
dec rax             ; rax = rax - 1
neg rax             ; rax = -rax  (işaret değiştir)
add qword [x], 5    ; bellekteki değeri doğrudan arttır
```

Bu komutlar **bayrakları günceller** (ZF, SF, CF, OF...). Bu bayraklar Bölüm 10'da işe yarayacak.

Örnek:

```nasm
mov rdi, 5
add rdi, 7          ; rdi = 12
sub rdi, 2          ; rdi = 10
mov rax, 60
syscall             ; çıkış kodu 10
```

### Çarpma

**`mul`: işaretsiz çarpma, tek operand:**

```nasm
mov rax, 6
mov rbx, 7
mul rbx             ; rdx:rax = rax * rbx  → rax = 42, rdx = 0
```

`mul` 64 bit çarpımın 128 bitlik sonucunu `rdx:rax` çiftine yazar (üst yarı `rdx`, alt yarı `rax`). Küçük sayılarda sonuç `rax`'te, `rdx` sıfırdır.

**`imul`: işaretli çarpma, üç kullanım biçimi:**

```nasm
imul rbx            ; rdx:rax = rax * rbx   (tek operand, mul gibi ama işaretli)
imul rax, rbx       ; rax = rax * rbx       (iki operand, sonuç 64 bit)
imul rax, rbx, 10   ; rax = rbx * 10        (üç operand)
```

Günlük kullanımda genelde 2 veya 3 operandlı `imul` kullanılır. En pratik olanıdır.

### Bölme

**`div`: işaretsiz bölme:**

```nasm
; 17 / 5
mov rax, 17         ; bölünen
xor rdx, rdx        ; ÜST YARIYI SIFIRLA! (zorunlu)
mov rbx, 5          ; bölen
div rbx             ; rax = bölüm (3), rdx = kalan (2)
```

`div rbx` aslında `rdx:rax` (128 bit) sayısını `rbx`'e böler. Bu yüzden **`rdx`'i sıfırlamayı unutma**; yoksa programın rastgele çöker (bölüm taşması, `SIGFPE`).

**`idiv`: işaretli bölme.** Bölünenin işaretini `rdx`'e yaymak için `cqo` kullan:

```nasm
mov rax, -17
cqo                 ; rax'ın işaretini rdx'e genişlet (rdx:rax = -17, 128 bit)
mov rbx, 5
idiv rbx            ; rax = -3, rdx = -2   (sıfıra doğru yuvarlar)
```

| Komut                   | Ne yapar                                               |
| ----------------------- | ------------------------------------------------------ |
| `cbw` / `cwde` / `cdqe` | `al`→`ax` / `ax`→`eax` / `eax`→`rax` işaret genişletme |
| `cwd` / `cdq` / `cqo`   | `ax`→`dx:ax` / `eax`→`edx:eax` / `rax`→`rdx:rax`       |

**Bölme sıfıra olursa?** İşlemci bir istisna (exception) üretir, Linux programı `SIGFPE` ile öldürür ("Floating point exception" mesajı, adı yanıltıcı olsa da tamsayı bölmesidir).

### Örnek: (a + b) × c − d

```nasm
section .data
    a dq 4
    b dq 6
    c dq 5
    d dq 8

section .text
    global _start

_start:
    mov rax, [a]
    add rax, [b]        ; rax = 10
    imul rax, [c]       ; rax = 50
    sub rax, [d]        ; rax = 42

    mov rdi, rax        ; çıkış kodu = 42
    mov rax, 60
    syscall
```

### Elde ile toplama (adc/sbb)

64 bitten büyük sayıları toplamak için `add` + `adc` (add with carry) kullanılır:

```nasm
; 128 bit toplama: (rdx:rax) += (rcx:rbx)
add rax, rbx        ; alt yarılar; taşma varsa CF=1
adc rdx, rcx        ; üst yarılar + CF
```

### Çıkış kodu sınırı

Linux çıkış kodu **yalnızca alt 8 bittir** (0–255). `exit(300)` → kabuk `44` gösterir (300 mod 256). Büyük sonuçları görmek için Bölüm 15'te sayı yazdırma yordamı yazacağız.

### Alıştırmalar

1. `100 / 7` bölümünü ve kalanını hesapla. Önce çıkış kodu olarak bölümü, sonra kalanı ver.
2. `a * a + b * b` değerini hesapla (a=3, b=4 → 25).
3. `-100 / 7`'yi `idiv` ile hesapla. Bölüm ve kalan ne çıkıyor? (Çıkış kodunda negatif sayılar nasıl görünür? Dene ve nedenini düşün.)
4. `rdx`'i sıfırlamadan `div` yapmayı dene ve ne olduğunu gözle.

---

[← Bölüm 5: Veri tanımlama, MOV ve adresleme](05-veri-tanimlama-mov-ve-adresleme.md) · [Bölüm 7: GDB ile hata ayıklama →](07-gdb-ile-hata-ayiklama.md)
