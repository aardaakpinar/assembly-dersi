[Ana sayfa / İçindekiler](../README.md)

# Ek B: Komut kartı

**Veri taşıma**

| Komut                        | Açıklama                              |
| ---------------------------- | ------------------------------------- |
| `mov a, b`                   | a ← b                                 |
| `movzx` / `movsx` / `movsxd` | sıfır / işaret genişletmeli kopyalama |
| `lea a, [adr]`               | a ← adres (bellekten okumaz)          |
| `xchg a, b`                  | takas                                 |
| `push x` / `pop x`           | stack'e koy / stack'ten al            |

**Aritmetik**

| Komut          | Açıklama                                         |
| -------------- | ------------------------------------------------ |
| `add` / `sub`  | toplama / çıkarma                                |
| `adc` / `sbb`  | elde ile toplama / ödünçle çıkarma               |
| `inc` / `dec`  | 1 artır / azalt                                  |
| `neg`          | işaret değiştir                                  |
| `mul` / `imul` | işaretsiz / işaretli çarpma                      |
| `div` / `idiv` | işaretsiz / işaretli bölme (`rdx:rax` ÷ operand) |
| `cqo`          | `rax` işaretini `rdx`'e yay                      |

**Mantık ve bit**

| Komut                           | Açıklama                               |
| ------------------------------- | -------------------------------------- |
| `and` / `or` / `xor` / `not`    | bit işlemleri                          |
| `shl` / `shr` / `sar`           | kaydırma                               |
| `rol` / `ror`                   | döndürme                               |
| `test a, b`                     | `a & b` (sadece bayrak)                |
| `bt` / `bts` / `btr` / `btc`    | bit testi / set / reset / toggle       |
| `popcnt`, `bsf`, `bsr`, `bswap` | bit sayma, bit arama, bayt sırası ters |

**Karşılaştırma ve atlama**

| Komut                              | Açıklama                                  |
| ---------------------------------- | ----------------------------------------- |
| `cmp a, b`                         | `a - b` (sadece bayrak)                   |
| `jmp`                              | koşulsuz atla                             |
| `je`/`jne`                         | eşit / eşit değil                         |
| `jg`/`jge`/`jl`/`jle`              | işaretli >, ≥, <, ≤                       |
| `ja`/`jae`/`jb`/`jbe`              | işaretsiz >, ≥, <, ≤                      |
| `js`/`jns`, `jc`/`jnc`, `jo`/`jno` | işaret / elde / taşma bayrağına göre      |
| `cmovcc`                           | koşullu mov                               |
| `setcc`                            | koşula göre 0/1                           |
| `loop`                             | `rcx`'i azalt, sıfır değilse atla (yavaş) |

**Fonksiyon**

| Komut     | Açıklama                          |
| --------- | --------------------------------- |
| `call f`  | dönüş adresini push et, f'ye atla |
| `ret`     | dönüş adresini pop et, oraya atla |
| `leave`   | `mov rsp, rbp; pop rbp`           |
| `syscall` | çekirdeğe çağrı                   |

**String**

| Komut                  | Açıklama                        |
| ---------------------- | ------------------------------- |
| `movsb/w/d/q`          | `[rdi] ← [rsi]`                 |
| `stosb/w/d/q`          | `[rdi] ← al/ax/eax/rax`         |
| `lodsb/w/d/q`          | `al/ax/eax/rax ← [rsi]`         |
| `scasb/w/d/q`          | `al` ile `[rdi]` karşılaştır    |
| `cmpsb/w/d/q`          | `[rsi]` ile `[rdi]` karşılaştır |
| `rep`, `repe`, `repne` | `rcx` kez tekrarla (koşullu)    |
| `cld` / `std`          | yönü ileri / geri yap           |

**NASM direktifleri**

| Direktif                           | Açıklama                              |
| ---------------------------------- | ------------------------------------- |
| `section .text/.data/.bss/.rodata` | bölüm seç                             |
| `global x` / `extern x`            | sembolü dışarı aç / dışarıdan al      |
| `db`/`dw`/`dd`/`dq`                | veri tanımla                          |
| `resb`/`resw`/`resd`/`resq`        | boş yer ayır                          |
| `equ`                              | sabit                                 |
| `times n x`                        | x'i n kere tekrarla                   |
| `$`                                | şu anki adres                         |
| `%macro` / `%endmacro`             | makro                                 |
| `%include "dosya"`                 | dosya dahil et                        |
| `default rel`                      | RIP-göreli adreslemeyi varsayılan yap |

---

[← Ek A: Syscall tablosu](ek-a-syscall-tablosu.md) · [Ek C: Sık yapılan hatalar →](ek-c-sik-yapilan-hatalar.md)
