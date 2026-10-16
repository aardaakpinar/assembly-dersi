# Türkçe x64 Assembly Dersi (Linux / NASM)

Sıfırdan başlayıp Linux üzerinde x86-64 assembly yazmayı öğreten, uygulamalı ve Türkçe bir ders. Her bölümde açıklama, çalıştırılabilir kod ve alıştırmalar var. Kolaydan zora doğru ilerler.

## Bu ders kimler için?

- Programlamayı (C, Python vb.) az da olsa bilenler
- Bilgisayarın alt katmanını, derleyicinin ne ürettiğini merak edenler
- Tersine mühendislik, güvenlik veya işletim sistemi geliştirmeye hazırlananlar

Assembly bilmek zorunda değilsin ama temel programlama kavramlarını (değişken, döngü, fonksiyon) bilmen işini çok kolaylaştırır.

## Gereksinimler

- Linux (Ubuntu, Debian, Fedora, Arch vb.), **x86-64** işlemci
- Windows'ta: WSL2 (Ubuntu) kullanabilirsin
- Apple Silicon (M1/M2/...) Mac'te: Docker ile `--platform linux/amd64` veya bir x86-64 Linux sanal makine/sunucu
- Terminal bilgisi (cd, ls, çalıştırma) yeterli

## Nasıl ilerlemeli?

1. Bölümleri sırayla oku. Her bölüm öncekine dayanır.
2. **Kodları kopyalama, elle yaz.** Hata yapmak öğrenmenin parçası.
3. Her bölümün sonundaki alıştırmaları yap.
4. Takıldığında GDB kullan ([Bölüm 7](01-baslangic/07-gdb-ile-hata-ayiklama.md)). Programı adım adım izlemek en iyi öğretmen.

**Ders akışı:** Önce register'ları ve temel komutları (`mov`, `add`, `mul`...) öğrenip sonuçları çıkış kodu ile (`echo $?`) göreceksin. Sonra sistem çağrılarını öğrenecek (Bölüm 8) ve ancak ardından ekrana yazı yazdıracaksın (Bölüm 9: Merhaba Dünya).

**Seviyeler:** Başlangıç (1–9) · Orta (10–16) · İleri (17–20)

## İçindekiler

### Başlangıç

1. [Assembly nedir?](01-baslangic/01-assembly-nedir.md)
2. [Ortam kurulumu ve ilk program](01-baslangic/02-ortam-kurulumu-ve-ilk-program.md)
3. [Sayı sistemleri ve bellek](01-baslangic/03-sayi-sistemleri-ve-bellek.md)
4. [Registerlar (Yazmaçlar)](01-baslangic/04-registerlar-yazmaclar.md)
5. [Veri tanımlama, MOV ve adresleme](01-baslangic/05-veri-tanimlama-mov-ve-adresleme.md)
6. [Aritmetik komutlar](01-baslangic/06-aritmetik-komutlar.md)
7. [GDB ile hata ayıklama](01-baslangic/07-gdb-ile-hata-ayiklama.md)
8. [Sistem çağrıları (syscall)](01-baslangic/08-sistem-cagrilari-syscall.md)
9. [Merhaba Dünya](01-baslangic/09-merhaba-dunya.md)

### Orta

10. [Bayraklar ve koşullu atlamalar](02-orta/10-bayraklar-ve-kosullu-atlamalar.md)
11. [Döngüler](02-orta/11-donguler.md)
12. [Bit operasyonları](02-orta/12-bit-operasyonlari.md)
13. [Diziler ve stringler](02-orta/13-diziler-ve-stringler.md)
14. [Stack (Yığın)](02-orta/14-stack-yigin.md)
15. [Fonksiyonlar ve çağrı sözleşmesi](02-orta/15-fonksiyonlar-ve-cagri-sozlesmesi.md)
16. [Makrolar ve çok dosyalı projeler](02-orta/16-makrolar-ve-cok-dosyali-projeler.md)

### İleri

17. [C ile birlikte çalışmak](03-ileri/17-c-ile-birlikte-calismak.md)
18. [Dosyalar ve komut satırı argümanları](03-ileri/18-dosyalar-ve-komut-satiri-argumanlari.md)
19. [Kayan noktalı sayılar ve SIMD](03-ileri/19-kayan-noktali-sayilar-ve-simd.md)
20. [Projeler ve çözümler](03-ileri/20-projeler-ve-cozumler.md)

### Ekler

- [Ek A: Syscall tablosu](ek/ek-a-syscall-tablosu.md)
- [Ek B: Komut kartı](ek/ek-b-komut-karti.md)
- [Ek C: Sık yapılan hatalar](ek/ek-c-yapilan-hatalar.md)
- [Ek D: Kaynaklar](ek/ek-d-kaynaklar.md)

## Katkıda bulunmak

Hata mı buldun, bir konuyu daha iyi anlatabileceğini mi düşünüyorsun? Issue açabilir veya pull request gönderebilirsin. Özellikle şu katkılar değerli:

- Yazım / anlatım düzeltmeleri
- Yeni alıştırmalar ve çözümler
- Ek bölümler (ör. ARM64'e giriş, bootloader yazımı)

## Lisans

Bu ders **CC BY-SA 4.0** altında paylaşılabilir. Kod örnekleri **MIT** lisansı ile kullanılabilir.
