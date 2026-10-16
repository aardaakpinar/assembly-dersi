[Ana sayfa / İçindekiler](../README.md)

# Bölüm 5: Veri tanımlama, MOV ve adresleme

### Bir programın bölümleri (sections)

| Bölüm             | İçerik                                                                             |
| ----------------- | ---------------------------------------------------------------------------------- |
| `section .text`   | Kod (salt okunur, çalıştırılabilir)                                                |
| `section .data`   | Başlangıç değerli, değiştirilebilir veri                                           |
| `section .rodata` | Başlangıç değerli, **salt okunur** veri (sabit stringler)                          |
| `section .bss`    | Başlangıç değeri **olmayan** veri (program başlayınca sıfır). Dosyada yer kaplamaz |

### Veri tanımlama

Başlangıç değerli veriler (`.data`):

```nasm
section .data
    yas     db 25              ; 1 bayt
    sayi    dw 1000            ; 2 bayt
    buyuk   dd 100000          ; 4 bayt
    dev     dq 5000000000      ; 8 bayt
    isim    db "Ali", 0        ; karakter dizisi + sonlandırıcı 0
    dizi    dd 1, 2, 3, 4      ; 4 elemanlı dizi (her biri 4 bayt)
    sifirlar times 10 db 0     ; 10 tane sıfır bayt
```

Başlangıç değersiz veriler (`.bss`) için yer **ayırma** (`res*`):

```nasm
section .bss
    tampon  resb 64            ; 64 bayt
    sonuc   resq 1             ; 1 adet 8 baytlık
    liste   resd 100           ; 100 adet 4 baytlık
```

| Tanım | Ayırma | Boyut  |
| ----- | ------ | ------ |
| `db`  | `resb` | 1 bayt |
| `dw`  | `resw` | 2 bayt |
| `dd`  | `resd` | 4 bayt |
| `dq`  | `resq` | 8 bayt |

Etiket (label) aslında bir **adrestir**. `yas` yazdığında o baytın adresini kastedersin.

### MOV komutu

`mov hedef, kaynak`: kaynaktan hedefe **kopyalar** (kaynak değişmez).

```nasm
mov rax, 10          ; register ← sabit
mov rbx, rax         ; register ← register
mov rcx, [dev]       ; register ← bellek (köşeli parantez: bellekteki DEĞER)
mov [sonuc], rax     ; bellek ← register
mov rsi, dev         ; register ← ADRES (köşeli parantez yok)
mov dword [buyuk], 5 ; bellek ← sabit (boyut belirtmek ZORUNLU)
```

#### 🔑 En sık karıştırılan konu: `dev` ile `[dev]`

```nasm
mov rax, dev      ; rax = dev değişkeninin ADRESİ (ör. 0x404010)
mov rax, [dev]    ; rax = o adreste duran DEĞER (ör. 5000000000)
```

C'deki karşılığı: `&dev` ile `dev`.

#### MOV kuralları

1. **Bellekten belleğe doğrudan mov yoktur.**
    ```nasm
    mov [a], [b]      ; ❌ HATA
    mov rax, [b]      ; ✅ önce register'a
    mov [a], rax      ;    sonra belleğe
    ```
2. Boyutlar uyuşmalı: `mov eax, rbx` hatadır.
3. Bellek ile sabit işlemde boyut belirtmelisin, çünkü NASM kaç bayt yazacağını bilemez:
    ```nasm
    mov byte  [yas], 30
    mov word  [sayi], 7
    mov dword [buyuk], 9
    mov qword [dev], 1
    ```
4. 64 bit sabit yalnızca register'a yüklenebilir (`mov rax, 0x1122334455667788`). Belleğe 64 bit sabit yazmak için önce register kullan. Belleğe en fazla 32 bit (işaretli genişletilen) sabit yazılır.

### Boyut genişletme: movzx, movsx, movsxd

Küçük bir değeri büyük register'a alırken üst bitler ne olsun?

```nasm
movzx rax, byte [yas]      ; zero-extend: üst bitleri 0'la doldur (işaretsiz için)
movsx rax, byte [yas]      ; sign-extend: işaret bitini kopyala (işaretli için)
movsxd rax, dword [buyuk]  ; 32 bit işaretliyi 64 bite genişlet
mov eax, [buyuk]           ; 32 bit yükle; üst 32 bit otomatik sıfırlanır
```

Örnek: bellekte bayt `0xFF` varsa

- `movzx rax, byte [...]` → `rax = 255`
- `movsx rax, byte [...]` → `rax = -1` (`0xFFFFFFFFFFFFFFFF`)

### Adresleme modları

Bellek operandı şu genel formdadır:

```
[taban + indeks * ölçek + kayma]
```

- taban: herhangi bir 64 bit register
- indeks: herhangi bir register (`rsp` hariç)
- ölçek: 1, 2, 4 veya 8
- kayma: sabit sayı

```nasm
mov rax, [rbx]               ; taban
mov rax, [rbx + 8]           ; taban + kayma
mov rax, [rbx + rcx]         ; taban + indeks
mov eax, [dizi + rcx*4]      ; dizi[rcx] (4 baytlık elemanlar)
mov rax, [rbx + rcx*8 + 16]  ; hepsi birden
```

Bu yapı doğrudan dizi erişimine uygundur: `dizi[i]` → `[dizi + i*eleman_boyutu]`.

### LEA: adres hesapla (belleğe erişmeden)

`lea` (Load Effective Address) adresi **hesaplar ama bellekten okumaz**:

```nasm
lea rax, [rbx + rcx*4 + 8]   ; rax = rbx + rcx*4 + 8  (bellek erişimi YOK)
```

Bu yüzden aritmetik için de kullanılır:

```nasm
lea rax, [rdi + rsi]         ; rax = rdi + rsi (bayrakları değiştirmeden toplama)
lea rax, [rdi + rdi*4]       ; rax = rdi * 5
```

### RIP-göreli adresleme

Modern programlar çoğunlukla **konumdan bağımsız (PIE)** derlenir. Bu durumda veri adresleri `rip`'e göre hesaplanır:

```nasm
default rel                  ; dosyanın başında: varsayılan olarak RIP-göreli adresle
mov rax, [dev]               ; artık otomatik [rip + fark]
lea rsi, [rel mesaj]         ; veya tek tek: rel anahtar sözcüğü
```

Bu derste `ld` ile doğrudan bağladığımız küçük programlarda buna gerek yok. C ile birlikte çalışırken (Bölüm 16) şart olacak.

### XCHG

```nasm
xchg rax, rbx      ; iki register'ın değerini takas et
```

### Tam örnek

```nasm
section .data
    sayi1 dq 100
    sayi2 dq 23

section .bss
    toplam resq 1

section .text
    global _start

_start:
    mov rax, [sayi1]     ; rax = 100
    add rax, [sayi2]     ; rax = 123
    mov [toplam], rax    ; belleğe yaz

    mov rdi, [toplam]    ; çıkış kodu = 123
    mov rax, 60
    syscall
```

```bash
nasm -f elf64 -g -F dwarf ornek.asm -o ornek.o && ld ornek.o -o ornek
./ornek; echo $?        # 123
```

### Alıştırmalar

1. `dd 10, 20, 30` dizisinin 3. elemanını (30) `eax`'e yükle.
2. `mov rax, dizi` ile `mov rax, [dizi]` arasındaki farkı kendi cümlelerinle yaz.
3. İki değişkenin değerini takas eden bir program yaz (önce `xchg` ile, sonra sadece `mov` ile).
4. `lea` ile `rdi * 9` hesapla. İpucu: `[rdi + rdi*8]`

---

[← Bölüm 4: Registerlar (Yazmaçlar)](04-registerlar-yazmaclar.md) · [Bölüm 6: Aritmetik komutlar →](06-aritmetik-komutlar.md)
