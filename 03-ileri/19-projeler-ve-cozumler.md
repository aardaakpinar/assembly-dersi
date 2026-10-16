[Ana sayfa / İçindekiler](../README.md)

# Bölüm 19: Projeler ve çözümler

Öğrendiklerini pekiştirmek için projeler. Zorluk sırasına göre:

### 🟢 Kolay

1. **Selamlayıcı:** Adı sor, "Merhaba, X!" yaz. (Bölüm 8)
2. **Toplama makinesi:** İki tek haneli sayıyı klavyeden oku, toplamı yazdır. (`atoi` + `yazdir_sayi`)
3. **Geri sayım:** 10'dan 1'e kadar say, sonra "Basla!" yaz.
4. **Tek/çift:** Girilen sayı tek mi çift mi söylesin.

### 🟡 Orta

5. **Faktöriyel hesaplayıcı:** Klavyeden sayı al, faktöriyelini yazdır. (20!'in 64 bit'e sığdığını, 21!'in sığmadığını gözle.)
6. **Asal sayı testi:** Sayıyı 2'den karekökü civarına kadar bölerek test et.
7. **Palindrom kontrolü:** String ileri-geri aynı mı?
8. **Basit hesap makinesi:** `./hesap 12 + 30` gibi komut satırı argümanlarını işle.
9. **wc benzeri:** Dosyadaki satır sayısını say.
10. **Caesar şifreleme:** Metni belirli kaydırma ile şifrele.

### 🔴 Zor

11. **Mini `cat`/`hexdump`:** Dosyayı hex olarak yazdır.
12. **Basit shell:** Satır oku, `fork` + `execve` ile çalıştır. (Syscall'lar: 57, 59, 61)
13. **TCP echo sunucusu:** `socket`, `bind`, `listen`, `accept`, `read`, `write` (syscall 41, 49, 50, 43...). Saf assembly ile ağ programı!
14. **Sıralama algoritması:** Bubble sort, sonra quick sort.
15. **Oyun:** Terminalde "sayı tahmin" oyunu (rastgele sayı için `getrandom` syscall 318 veya `rdrand` komutu).

### Çözüm: string ters çevirme

```nasm
section .data
    metin db "assembly", 0
    uzun  equ $ - metin - 1         ; sonlandırıcı 0 hariç uzunluk
    yeni  db 10

section .text
    global _start

_start:
    lea rsi, [metin]                ; sol işaretçi
    lea rdi, [metin + uzun - 1]     ; sağ işaretçi
.dongu:
    cmp rsi, rdi
    jae .bitti                      ; işaretçiler buluştu/geçti
    mov al, [rsi]
    mov bl, [rdi]
    mov [rsi], bl                   ; takas
    mov [rdi], al
    inc rsi
    dec rdi
    jmp .dongu
.bitti:
    mov rax, 1
    mov rdi, 1
    lea rsi, [metin]
    mov rdx, uzun
    syscall

    mov rax, 1
    mov rdi, 1
    lea rsi, [yeni]
    mov rdx, 1
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall
```

Çıktı: `ylbmessa`. İki işaretçi uçlardan ortaya doğru ilerler, her adımda takas yapar. `.data` bölümü yazılabilir olduğu için bu çalışır (`.rodata`'da segfault alırdın).

### Çözüm: asal sayı testi

```nasm
; rdi = n → rax = 1 (asal) veya 0 (asal değil)
asal_mi:
    cmp rdi, 2
    jb .degil                       ; n < 2 asal değil
    mov rcx, 2                      ; bölen
.dongu:
    mov rax, rcx
    mul rax                         ; rax = rcx * rcx
    cmp rax, rdi
    ja .asal                        ; rcx² > n ise bölen kalmadı → asal
    mov rax, rdi
    xor rdx, rdx
    div rcx                         ; rdx = n % rcx
    test rdx, rdx
    jz .degil                       ; tam bölündü → asal değil
    inc rcx
    jmp .dongu
.asal:
    mov rax, 1
    ret
.degil:
    xor eax, eax
    ret
```

Kullanım: `mov rdi, 97` → `call asal_mi` → `rax = 1`. Bölenleri yalnızca `√n`'ye kadar denemek yeterlidir; `rcx² > n` kontrolü bunu karekök almadan yapar.

### Çözüm: faktöriyel için çıkış kodu yerine yazdırma

Bölüm 14'teki `faktoriyel` ve `yazdir_sayi` fonksiyonlarını birleştirip şunu çalıştır:

```nasm
_start:
    mov rdi, 15
    call faktoriyel
    mov rdi, rax
    call yazdir_sayi            ; 1307674368000
    mov rax, 60
    xor rdi, rdi
    syscall
```

### Sonraki adımlar

Bu dersi bitirdiysen temel x64 assembly'yi biliyorsun. Buradan sonra:

- **Tersine mühendislik:** Ghidra, radare2, IDA ile gerçek programları incele; CTF'lerde `crackme` ve `pwn` kategorileri.
- **Güvenlik:** Stack overflow, ROP (return-oriented programming), shellcode yazımı.
- **İşletim sistemi geliştirme:** 16 bit real mode → 32 bit protected mode → 64 bit long mode geçişi, bootloader yazmak.
- **Optimizasyon:** Intel/AMD işlemci mimarisi, pipeline, cache, AVX ile vektörleştirme.
- **Diğer mimariler:** ARM64 (Apple Silicon, Raspberry Pi) ve RISC-V, benzer mantık, farklı komutlar.

---

[← Bölüm 18: Kayan noktalı sayılar ve SIMD](18-kayan-noktali-sayilar-ve-simd.md) · [Ek A: Syscall tablosu →](../ek/ek-a-syscall-tablosu.md)
