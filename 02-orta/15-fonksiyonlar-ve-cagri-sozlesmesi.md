[Ana sayfa / İçindekiler](../README.md)

# Bölüm 15: Fonksiyonlar ve çağrı sözleşmesi

### call ve ret

```nasm
call fonksiyon     ; 1) dönüş adresini (call'dan sonraki komut) stack'e push et
                   ; 2) fonksiyon'a atla
ret                ; stack'ten dönüş adresini pop et ve oraya atla
```

Yani:

```nasm
call f    ≈    push (sonraki_komutun_adresi)
               jmp f
ret       ≈    pop rip
```

Basit örnek:

```nasm
section .text
    global _start

_start:
    mov rdi, 5
    call iki_kati          ; rax = 10
    mov rdi, rax
    mov rax, 60
    syscall

iki_kati:
    lea rax, [rdi*2]
    ret
```

### System V AMD64 çağrı sözleşmesi (ABI)

Fonksiyonlara argüman nasıl verilir, sonuç nasıl döner? Herkesin (C derleyicileri, kütüphaneler, sen) aynı kuralları kullanması gerekir. Linux x64 için kural **System V AMD64 ABI**'dir.

**Tam sayı / pointer argümanlar (sırayla):**

| Argüman      | Register                           |
| ------------ | ---------------------------------- |
| 1            | `rdi`                              |
| 2            | `rsi`                              |
| 3            | `rdx`                              |
| 4            | `rcx`                              |
| 5            | `r8`                               |
| 6            | `r9`                               |
| 7 ve sonrası | stack'te (sağdan sola push edilir) |

Dikkat: Syscall'da 4. argüman `r10`'du, fonksiyonlarda `rcx`. İkisini karıştırma.

**Kayan nokta argümanlar:** `xmm0`–`xmm7`

**Dönüş değeri:** `rax` (tam sayı / pointer), `xmm0` (kayan nokta). 128 bit için `rdx:rax`.

**Registerlar kimin sorumluluğunda?**

| Tür                        | Registerlar                                 | Anlamı                                                                                            |
| -------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Caller-saved** (uçucu)   | `rax, rcx, rdx, rsi, rdi, r8, r9, r10, r11` | Fonksiyon bunları **istediği gibi bozabilir**. Çağıran, değerleri lazımsa çağrıdan önce saklamalı |
| **Callee-saved** (korunan) | `rbx, rbp, r12, r13, r14, r15`              | Fonksiyon bunları kullanırsa **çıkarken eski değerlerini geri koymalı**                           |
| `rsp`                      |                                             | Çıkışta girişteki değerine dönmeli                                                                |

> **Pratik kural:** Bir fonksiyon çağırdıktan sonra da lazım olan değerleri `rbx`, `r12`–`r15` içinde tut. Fonksiyon bunları korumak zorunda.

### Stack frame (çerçeve)

Fonksiyonların standart giriş-çıkış kalıbı:

```nasm
fonksiyon:
    push rbp              ; eski rbp'yi sakla
    mov rbp, rsp          ; yeni çerçeve tabanı
    sub rsp, 32           ; yerel değişkenler için yer aç (isteğe bağlı)

    ; ... fonksiyon gövdesi ...
    ; yerel değişkenler: [rbp - 8], [rbp - 16], ...
    ; 7. argüman: [rbp + 16], 8. argüman: [rbp + 24], ...

    mov rsp, rbp          ; yerel alanı serbest bırak  (veya: leave)
    pop rbp               ; eski rbp'yi geri al
    ret
```

`leave` komutu `mov rsp, rbp` + `pop rbp` işini yapar.

Stack'in görünümü:

```
yüksek adres
   ┌────────────────────┐
   │ 7. argüman         │ [rbp + 16]
   ├────────────────────┤
   │ dönüş adresi       │ [rbp + 8]
   ├────────────────────┤
   │ eski rbp           │ [rbp]        ← rbp burada
   ├────────────────────┤
   │ yerel değişken 1   │ [rbp - 8]
   ├────────────────────┤
   │ yerel değişken 2   │ [rbp - 16]   ← rsp burada
   └────────────────────┘
düşük adres
```

Küçük "yaprak" (başka fonksiyon çağırmayan) fonksiyonlar çoğunlukla çerçeve kurmaz. Basit olanlar `lea`/`ret` kadar kısadır.

### Red zone

System V ABI'de yaprak fonksiyonlar, `rsp`'nin **altındaki 128 baytı** `rsp`'yi değiştirmeden geçici alan olarak kullanabilir. Sinyal yöneticileri bu bölgeye dokunmaz. Kullanıcı kodunda önemli olan: yaprak fonksiyonda `sub rsp` yapmadan `[rsp-8]` kullanabilirsin ama başka fonksiyon çağıracaksan kullanamazsın.

### Stack hizalaması

ABI'ye göre `call` komutu çalışmadan hemen önce **`rsp` 16'nın katı** olmalıdır. Çağrılan fonksiyonun girişinde (dönüş adresi push edildikten sonra) `rsp % 16 == 8` olur. Libc fonksiyonları (printf vb.) buna **güvenir**; hizalama bozuksa SSE komutlarında segfault alabilirsin. Bunu Bölüm 17'de C kütüphanesi çağırırken ayrıntılı göreceksin.

### Örnek: faktöriyel (özyinelemeli)

```nasm
; rdi = n → rax = n!
faktoriyel:
    cmp rdi, 1
    jbe .taban               ; n <= 1 ise 1 döndür
    push rdi                 ; n'yi sakla
    dec rdi
    call faktoriyel          ; rax = (n-1)!
    pop rdi                  ; n'yi geri al
    imul rax, rdi            ; rax = (n-1)! * n
    ret
.taban:
    mov rax, 1
    ret
```

Çağrı: `mov rdi, 5` → `call faktoriyel` → `rax = 120`.

Her çağrı stack'e dönüş adresi ve `rdi` koyar, geri dönüşte bunlar sırayla alınır. GDB'de `x/20xg $rsp` ile stack'in büyüyüp küçülmesini izlemek çok öğretici.

### Örnek: sayı yazdırma fonksiyonu

Artık sayıları ekrana yazdırabiliriz. Sayıyı basamaklarına ayırmak için sürekli 10'a bölüp kalanları alırız.

```nasm
section .bss
    tampon resb 21               ; en fazla 20 basamak + satır sonu

section .text
    global _start

_start:
    mov r12, 1                   ; r12: callee-saved, yazdir_sayi'dan etkilenmez
.dongu:
    mov rdi, r12
    call yazdir_sayi
    inc r12
    cmp r12, 10
    jle .dongu

    mov rax, 60
    xor rdi, rdi
    syscall

; Girdi:  rdi = yazdırılacak işaretsiz sayı
; Bozar:  rax, rcx, rdx, rsi, rdi, r11  (caller-saved)
yazdir_sayi:
    mov rax, rdi
    lea rsi, [tampon + 20]       ; tamponu sondan dolduruyoruz
    mov byte [rsi], 10           ; son karakter: satır sonu
    mov rcx, 10                  ; bölen
.basamak:
    xor rdx, rdx
    div rcx                      ; rax = bölüm, rdx = kalan (son basamak)
    add dl, '0'                  ; rakamı ASCII karaktere çevir
    dec rsi
    mov [rsi], dl
    test rax, rax
    jnz .basamak                 ; bölüm 0 olana kadar devam

    lea rdx, [tampon + 21]
    sub rdx, rsi                 ; uzunluk = bitiş - başlangıç
    mov rax, 1
    mov rdi, 1
    syscall
    ret
```

Çıktı 1'den 10'a kadar her biri ayrı satırda. Bu fonksiyonu artık tüm programlarında kullanabilirsin.

### 7'den fazla argüman

```nasm
; f(1,2,3,4,5,6,7,8) çağrısı
push 8                  ; 8. argüman (önce)
push 7                  ; 7. argüman
mov rdi, 1
mov rsi, 2
mov rdx, 3
mov rcx, 4
mov r8, 5
mov r9, 6
call f
add rsp, 16             ; çağıran, stack'teki argümanları temizler
```

Fonksiyonda `[rbp + 16]` = 7. argüman, `[rbp + 24]` = 8. argüman.

### Alıştırmalar

1. İki sayının en büyüğünü döndüren `maks(rdi, rsi)` fonksiyonu yaz.
2. `fib(n)` özyinelemeli Fibonacci fonksiyonu yaz. (Callee-saved register kullanman gerekecek.)
3. `strlen`, `yazdir_string(rdi=adres)` ve `yazdir_sayi` fonksiyonlarından oluşan küçük bir "kütüphane" oluştur.
4. `yazdir_sayi`'yı negatif sayıları da yazdıracak şekilde genişlet (önce `-` yaz, sonra `neg` ile pozitif yap).
5. GDB'de `faktoriyel(4)` çağrısında her `call` sonrası stack'i izle.

---

[← Bölüm 14: Stack (Yığın)](14-stack-yigin.md) · [Bölüm 16: Makrolar ve çok dosyalı projeler →](16-makrolar-ve-cok-dosyali-projeler.md)
