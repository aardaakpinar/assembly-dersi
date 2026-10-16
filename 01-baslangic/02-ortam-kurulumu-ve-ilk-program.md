[Ana sayfa / İçindekiler](../README.md)

# Bölüm 2: Ortam kurulumu ve ilk program

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

### İlk program: bir sayı döndürmek

Ekrana yazı yazdırmak için işletim sisteminin yardımı gerekir. Bunu birkaç bölüm sonra (Bölüm 8 ve 9) öğreneceğiz. Önce daha basit ve çok öğretici bir şey yapalım: programımız **bir sayı hesaplasın ve bunu çıkış kodu (exit code) olarak döndürsün.**

Linux'ta her program biterken işletim sistemine 0–255 arası bir sayı bırakır. Buna **çıkış kodu** denir. Terminalde son çalışan programın çıkış kodunu `echo $?` komutuyla görebilirsin. Ekrana bir şey yazdırmadan sonucu görmenin en basit yolu budur. Sonraki birkaç bölümde hesaplarımızın sonucunu hep böyle okuyacağız.

`ilk.asm` dosyasını oluştur:

```nasm
section .text
    global _start

_start:
    mov rdi, 42         ; çıkış kodu olarak döndürmek istediğimiz sayı
    mov rax, 60         ; "programı bitir" işlemi (exit)
    syscall             ; işletim sistemine bildir
```

Derle, çalıştır, sonucu oku:

```bash
nasm -f elf64 ilk.asm -o ilk.o
ld ilk.o -o ilk
./ilk
echo $?             # 42
```

Ekrana hiçbir şey yazılmaz ama `echo $?` **42** gösterir. Bu, bilgisayara ilk kez kendi komutunla bir değer hesaplattığın an.

> **Şimdilik kalıp olarak al:** `mov rax, 60` ve `syscall` satırları programı bitirir ve `rdi`'ye koyduğun sayı çıkış kodu olur. Bunların neden böyle çalıştığını Bölüm 8'de (sistem çağrıları) anlatacağız. Şimdilik kural şu: **programın sonuna bu iki satırı ekle, sonucu `rdi`'ye koy.**

### Satır satır açıklama

| Satır           | Anlamı                                                                   |
| --------------- | ------------------------------------------------------------------------ |
| `section .text` | Çalıştırılacak komutların bölümü                                         |
| `global _start` | `_start` etiketini linker'a görünür yapar. Programın giriş noktası budur |
| `_start:`       | Etiket (label). Programın başladığı yeri işaretler                       |
| `mov rdi, 42`   | `rdi` register'ına 42 koy. Register'ları Bölüm 4'te tanıyacağız          |
| `mov rax, 60`   | `rax` register'ına 60 koy (60 = "bitir" işleminin numarası)              |
| `syscall`       | İşletim sistemini çağır                                                  |
| `; ...`         | Yorum satırı. Assembler yok sayar                                        |

`mov` komutunun yazımı hep aynı: **`mov hedef, kaynak`**. Soldaki yer, sağdaki değeri alır.

### Çıkış satırlarını unutursan

Son iki satırı (`mov rax, 60` ve `syscall`) silip programı çalıştırmayı dene. **Segmentation fault** alırsın. İşlemci programın bittiğini bilmez. Komutlar bitince bellekte arkada duran rastgele baytları komut sanıp çalıştırmaya devam eder ve çöker. Assembly'de programı sonlandırmak senin görevindir.

### Dene

```nasm
    mov rdi, 7          ; echo $? → 7
    mov rdi, 255        ; echo $? → 255
    mov rdi, 300        ; echo $? → 44  (çıkış kodu 256'ya göre mod alınır: 300 - 256)
    mov rdi, -1         ; echo $? → 255
```

Çıkış kodu **8 bitlik** olduğu için 0–255 aralığında kalır. Bu sınır yüzünden büyük sayıları bu yöntemle göremeyiz. İleride ekrana sayı yazdırmayı da öğreneceğiz.

### Kullanışlı araçlar

```bash
objdump -d -M intel ilk       # makine kodunu assembly olarak göster
readelf -h ilk                # ELF başlığını göster
file ilk                      # dosya türü
```

`objdump` çıktısında yazdığın komutların yanında makine kodu baytlarını göreceksin (Bölüm 1'deki "assembly = makine kodunun okunabilir hali" fikrinin kanıtı). Küçük bir sürpriz: NASM `mov rdi, 42` komutunu `mov edi, 0x2a` olarak üretmiş olabilir. 32 bit yazma üst yarıyı zaten sıfırladığı için (Bölüm 4) sonuç aynı, makine kodu ise daha kısadır.

### Alıştırmalar

1. Çıkış kodunu 100 yap ve `echo $?` ile doğrula.
2. 256 ve 257 değerlerini dene. Ne görüyorsun? Neden?
3. `objdump -d -M intel ilk` çıktısında kendi üç komutunu bul.
4. Programdaki `global _start` satırını sil, derle ve bağla (`ld`). Hangi uyarıyı alıyorsun?

---

[← Bölüm 1: Assembly nedir?](01-assembly-nedir.md) · [Bölüm 3: Sayı sistemleri ve bellek →](03-sayi-sistemleri-ve-bellek.md)
