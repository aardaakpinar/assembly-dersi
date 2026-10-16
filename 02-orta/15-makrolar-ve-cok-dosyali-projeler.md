[Ana sayfa / İçindekiler](../README.md)

# Bölüm 15: Makrolar ve çok dosyalı projeler

### Sabitler: equ ve %define

```nasm
SYS_WRITE equ 1
SYS_EXIT  equ 60
STDOUT    equ 1

mov rax, SYS_WRITE
mov rdi, STDOUT
```

`%define` metin değiştirir (C'deki `#define` gibi):

```nasm
%define SATIR_SONU 10
```

### Makrolar

Tekrar eden kod parçalarını makroya çevirebilirsin. Makro, **fonksiyon çağrısı değildir**: her kullanıldığı yere kod olarak açılır.

```nasm
%macro yaz 2                 ; 2 parametre alır
    mov rax, 1
    mov rdi, 1
    mov rsi, %1              ; 1. parametre
    mov rdx, %2              ; 2. parametre
    syscall
%endmacro

%macro cikis 1
    mov rax, 60
    mov rdi, %1
    syscall
%endmacro
```

Kullanım:

```nasm
section .data
    mesaj db "Merhaba makro!", 10
    uzun  equ $ - mesaj

section .text
    global _start

_start:
    yaz mesaj, uzun
    cikis 0
```

Dikkat: Bu makro `rax`, `rdi`, `rsi`, `rdx`, `rcx`, `r11` registerlarını bozar. Makro içinde etiket gerekiyorsa **makro-yerel etiket** (`%%`) kullan ki her açılışta farklı isim alsın:

```nasm
%macro bekle_sifir 1
%%dongu:
    dec %1
    jnz %%dongu
%endmacro
```

### Dosya dahil etme

```nasm
%include "makrolar.inc"      ; başka bir dosyanın içeriğini buraya kopyala
```

### Çok dosyalı proje

Büyük projeleri birden fazla `.asm` dosyasına böl. Dosyalar arası paylaşım için `global` (dışarıya aç) ve `extern` (dışarıdan al) kullanılır.

**`kutuphane.asm`:**

```nasm
section .bss
    tampon resb 21

section .text
    global yazdir_sayi          ; dışarıya aç

yazdir_sayi:
    mov rax, rdi
    lea rsi, [tampon + 20]
    mov byte [rsi], 10
    mov rcx, 10
.basamak:
    xor rdx, rdx
    div rcx
    add dl, '0'
    dec rsi
    mov [rsi], dl
    test rax, rax
    jnz .basamak
    lea rdx, [tampon + 21]
    sub rdx, rsi
    mov rax, 1
    mov rdi, 1
    syscall
    ret
```

**`ana.asm`:**

```nasm
    extern yazdir_sayi          ; başka dosyadan gelecek

section .text
    global _start

_start:
    mov rdi, 123456789
    call yazdir_sayi
    mov rax, 60
    xor rdi, rdi
    syscall
```

Derleme:

```bash
nasm -f elf64 ana.asm -o ana.o
nasm -f elf64 kutuphane.asm -o kutuphane.o
ld ana.o kutuphane.o -o program
./program
```

### Makefile

Her seferinde komut yazmamak için:

```make
ASM      = nasm
ASMFLAGS = -f elf64 -g -F dwarf
LD       = ld

all: program

program: ana.o kutuphane.o
	$(LD) ana.o kutuphane.o -o program

%.o: %.asm
	$(ASM) $(ASMFLAGS) $< -o $@

clean:
	rm -f *.o program
```

(Satır başlarındaki girinti **TAB** karakteri olmalı, boşluk değil.) Kullanım: `make`, `make clean`.

### Alıştırmalar

1. `yaz`, `cikis` ve `satir_sonu` makrolarını bir `makrolar.inc` dosyasına koy ve `%include` ile kullan.
2. `yazdir_sayi`, `yazdir_string`, `strlen`, `atoi` fonksiyonlarından oluşan `kutuphane.asm` yaz. Ana programı ayrı dosyada tut. Makefile ekle.

---

[← Bölüm 14: Fonksiyonlar ve çağrı sözleşmesi](14-fonksiyonlar-ve-cagri-sozlesmesi.md) · [Bölüm 16: C ile birlikte çalışmak →](../03-ileri/16-c-ile-birlikte-calismak.md)
