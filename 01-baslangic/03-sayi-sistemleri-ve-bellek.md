[Ana sayfa / İçindekiler](../README.md)

# Bölüm 3: Sayı sistemleri ve bellek

### Bit ve bayt

- **Bit:** 0 veya 1
- **Bayt (byte):** 8 bit → 0–255 arası değer
- Bu derste kullanacağımız boyut adları:

| Ad         | NASM | Boyut           | İşaretsiz aralık               |
| ---------- | ---- | --------------- | ------------------------------ |
| Byte       | `db` | 8 bit (1 bayt)  | 0 – 255                        |
| Word       | `dw` | 16 bit (2 bayt) | 0 – 65.535                     |
| Doubleword | `dd` | 32 bit (4 bayt) | 0 – 4.294.967.295              |
| Quadword   | `dq` | 64 bit (8 bayt) | 0 – 18.446.744.073.709.551.615 |

> "Word" adı 16 bit olan eski işlemcilerden kalmadır. x64'te doğal boyut 64 bit olsa da isimlendirme değişmedi.

### İkilik ve onaltılık sistem

Bilgisayar ikilik (binary) çalışır. Onaltılık (hex) ise ikilik sayıların kısa yazımıdır: **4 bit = 1 hex basamağı**.

| Onluk | İkilik | Hex |
| ----- | ------ | --- |
| 0     | 0000   | 0   |
| 5     | 0101   | 5   |
| 9     | 1001   | 9   |
| 10    | 1010   | A   |
| 11    | 1011   | B   |
| 12    | 1100   | C   |
| 13    | 1101   | D   |
| 14    | 1110   | E   |
| 15    | 1111   | F   |

Örnek: `0xA5` = `1010 0101` = 165 (onluk).

NASM'de sayı yazma biçimleri:

```nasm
mov rax, 255        ; onluk
mov rax, 0xFF       ; hex
mov rax, 0b11111111 ; ikilik
mov rax, 0o377      ; sekizlik
mov al, 'A'         ; karakter (ASCII 65)
```

### Negatif sayılar: ikiye tümleyen (two's complement)

Bilgisayarlar negatif sayıları **ikiye tümleyen** yöntemiyle saklar. Bir sayının negatifini bulmak için: tüm bitleri tersle, sonra 1 ekle.

8 bit ile `-5` bulalım:

```
 5  = 0000 0101
ters= 1111 1010
+1  = 1111 1011  → -5
```

Bu yöntemin güzelliği: toplama devresi hem pozitif hem negatif sayılarda aynı çalışır. En soldaki bit **işaret biti**dir (1 ise negatif).

| 8 bit hex | İşaretsiz | İşaretli |
| --------- | --------- | -------- |
| `0x00`    | 0         | 0        |
| `0x7F`    | 127       | 127      |
| `0x80`    | 128       | -128     |
| `0xFF`    | 255       | -1       |

**Önemli:** Aynı bit düzeni, işaretli mi işaretsiz mi yorumlanacağına göre farklı sayı olur. İşlemci bunu bilmez. Hangi yorumu kullanacağını **komut seçimin** belirler (ör. `ja` işaretsiz, `jg` işaretli karşılaştırma için, Bölüm 9).

### Bellek

Bellek, her baytın bir **adresi** olan çok uzun bir bayt dizisidir.

```
Adres      İçerik
0x1000     0x48
0x1001     0x65
0x1002     0x6C
0x1003     0x6C
...
```

x64'te adresler 64 bit genişliğindedir (pratikte 48 bit kullanılır).

### Little-endian: baytlar ters dizilir

x86 işlemciler **little-endian**'dır: çok baytlı bir sayının **en düşük değerlikli baytı en düşük adrese** yazılır.

`0x12345678` sayısını (32 bit) belleğe yazarsan:

```
Adres   : +0   +1   +2   +3
İçerik  : 78   56   34   12
```

Bunu GDB ile görmeyi Bölüm 7'de deneyeceksin. Şimdilik bilmen gereken: bellekte bir sayıyı bayt bayt okursan "ters" görürsün, bu normaldir.

### Bir programın bellek düzeni

Linux'ta çalışan bir programın belleği kabaca şöyle bölünür:

```
Yüksek adres  ┌──────────────────┐
              │      Stack       │  ↓ aşağı doğru büyür (yerel değişkenler, dönüş adresleri)
              │        ↓         │
              │                  │
              │        ↑         │
              │      Heap        │  ↑ yukarı büyür (dinamik bellek)
              ├──────────────────┤
              │ .bss             │  Başlangıç değeri olmayan veriler (sıfırlanır)
              │ .data            │  Başlangıç değeri olan veriler
              │ .text            │  Program kodu (salt okunur + çalıştırılabilir)
Düşük adres   └──────────────────┘
```

### Alıştırmalar

1. `0xFF`'in işaretsiz ve 8 bit işaretli karşılığı nedir?
2. `-1`'i 8 bit ikiye tümleyen ile yaz.
3. `0xDEADBEEF` değeri little-endian belleğe nasıl yazılır?
4. `0b1100_1010` sayısını (NASM alt çizgiye izin verir) hex'e çevir.

<details>
<summary>Cevaplar</summary>

1. İşaretsiz 255, işaretli -1
2. `1111 1111`
3. `EF BE AD DE`
4. `0xCA`
 </details>

---

[← Bölüm 2: Ortam kurulumu ve ilk program](02-ortam-kurulumu-ve-ilk-program.md) · [Bölüm 4: Registerlar (Yazmaçlar) →](04-registerlar-yazmaclar.md)
