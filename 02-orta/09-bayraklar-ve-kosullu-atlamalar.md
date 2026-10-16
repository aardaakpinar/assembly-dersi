[Ana sayfa / İçindekiler](../README.md)

# Bölüm 9: Bayraklar ve koşullu atlamalar

Şimdiye kadar programlar yukarıdan aşağı sıralı çalıştı. Gerçek programlar karar verir (if/else) ve tekrar eder (döngü). Assembly'de bunun yolu **atlamalardır (jump)**.

### Koşulsuz atlama: JMP

```nasm
    jmp son          ; doğrudan 'son' etiketine atla
    mov rax, 1       ; bu satır hiç çalışmaz
son:
    mov rax, 2
```

### Karşılaştırma: CMP ve TEST

`cmp a, b` aslında `a - b` işlemini yapar ama **sonucu atar, yalnızca bayrakları günceller.**

```nasm
cmp rax, rbx        ; rax - rbx hesapla, bayrakları ayarla
test rax, rax       ; rax AND rax hesapla, bayrakları ayarla (rax sıfır mı diye bakmak için ideal)
test rax, 1         ; en düşük bit 1 mi? (tek sayı kontrolü)
```

### Koşullu atlamalar

Bir `cmp` komutundan hemen sonra, bayraklara göre atlama yapılır:

**Eşitlik (işaretli/işaretsiz fark etmez):**

| Komut         | Anlam                    | Bayrak |
| ------------- | ------------------------ | ------ |
| `je` / `jz`   | eşit / sıfır             | ZF=1   |
| `jne` / `jnz` | eşit değil / sıfır değil | ZF=0   |

**İşaretli karşılaştırma (negatif sayılar olabilir):**

| Komut | `cmp a, b` sonrası anlam |
| ----- | ------------------------ |
| `jg`  | a > b                    |
| `jge` | a ≥ b                    |
| `jl`  | a < b                    |
| `jle` | a ≤ b                    |

**İşaretsiz karşılaştırma (adresler, boyutlar):**

| Komut | `cmp a, b` sonrası anlam |
| ----- | ------------------------ |
| `ja`  | a > b (above)            |
| `jae` | a ≥ b                    |
| `jb`  | a < b (below)            |
| `jbe` | a ≤ b                    |

**Tek bayrağa bakanlar:**

| Komut        | Anlam                                          |
| ------------ | ---------------------------------------------- |
| `js` / `jns` | işaret bayrağı 1 / 0 (negatif / negatif değil) |
| `jc` / `jnc` | elde bayrağı 1 / 0                             |
| `jo` / `jno` | taşma bayrağı 1 / 0                            |

### 🔑 İşaretli mi, işaretsiz mi?

`rax = 0xFFFFFFFFFFFFFFFF` ve `rbx = 1` olsun. `cmp rax, rbx` sonrası:

- `jg` (işaretli): rax = -1, rbx = 1 → -1 > 1 **değil** → atlamaz
- `ja` (işaretsiz): rax = 18 kentilyon, rbx = 1 → büyük → **atlar**

Aynı bit düzeni, farklı sonuç. **Doğru atlama komutunu seçmek senin sorumluluğun.** C'de `int` ile `unsigned` farkı derleyicinin hangi komutu seçeceğini belirler.

### if / else yapısı

C:

```c
if (a > b) {
    max = a;
} else {
    max = b;
}
```

Assembly:

```nasm
    mov rax, [a]
    mov rbx, [b]
    cmp rax, rbx
    jle .b_buyuk         ; a <= b ise b'ye git
    mov [max], rax       ; a > b
    jmp .devam
.b_buyuk:
    mov [max], rbx
.devam:
```

Kalıp: **koşulun tersini** kullanıp "if bloğunu atla" demek. Kolay okunur ve derleyicilerin yaptığı da budur.

### Yerel etiketler

`.` ile başlayan etiketler bir önceki normal etiketin altına aittir. Böylece her fonksiyonda aynı isimleri (`.dongu`, `.bitti`) tekrar tekrar kullanabilirsin:

```nasm
fonk1:
.dongu: ...       ; gerçek adı: fonk1.dongu
fonk2:
.dongu: ...       ; gerçek adı: fonk2.dongu  (çakışma yok)
```

### Tam örnek: pozitif / negatif / sıfır

```nasm
section .data
    sayi    dq -7
    poz     db "Pozitif", 10
    poz_len equ $ - poz
    neg_    db "Negatif", 10
    neg_len equ $ - neg_
    sif     db "Sifir", 10
    sif_len equ $ - sif

section .text
    global _start

_start:
    mov rax, [sayi]
    test rax, rax        ; bayrakları ayarla
    jz .sifir
    js .negatif

.pozitif:
    mov rsi, poz
    mov rdx, poz_len
    jmp .yaz
.negatif:
    mov rsi, neg_
    mov rdx, neg_len
    jmp .yaz
.sifir:
    mov rsi, sif
    mov rdx, sif_len

.yaz:
    mov rax, 1
    mov rdi, 1
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall
```

(`neg` bir komut adı olduğu için etiketi `neg_` yaptık; **komut adlarını etiket olarak kullanamazsın.**)

### Koşullu mov: CMOVcc

Küçük if/else için atlamasız alternatif:

```nasm
; rax = max(rax, rbx)
cmp rax, rbx
cmovl rax, rbx      ; rax < rbx ise rax = rbx
```

`cmove`, `cmovne`, `cmovg`, `cmovl`, `cmova`, `cmovb` ... aynı koşul ekleri.

### Koşula göre 0/1 üretmek: SETcc

```nasm
cmp rax, rbx
setl cl             ; rax < rbx ise cl = 1, değilse 0
movzx rcx, cl       ; tam register'a genişlet
```

### Alıştırmalar

1. İki sayının küçüğünü çıkış kodu olarak döndüren program yaz.
2. Bir sayının mutlak değerini hesapla (`neg` ve `jns` / `cmovs` kullan).
3. Bir sayı çift mi tek mi? Ekrana "Cift" veya "Tek" yazdır. (İpucu: `test rax, 1`)
4. Bir notu harfe çevir: ≥90 → A, ≥80 → B, ≥70 → C, aksi halde F (çıkış kodu olarak harfin ASCII değerini ver).

---

[← Bölüm 8: Sistem çağrıları (syscall)](../01-baslangic/08-sistem-cagrilari-syscall.md) · [Bölüm 10: Döngüler →](10-donguler.md)
