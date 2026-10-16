[Ana sayfa / İçindekiler](../README.md)

# Bölüm 10: Döngüler

Assembly'de ayrı bir `for` / `while` yoktur. **Geri atlama ve koşul** ile yapılır.

### Temel döngü kalıpları

**while / for kalıbı:**

```c
for (i = 0; i < 10; i++) { ... }
```

```nasm
    xor rcx, rcx         ; i = 0
.dongu:
    cmp rcx, 10
    jge .bitti           ; i >= 10 ise çık
    ; ... döngü gövdesi ...
    inc rcx
    jmp .dongu
.bitti:
```

**do-while kalıbı (derleyicilerin tercihi, daha az atlama):**

```nasm
    mov rcx, 10          ; sayaç
.dongu:
    ; ... gövde ...
    dec rcx
    jnz .dongu           ; sıfır değilse tekrar
```

`dec` ZF'yi günceller, ayrıca `cmp` gerekmez.

### Örnek: 1'den 10'a kadar toplam

```nasm
section .text
    global _start

_start:
    xor rax, rax         ; toplam = 0
    mov rcx, 1           ; i = 1
.dongu:
    add rax, rcx         ; toplam += i
    inc rcx
    cmp rcx, 10
    jle .dongu           ; i <= 10 ise devam

    mov rdi, rax         ; 55
    mov rax, 60
    syscall
```

### LOOP komutu

`loop etiket`: `rcx`'i bir azaltır, sıfır değilse atlar. Kısa ama modern işlemcilerde `dec + jnz` ikilisinden yavaştır; öğrenmek iyi ama kullanma:

```nasm
    mov rcx, 10
.dongu:
    ; gövde
    loop .dongu
```

⚠️ `syscall` komutu `rcx`'i bozduğu için döngü içinde syscall varsa sayaç olarak `rcx` **kullanma**; `r12`–`r15` kullan.

### Örnek: faktöriyel (5!)

```nasm
_start:
    mov rax, 1           ; sonuç
    mov rcx, 5           ; n
.dongu:
    imul rax, rcx        ; sonuç *= n
    dec rcx
    jnz .dongu           ; n = 0 olunca bitir
    mov rdi, rax         ; 120
    mov rax, 60
    syscall
```

### Örnek: iç içe döngü, yıldız üçgeni

```
*
**
***
****
*****
```

```nasm
section .data
    yildiz db '*'
    yeni   db 10

section .text
    global _start

_start:
    mov r12, 1               ; satır numarası (1..5)
.satir:
    mov r13, r12             ; bu satırda r12 tane yıldız basılacak
.yildiz:
    mov rax, 1
    mov rdi, 1
    mov rsi, yildiz
    mov rdx, 1
    syscall
    dec r13
    jnz .yildiz

    mov rax, 1               ; satır sonu
    mov rdi, 1
    mov rsi, yeni
    mov rdx, 1
    syscall

    inc r12
    cmp r12, 5
    jle .satir

    mov rax, 60
    xor rdi, rdi
    syscall
```

Sayaçlar için `r12`/`r13` seçtik çünkü `syscall` onları bozmaz.

### Örnek: Fibonacci (10. terim)

```nasm
_start:
    mov rax, 0           ; F(0)
    mov rbx, 1           ; F(1)
    mov rcx, 10          ; kaç adım
.dongu:
    mov rdx, rax
    add rdx, rbx         ; sonraki = a + b
    mov rax, rbx         ; a = b
    mov rbx, rdx         ; b = sonraki
    dec rcx
    jnz .dongu
    mov rdi, rax         ; F(10) = 55
    mov rax, 60
    syscall
```

### Alıştırmalar

1. 1'den 100'e kadar tek sayıların toplamını bul. (Çıkış kodu 255'i geçeceği için `mod 256` görünür. Sonucun doğruluğunu GDB'de `p $rax` ile kontrol et.)
2. 8 satırlık bir "yıldız piramidi" çiz (boşluklarla ortalanmış).
3. Çarpım tablosunu (örneğin 7'nin tablosu) hesapla. Yazdırmak için Bölüm 14'teki `yazdir_sayi` fonksiyonunu bekle.
4. Bir dizideki en büyük elemanı bulan program için Bölüm 12'yi bekle.

---

[← Bölüm 9: Bayraklar ve koşullu atlamalar](09-bayraklar-ve-kosullu-atlamalar.md) · [Bölüm 11: Bit operasyonları →](11-bit-operasyonlari.md)
