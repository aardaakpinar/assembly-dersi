[Ana sayfa / İçindekiler](../README.md)

# Bölüm 18: Dosyalar ve komut satırı argümanları

### Dosya işlemleri için syscall'lar

| Numara | Ad      | rdi        | rsi       | rdx         |
| ------ | ------- | ---------- | --------- | ----------- |
| 2      | `open`  | dosya yolu | bayraklar | izinler     |
| 3      | `close` | fd         |           |             |
| 0      | `read`  | fd         | tampon    | bayt sayısı |
| 1      | `write` | fd         | veri      | bayt sayısı |
| 8      | `lseek` | fd         | konum     | nereden     |

**`open` bayrakları** (sayısal değerler, x86-64 Linux):

| Bayrak     | Değer        |
| ---------- | ------------ |
| `O_RDONLY` | 0            |
| `O_WRONLY` | 1            |
| `O_RDWR`   | 2            |
| `O_CREAT`  | 0x40 (64)    |
| `O_TRUNC`  | 0x200 (512)  |
| `O_APPEND` | 0x400 (1024) |

Bayraklar `or` ile birleştirilir: yazmak için aç + yoksa oluştur + içeriği sil = `1 | 0x40 | 0x200 = 0x241`.

### Örnek: dosyaya yazma

```nasm
section .data
    dosya   db "cikti.txt", 0          ; yol, 0 ile sonlanır
    veri    db "Assembly ile yazildi!", 10
    veri_len equ $ - veri

section .text
    global _start

_start:
    ; fd = open("cikti.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644)
    mov rax, 2
    lea rdi, [dosya]
    mov rsi, 0x241
    mov rdx, 0o644                      ; dosya izinleri (sekizlik)
    syscall

    test rax, rax
    js .hata                            ; negatifse hata
    mov r12, rax                        ; fd'yi sakla

    ; write(fd, veri, veri_len)
    mov rax, 1
    mov rdi, r12
    lea rsi, [veri]
    mov rdx, veri_len
    syscall

    ; close(fd)
    mov rax, 3
    mov rdi, r12
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall

.hata:
    mov rax, 60
    mov rdi, 1                          ; hata çıkış kodu
    syscall
```

```bash
./program && cat cikti.txt
```

### Örnek: dosya okuma ve ekrana yazma

```nasm
section .data
    dosya db "cikti.txt", 0

section .bss
    tampon resb 256

section .text
    global _start

_start:
    mov rax, 2                  ; open(dosya, O_RDONLY)
    lea rdi, [dosya]
    xor rsi, rsi
    xor rdx, rdx
    syscall
    test rax, rax
    js .hata
    mov r12, rax

    mov rax, 0                  ; read(fd, tampon, 256)
    mov rdi, r12
    lea rsi, [tampon]
    mov rdx, 256
    syscall
    mov r13, rax                ; okunan bayt sayısı

    mov rax, 1                  ; write(1, tampon, okunan)
    mov rdi, 1
    lea rsi, [tampon]
    mov rdx, r13
    syscall

    mov rax, 3                  ; close(fd)
    mov rdi, r12
    syscall

    xor rdi, rdi
    jmp .cik
.hata:
    mov rdi, 1
.cik:
    mov rax, 60
    syscall
```

Dosya yoksa `open` `-2` (ENOENT) döndürür ve program `1` ile çıkar. `strace` ile başarısız `open` çağrısını görebilirsin.

### Komut satırı argümanları

Program başlarken stack'in yapısı (Bölüm 14'te görmüştük):

```
[rsp]       = argc
[rsp + 8]   = argv[0]  (program adı)
[rsp + 16]  = argv[1]  (ilk argüman)
[rsp + 24]  = argv[2]
...
```

`argv[i]` bir **pointer**'dır (null-terminated string adresi).

**Örnek: ilk argümanı ekrana yazdır** (`./program Merhaba` → `Merhaba`):

```nasm
section .data
    yeni db 10

section .text
    global _start

_start:
    mov rax, [rsp]              ; argc
    cmp rax, 2
    jl .cikis                   ; argüman yoksa çık

    mov rsi, [rsp + 16]         ; rsi = argv[1] adresi
    xor rdx, rdx                ; uzunluk = 0
.say:
    cmp byte [rsi + rdx], 0
    je .yaz
    inc rdx
    jmp .say

.yaz:
    mov rax, 1
    mov rdi, 1
    syscall                     ; rsi (adres) ve rdx (uzunluk) hazır

    mov rax, 1
    mov rdi, 1
    lea rsi, [yeni]
    mov rdx, 1
    syscall

.cikis:
    mov rax, 60
    xor rdi, rdi
    syscall
```

### Ortam değişkenleri (ileri bilgi)

`argv` dizisi bir `NULL` işaretçisiyle biter, hemen arkasından `envp` (ortam değişkenleri) dizisi başlar. `[rsp + 8 + (argc+1)*8]` adresinde ilk ortam değişkeni pointer'ı bulunur.

### Alıştırmalar

1. Komut satırından verilen dosyayı (argv[1]) açıp ekrana basan `cat` benzeri program yaz.
2. Argüman sayısını (`argc`) `yazdir_sayi` ile ekrana yazdır.
3. Bir dosyaya kullanıcıdan alınan bir satırı ekleyen (`O_APPEND`) program yaz.
4. Dosya yoksa ekrana (stderr'e) "Dosya bulunamadi" yazdır.

---

[← Bölüm 17: C ile birlikte çalışmak](17-c-ile-birlikte-calismak.md) · [Bölüm 19: Kayan noktalı sayılar ve SIMD →](19-kayan-noktali-sayilar-ve-simd.md)
