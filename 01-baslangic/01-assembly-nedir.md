[Ana sayfa / İçindekiler](../README.md)

# Bölüm 1: Assembly nedir?

### İşlemci ne anlar?

İşlemci (CPU) yalnızca **makine kodu** anlar: bellekte duran ve belirli bir anlamı olan baytlar dizisi. Örneğin x86-64 işlemcide şu baytlar "rax register'ına 60 yükle" demektir:

```
48 C7 C0 3C 00 00 00
```

İnsan için okunamaz. **Assembly**, bu makine kodunun insan tarafından okunabilir karşılığıdır:

```nasm
mov rax, 60
```

Assembly ile makine kodu arasında (neredeyse) birebir eşleşme vardır. Bir **assembler** (bu derste NASM) assembly metnini makine koduna çevirir.

### Diller arasındaki konum

```
Python / JavaScript   → Çok yüksek seviye (yorumlayıcı)
C / C++ / Rust / Go   → Yüksek seviye (derleyici → makine kodu)
Assembly              → Makine kodunun metin hali
Makine kodu           → CPU'nun çalıştırdığı baytlar
```

C'de yazdığın `x = a + b;` satırı derleyici tarafından birkaç assembly komutuna çevrilir. Assembly'de bu çeviriyi sen yaparsın.

### Neden öğrenmeli?

- **Bilgisayarı gerçekten anlamak:** Pointer, stack, çağrı sözleşmesi, bellek düzeni artık soyut kavramlar olmaktan çıkar.
- **Tersine mühendislik ve güvenlik:** Zararlı yazılım analizi, açık araştırması, CTF yarışmaları assembly bilgisi ister.
- **Performans ve gömülü sistemler:** Kritik döngüleri optimize etmek, işletim sistemi çekirdeği, bootloader yazmak.
- **Hata ayıklama:** Derleyicinin ne ürettiğine bakarak anlaşılmaz hataları çözebilirsin.

### Assembly'nin dezavantajları

- Taşınabilir değil: x86-64 assembly ARM işlemcide çalışmaz.
- Yazması uzun ve hataya açık.
- Günlük yazılım geliştirmede nadiren gerekir.

### Assembly türleri (sözdizimi)

x86 için iki ana sözdizimi var:

|                | **Intel sözdizimi**         | **AT&T sözdizimi**        |
| -------------- | --------------------------- | ------------------------- |
| Örnek          | `mov rax, 5`                | `movq $5, %rax`           |
| Operand sırası | **hedef, kaynak**           | kaynak, hedef             |
| Kullanan       | NASM, MASM, Intel belgeleri | GNU `as`, GCC varsayılanı |

**Bu derste Intel sözdizimi ve NASM assembler'ı kullanıyoruz.** Operand sırasını aklında tut: **önce hedef, sonra kaynak** (`mov hedef, kaynak`).

### Alıştırmalar

1. `mov rbx, 10` komutunda hedef ve kaynak nedir?
2. Neden x86-64 assembly bir Raspberry Pi'de (ARM) çalışmaz?

---

[← Ana sayfa](../README.md) · [Bölüm 2: Ortam kurulumu ve ilk program →](02-ortam-kurulumu-ve-ilk-program.md)
