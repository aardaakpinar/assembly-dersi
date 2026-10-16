[Ana sayfa / İçindekiler](../README.md)

# Bölüm 7: GDB ile hata ayıklama

Assembly'de en değerli beceri: **programı komut komut izleyip register'ları ve belleği görmek.** Bunu erken öğrenmek seni çok hızlandırır.

### Hata ayıklama bilgisiyle derle

```bash
nasm -f elf64 -g -F dwarf program.asm -o program.o
ld program.o -o program
```

`-g -F dwarf` GDB'ye kaynak satırı bilgisi ekler.

### GDB'yi Intel sözdizimine ayarla

GDB varsayılan olarak AT&T gösterir. Kalıcı olarak Intel yapmak için:

```bash
echo "set disassembly-flavor intel" >> ~/.gdbinit
```

### Temel komutlar

```bash
gdb ./program
```

| Komut                  | Kısaltma   | Ne yapar                                     |
| ---------------------- | ---------- | -------------------------------------------- |
| `break _start`         | `b _start` | `_start`'a durak noktası koy                 |
| `break program.asm:12` |            | 12. satıra durak koy                         |
| `run`                  | `r`        | Programı çalıştır                            |
| `starti`               |            | İlk komutta dur                              |
| `stepi`                | `si`       | **Tek komut** ilerle (`call`'un içine girer) |
| `nexti`                | `ni`       | Tek komut ilerle (`call`'u atlar)            |
| `continue`             | `c`        | Sonraki durak noktasına kadar çalış          |
| `info registers`       | `i r`      | Tüm register'ları göster                     |
| `info registers rax`   |            | Sadece `rax`                                 |
| `print $rax`           | `p $rax`   | `rax`'ı onluk göster                         |
| `print/x $rax`         | `p/x $rax` | Hex göster                                   |
| `print/t $al`          |            | İkilik göster                                |
| `x/8xb &mesaj`         |            | Bellekten 8 bayt, hex                        |
| `x/4xg $rsp`           |            | Stack'ten 4 quadword, hex                    |
| `x/s &mesaj`           |            | String olarak göster                         |
| `x/5i $rip`            |            | Sıradaki 5 komutu göster                     |
| `disassemble`          | `disas`    | Geçerli bölgeyi göster                       |
| `layout asm`           |            | Üstte kaynak/asm penceresi (TUI)             |
| `layout regs`          |            | Register penceresi                           |
| `quit`                 | `q`        | Çık                                          |

`x` komutunun biçimi: `x/<adet><biçim><boyut> <adres>`

- biçim: `x` hex, `d` onluk, `u` işaretsiz, `t` ikilik, `c` karakter, `s` string, `i` komut
- boyut: `b` bayt, `h` 2 bayt, `w` 4 bayt, `g` 8 bayt

### Uygulamalı oturum

Şu programı alalım (`gdb_deneme.asm`):

```nasm
section .data
    sayi dd 0x12345678

section .text
    global _start

_start:
    mov eax, [sayi]
    mov rbx, 10
    add rax, rbx
    mov rdi, 0
    mov rax, 60
    syscall
```

GDB oturumu:

```
$ gdb ./gdb_deneme
(gdb) break _start
(gdb) run
(gdb) x/4xb &sayi            # little-endian'ı gör: 0x78 0x56 0x34 0x12
(gdb) stepi
(gdb) p/x $rax               # 0x12345678
(gdb) stepi
(gdb) stepi
(gdb) p/x $rax               # 0x12345682
(gdb) info registers rflags  # bayrakları gör
```

### Faydalı püf noktaları

- `layout regs` ile sürekli görünen register penceresi aç. `si` ile adım at, değişen register'ları izle.
- Ekran bozulursa `Ctrl+L` ile yenile.
- **GEF** veya **pwndbg** gibi eklentiler çok güzel bir arayüz sunar (isteğe bağlı).
- Bölüm 9'da `strace` ile programın sistem çağrılarını izlemeyi de göreceksin.

### Alıştırmalar

1. Bölüm 6'daki `(a + b) * c - d` programını GDB'de adım adım izle. Her adımda `rax` ne oluyor?
2. Bölüm 5'teki toplama programında `x/gd &sayi1` ile bellekteki değeri oku.
3. `stepi` ile ilerleyip `syscall` komutuna gel. Tam o noktada `info registers rax rdi` ile hangi sistem çağrısının, hangi çıkış koduyla yapılacağını gör.

---

[← Bölüm 6: Aritmetik komutlar](06-aritmetik-komutlar.md) · [Bölüm 8: Sistem çağrıları (syscall) →](08-sistem-cagrilari-syscall.md)
