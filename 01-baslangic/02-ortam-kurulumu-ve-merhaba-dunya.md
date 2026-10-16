[Ana sayfa / İçindekiler](../README.md)

# Bölüm 2: Ortam kurulumu ve Merhaba Dünya

### Kurulum

Gerekenler: **NASM** (assembler), **ld** (linker, `binutils` içinde), **gdb** (debugger), `make`.

```bash
# Debian / Ubuntu
sudo apt update
sudo apt install nasm gdb binutils build-essential

# Fedora
sudo dnf install nasm gdb binutils gcc make

# Arch
sudo pacman -S nasm gdb binutils gcc make
```

Kontrol:

```bash
nasm -v
ld -v
gdb --version
```

### Derleme süreci

Assembly dosyasından çalıştırılabilir programa giden yol iki adımdır:

```
program.asm  --(nasm)-->  program.o  --(ld)-->  program
 kaynak kod               nesne dosyası          çalıştırılabilir
```

- **Assemble:** NASM, metni makine koduna çevirir ve `.o` (nesne) dosyası üretir.
- **Link:** `ld`, nesne dosyalarını birleştirir, adresleri çözer, çalıştırılabilir ELF dosyası üretir.

### İlk program

`merhaba.asm` dosyasını oluştur:

```nasm
section .data
    mesaj  db "Merhaba, Dünya!", 10    ; 10 = satır sonu (\n)
    uzunluk equ $ - mesaj              ; mesajın bayt cinsinden uzunluğu

section .text
    global _start

_start:
    ; write(1, mesaj, uzunluk)
    mov rax, 1          ; syscall numarası: write
    mov rdi, 1          ; dosya tanımlayıcı: 1 = stdout (ekran)
    mov rsi, mesaj      ; yazılacak verinin adresi
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
```

Çıktı: `Merhaba, Dünya!`

Çıkış kodunu görmek için:

```bash
echo $?     # 0 yazmalı
```

### Satır satır açıklama

| Satır                   | Anlamı                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------ |
| `section .data`         | Başlangıç değeri olan verilerin bölümü                                               |
| `mesaj db "..."`        | `mesaj` etiketiyle bayt (`db`) dizisi tanımlar                                       |
| `10`                    | ASCII satır sonu karakteri                                                           |
| `uzunluk equ $ - mesaj` | `$` şu anki adres. Fark, mesajın uzunluğudur. Sabit olarak derleme anında hesaplanır |
| `section .text`         | Çalıştırılacak kodun bölümü                                                          |
| `global _start`         | `_start` etiketini linker'a görünür yapar. Programın giriş noktası budur             |
| `syscall`               | İşletim sistemi çekirdeğine istek gönderir                                           |

> **Not:** `ü` gibi Türkçe karakterler UTF-8'de 2 bayt tutar. `uzunluk equ $ - mesaj` bayt sayar, bu yüzden uzunluk otomatik doğru çıkar. Elle sayıp yazsaydın hata yapardın.

### Programı bitirmeyi unutursan?

`exit` çağrısını koymazsan işlemci `syscall` komutundan sonraki baytları (rastgele veri) komut sanıp çalıştırmaya devam eder ve program **segmentation fault** ile çöker. Assembly'de programı sonlandırmak senin görevindir.

### Kullanışlı araçlar

```bash
objdump -d -M intel merhaba       # makine kodunu assembly olarak göster
readelf -h merhaba                # ELF başlığını göster
file merhaba                      # dosya türü
strace ./merhaba                  # programın yaptığı syscall'ları izle
```

`strace` çıktısında `write(1, "Merhaba, D\303\274nya!\n", 17) = 17` ve `exit(0)` satırlarını göreceksin. Yazdığın şeyin birebir karşılığı.

### Alıştırmalar

1. Mesajı kendi adınla değiştir.
2. İki satır çıktı veren bir program yaz (iki ayrı `write` veya mesaja `10` ekleyerek).
3. Çıkış kodunu `42` yap ve `echo $?` ile doğrula.
4. `objdump -d -M intel merhaba` çıktısında kendi komutlarını bul.

---

[← Bölüm 1: Assembly nedir?](01-assembly-nedir.md) · [Bölüm 3: Sayı sistemleri ve bellek →](03-sayi-sistemleri-ve-bellek.md)
