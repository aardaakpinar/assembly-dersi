[Ana sayfa / İçindekiler](../README.md)

# Bölüm 4: Registerlar (Yazmaçlar)

Register, işlemcinin içindeki çok hızlı küçük bellek hücresidir. İşlemci bellekle doğrudan çok işlem yapmaz. Veriyi register'a alır, orada işler, sonucu geri yazar. x64'te register'lar 64 bit genişliğindedir.

### Genel amaçlı registerlar

16 adet var:

`rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8, r9, r10, r11, r12, r13, r14, r15`

### Alt registerlar (parçalar)

Her 64 bit register'ın alt parçalarına ayrı isimle erişebilirsin:

| 64 bit | 32 bit | 16 bit | 8 bit (alt) | 8 bit (üst) |
| ------ | ------ | ------ | ----------- | ----------- |
| `rax`  | `eax`  | `ax`   | `al`        | `ah`        |
| `rbx`  | `ebx`  | `bx`   | `bl`        | `bh`        |
| `rcx`  | `ecx`  | `cx`   | `cl`        | `ch`        |
| `rdx`  | `edx`  | `dx`   | `dl`        | `dh`        |
| `rsi`  | `esi`  | `si`   | `sil`       | –           |
| `rdi`  | `edi`  | `di`   | `dil`       | –           |
| `rbp`  | `ebp`  | `bp`   | `bpl`       | –           |
| `rsp`  | `esp`  | `sp`   | `spl`       | –           |
| `r8`   | `r8d`  | `r8w`  | `r8b`       | –           |
| ...    | ...    | ...    | ...         | –           |
| `r15`  | `r15d` | `r15w` | `r15b`      | –           |

Görsel olarak `rax`:

```
 63                              32 31             16 15      8 7       0
┌──────────────────────────────────┬─────────────────┬─────────┬─────────┐
│                                  │                 │   ah    │   al    │
└──────────────────────────────────┴─────────────────┴─────────┴─────────┘
                                   │<-------- eax ----------------------->│
                                                     │<------- ax -------->│
│<---------------------------------- rax ---------------------------------->│
```

### ⚠️ Çok önemli kural: 32 bit yazma üst yarıyı sıfırlar

```nasm
mov rax, 0xFFFFFFFFFFFFFFFF
mov eax, 5            ; 32 bit yazma → rax = 0x0000000000000005 (üst 32 bit SIFIRLANIR)
```

Ama 8 ve 16 bit yazmalar üst kısma **dokunmaz**:

```nasm
mov rax, 0xFFFFFFFFFFFFFFFF
mov al, 5             ; rax = 0xFFFFFFFFFFFFFF05
mov ax, 5             ; rax = 0xFFFFFFFFFFFF0005
```

Bu yüzden derleyiciler `xor eax, eax` (2 bayt) komutunu `xor rax, rax` yerine tercih eder. Sonuç aynı, makine kodu daha kısa.

### Registerların geleneksel rolleri

Teknik olarak çoğu register istediğin gibi kullanılabilir ama bazı komutlar ve kurallar belirli register'ları bekler:

| Register      | Rol                                                                                          |
| ------------- | -------------------------------------------------------------------------------------------- |
| `rax`         | Dönüş değeri, syscall numarası, `mul`/`div` için operand                                     |
| `rcx`         | Sayaç (kaydırma `cl`, `rep` önekleri, `loop`)                                                |
| `rdx`         | `mul`/`div` için üst yarı, syscall 3. argüman                                                |
| `rsi` / `rdi` | String komutlarında kaynak / hedef, fonksiyon argümanları                                    |
| `rsp`         | **Stack pointer**: stack'in tepesini gösterir (Bölüm 13)                                     |
| `rbp`         | Base pointer: stack frame tabanı (Bölüm 14)                                                  |
| `rip`         | **Instruction pointer**: çalıştırılacak sonraki komutun adresi. Doğrudan `mov` ile yazılamaz |

### RFLAGS: bayrak register'ı

İşlem sonuçları hakkında bilgi tutan bit'lerdir. Koşullu atlamalar bunlara bakar (Bölüm 9'da ayrıntılı).

| Bayrak | Ad        | Ne zaman 1 olur?                         |
| ------ | --------- | ---------------------------------------- |
| `ZF`   | Zero      | Sonuç sıfır                              |
| `SF`   | Sign      | Sonuç negatif (en üst bit 1)             |
| `CF`   | Carry     | İşaretsiz taşma (elde)                   |
| `OF`   | Overflow  | İşaretli taşma                           |
| `PF`   | Parity    | Sonucun alt baytında 1'lerin sayısı çift |
| `DF`   | Direction | String komutları yönü (0 = ileri)        |

### Diğer registerlar (ileride)

- **xmm0–xmm15:** Kayan noktalı sayı ve SIMD için 128 bit (Bölüm 18)
- **Segment register'ları:** `cs`, `ds`, `fs`, `gs` vb. 64 bit modunda büyük ölçüde etkisiz (`fs`/`gs` iş parçacığı verisi için kullanılır)

### Alıştırmalar

1. `mov rax, -1` sonrası `mov ax, 0` yapılırsa `rax` ne olur?
2. Aynı başlangıçla `mov eax, 0` yapılırsa `rax` ne olur?
3. `r9`'un alt 8 bit parçasının adı nedir?

<details>
<summary>Cevaplar</summary>

1. `0xFFFFFFFFFFFF0000`
2. `0x0000000000000000`
3. `r9b`
 </details>

---

[← Bölüm 3: Sayı sistemleri ve bellek](03-sayi-sistemleri-ve-bellek.md) · [Bölüm 5: Veri tanımlama, MOV ve adresleme →](05-veri-tanimlama-mov-ve-adresleme.md)
