[Ana sayfa / İçindekiler](../README.md)

# Bölüm 12: Bit operasyonları

Assembly'nin güçlü yanlarından biri: bit'lerle doğrudan oynayabilmek.

### Mantıksal komutlar

| Komut      | İşlem             | Örnek (8 bit)               |
| ---------- | ----------------- | --------------------------- |
| `and a, b` | bit bit VE        | `1100 & 1010 = 1000`        |
| `or a, b`  | bit bit VEYA      | `1100 \| 1010 = 1110`       |
| `xor a, b` | bit bit ÖZEL VEYA | `1100 ^ 1010 = 0110`        |
| `not a`    | bitleri tersle    | `~1100 = 0011` (4 bit için) |

```nasm
mov al, 0b1100
and al, 0b1010          ; al = 0b1000
```

### Sık kullanılan kalıplar

**Register'ı sıfırla:**

```nasm
xor eax, eax            ; eax = 0 (mov eax, 0'dan kısa ve hızlı)
```

Bir değeri kendisiyle XOR'larsan sıfır olur.

**Bit maskeleme:**

```nasm
; Bir bayt içinde 3. bit (0'dan sayarak) açık mı?
test al, 0b1000
jnz .acik

; 3. biti AÇ (1 yap)
or  al, 0b1000

; 3. biti KAPAT (0 yap)
and al, 0b11110111      ; veya: and al, ~0b1000

; 3. biti TERSLE
xor al, 0b1000
```

**Çift mi tek mi:** en düşük bit 0 ise çift.

```nasm
test rdi, 1
jz .cift
```

**Büyük harfi küçük harfe çevir (ASCII):** `'A'` = `0x41`, `'a'` = `0x61`. Fark yalnızca 5. bit (`0x20`).

```nasm
or  al, 0x20            ; 'A' → 'a'
and al, 0xDF            ; 'a' → 'A'   (0xDF = ~0x20)
```

### Kaydırma (shift) ve döndürme (rotate)

```
shl rax, n     ; sola kaydır: sağdan 0 gelir        (× 2^n)
shr rax, n     ; mantıksal sağa kaydır: soldan 0     (÷ 2^n, işaretsiz)
sar rax, n     ; aritmetik sağa kaydır: işaret biti korunur (÷ 2^n, işaretli)
rol rax, n     ; sola döndür: soldan taşan bit sağdan girer
ror rax, n     ; sağa döndür
```

`n` ya sabit sayıdır ya da `cl` register'ıdır:

```nasm
mov cl, 3
shl rax, cl             ; rax = rax * 8
```

Örnekler:

```nasm
mov rax, 5
shl rax, 3              ; 5 * 8 = 40

mov rax, 40
shr rax, 2              ; 40 / 4 = 10

mov rax, -16
sar rax, 2              ; -16 / 4 = -4
```

Çıkan bit `CF` bayrağına girer.

**Çarpmayı kaydırma ile hızlandırma:** `x * 10 = x*8 + x*2`

```nasm
lea rax, [rdi + rdi*4]  ; rdi * 5
add rax, rax            ; * 2  → rdi * 10
```

### Diğer bit komutları

```nasm
bt  rax, 5              ; 5. biti CF'ye kopyala (test)
bts rax, 5              ; 5. biti CF'ye kopyala ve 1 yap
btr rax, 5              ; ... ve 0 yap
btc rax, 5              ; ... ve tersle
popcnt rbx, rax         ; rax'taki 1 bitlerinin sayısı
bsf rbx, rax            ; en düşük 1 bitin indeksi
bsr rbx, rax            ; en yüksek 1 bitin indeksi
```

### Püf noktası: 2'nin kuvveti mi?

`n & (n-1)` en düşük 1 bitini siler. 2'nin kuvvetlerinde tek bir bit 1 olduğu için sonuç sıfır olur:

```nasm
; rdi, 2'nin kuvveti mi? (rdi > 0 varsayıyoruz)
lea rax, [rdi - 1]
test rdi, rax
jz .kuvvet
```

### Örnek: bir sayının ikilik gösterimini yazdırma

```nasm
section .bss
    tampon resb 65

section .text
    global _start

_start:
    mov rax, 0b101101        ; yazdırılacak sayı (45)
    lea rdi, [tampon]
    mov rcx, 64              ; 64 bit işlenecek
.dongu:
    rol rax, 1               ; en üst biti sağa getir (CF'ye de girer)
    mov rdx, rax
    and dl, 1                ; en düşük bit
    add dl, '0'              ; '0' veya '1' karakteri
    mov [rdi], dl
    inc rdi
    dec rcx
    jnz .dongu
    mov byte [rdi], 10       ; satır sonu

    mov rax, 1
    mov rdi, 1
    lea rsi, [tampon]
    mov rdx, 65
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall
```

Çıktı: 64 haneli ikilik sayı (başta çok sıfır). `rol` ile 64 kez döndürünce `rax` başlangıç değerine geri döner.

### Alıştırmalar

1. Bir baytın bit sayısını (kaç tane 1 var) döngüyle say. Sonra `popcnt` ile doğrula.
2. Bir sayının 7. bitini tersleyen kod yaz.
3. Küçük harfleri büyük harfe çeviren bir program yaz. Bir string'in her karakterine `and al, 0xDF` uygula (Bölüm 13 bilgisiyle).
4. `rol` ve `ror` kullanarak 32 bit bir sayının bayt sırasını ters çevir (endian dönüşümü). Sonra `bswap` komutunu bul ve karşılaştır.

---

[← Bölüm 11: Döngüler](11-donguler.md) · [Bölüm 13: Diziler ve stringler →](13-diziler-ve-stringler.md)
