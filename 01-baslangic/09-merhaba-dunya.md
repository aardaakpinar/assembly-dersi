[Ana sayfa / İçindekiler](../README.md)

# Bölüm 9: Merhaba Dünya

Artık register'ları, `mov` ve `add` gibi komutları, çıkış kodunu ve sistem çağrılarının nasıl çalıştığını biliyorsun. Programlamanın geleneksel ilk programına sıra geldi: **ekrana yazı yazdırmak.**

### Plan

İki sistem çağrısı yeterli:

1. `write(1, mesaj, uzunluk)`: mesajı ekrana yaz
2. `exit(0)`: programı bitir

### Kod

`merhaba.asm`:

```nasm
section .data
    mesaj   db "Merhaba, Dünya!", 10    ; 10 = satır sonu (\n)
    uzunluk equ $ - mesaj               ; mesajın bayt cinsinden uzunluğu

section .text
    global _start

_start:
    ; write(1, mesaj, uzunluk)
    mov rax, 1          ; syscall numarası: write
    mov rdi, 1          ; dosya tanımlayıcı: 1 = stdout (ekran)
    mov rsi, mesaj      ; yazılacak verinin ADRESİ
    mov rdx, uzunluk    ; kaç bayt yazılacak
    syscall

    ; exit(0)
    mov rax, 60         ; syscall numarası: exit
    xor rdi, rdi        ; çıkış kodu: 0
    syscall
```

Derle ve çalıştır:

```bash
nasm -f elf64 merhaba.asm -o merhaba.o
ld merhaba.o -o merhaba
./merhaba
echo $?         # 0
```

Çıktı: `Merhaba, Dünya!`

### Satır satır açıklama

| Satır                   | Anlamı                                                                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `section .data`         | Başlangıç değeri olan verilerin bölümü (Bölüm 5)                                                                                                     |
| `mesaj db "..."`        | `mesaj` etiketiyle bayt dizisi tanımlar. Etiket, verinin **adresi**dir                                                                               |
| `10`                    | ASCII satır sonu karakteri (`\n`)                                                                                                                    |
| `uzunluk equ $ - mesaj` | `$` şu anki adrestir. Şimdiki adres eksi `mesaj`'ın başlangıç adresi, mesajın uzunluğunu verir. `equ` bunu **derleme anında hesaplanan sabit** yapar |
| `mov rax, 1`            | write sistem çağrısı numarası                                                                                                                        |
| `mov rdi, 1`            | fd 1 = ekran                                                                                                                                         |
| `mov rsi, mesaj`        | **adres** (köşeli parantez yok!). `[mesaj]` yazsaydın verinin kendisini (ilk 8 baytı) taşırdın                                                       |
| `mov rdx, uzunluk`      | kaç bayt yazılsın                                                                                                                                    |
| `xor rdi, rdi`          | `rdi = 0`. Bir register'ı kendisiyle XOR'lamak sıfırlar (Bölüm 12'de ayrıntısı)                                                                      |

> **Not:** `ü` gibi Türkçe karakterler UTF-8'de 2 bayt yer kaplar. `uzunluk equ $ - mesaj` bayt sayar, bu yüzden uzunluk otomatik doğru çıkar. Elle sayıp yazsaydın hata yapardın.

### Programı strace ile izle

`strace`, programın yaptığı tüm sistem çağrılarını gösterir:

```bash
strace ./merhaba
```

Çıktının sonunda şunları göreceksin:

```
write(1, "Merhaba, D\303\274nya!\n", 17Merhaba, Dünya!
) = 17
exit(0)                                 = ?
```

Yazdığın her şey birebir karşılığını bulur: `write` çağrısı, 17 bayt, `exit(0)`. Programın başka hiçbir şey yapmıyor. `ü` harfinin `\303\274` olarak iki bayt görünmesine dikkat et.

### Hata mesajı için stderr

`rdi = 2` yaparsan mesaj stderr'e gider. Terminalde ikisi aynı görünür ama ayrılabilir:

```bash
./merhaba 2>/dev/null     # stderr'e yazsaydık mesaj görünmezdi
```

### Klavyeden okuma: ismini sor, selamla

Şimdi `read` ile klavyeden veri alalım:

```nasm
section .data
    soru      db "Adin nedir? "
    soru_len  equ $ - soru
    selam     db "Merhaba, "
    selam_len equ $ - selam

section .bss
    isim resb 64                ; okunan ismi koyacağımız 64 baytlık tampon

section .text
    global _start

_start:
    ; soruyu yaz
    mov rax, 1
    mov rdi, 1
    mov rsi, soru
    mov rdx, soru_len
    syscall

    ; ismi oku
    mov rax, 0
    mov rdi, 0
    mov rsi, isim
    mov rdx, 64
    syscall
    mov r12, rax                ; okunan bayt sayısını sakla (rax'i birazdan bozacağız)

    ; "Merhaba, " yaz
    mov rax, 1
    mov rdi, 1
    mov rsi, selam
    mov rdx, selam_len
    syscall

    ; ismi yaz
    mov rax, 1
    mov rdi, 1
    mov rsi, isim
    mov rdx, r12                ; okunan bayt kadarını yaz
    syscall

    ; çık
    mov rax, 60
    xor rdi, rdi
    syscall
```

Dikkat edilecek noktalar:

- `read`, kullanıcı **Enter**'a basana kadar bekler. Enter tuşu da (`\n`, 10) okunan veriye dahildir. Bu yüzden son satır sonunu ayrıca yazmamıza gerek kalmadı.
- `rax`'teki dönüş değeri **gerçekte okunan bayt sayısıdır.** Onu `r12`'ye koyduk, çünkü bir sonraki `mov rax, 1` `rax`'i ezecek. (`r12`'yi `syscall` bozmaz, `rcx`'i bozardı.)
- `.bss` bölümü (`resb`) başlangıç değersiz tampon için idealdir (Bölüm 5).

### Alıştırmalar

1. Mesajı kendi adınla değiştir.
2. İki satır çıktı veren bir program yaz (iki ayrı `write` ile veya mesajın içine bir `10` daha koyarak).
3. Kullanıcıdan bir cümle okuyup aynen ekrana yazan **echo** programı yaz.
4. `fd = 2` ile stderr'e mesaj yazdır. `./program 2>/dev/null` ile mesajın gizlendiğini doğrula.
5. `strace` ile ismi soran programın sistem çağrılarını izle. `read`'in `rax`'te döndürdüğü sayıyı yorumla.
6. `objdump -d -M intel merhaba` çıktısında `mov rsi, mesaj` satırının nasıl göründüğüne bak. `mesaj`'ın adresi sayı olarak yazılmış mı?

---

[← Bölüm 8: Sistem çağrıları (syscall)](08-sistem-cagrilari-syscall.md) · [Bölüm 10: Bayraklar ve koşullu atlamalar →](../02-orta/10-bayraklar-ve-kosullu-atlamalar.md)
