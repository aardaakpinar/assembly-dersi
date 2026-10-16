[Ana sayfa / İçindekiler](../README.md)

# Ek C: Sık yapılan hatalar

| Belirti                                                                       | Olası neden                                                                                                                  |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Program çalışır ama **Segmentation fault** ile biter                          | `exit` syscall'ını koymamışsındır; ya da `ret`/`pop` ile stack'i bozmuşsundur                                                |
| `Segmentation fault` bellek erişiminde                                        | Geçersiz adrese erişim. `mov rax, [rbx]` ile `rbx` sıfır / geçersiz. GDB'de `p/x $rbx` kontrol et                            |
| Salt okunur bölüme yazma segfault'u                                           | `.rodata` veya `.text` içine yazmaya çalışıyorsun. Değişecek veriyi `.data` / `.bss`'e koy                                   |
| `error: operation size not specified`                                         | `mov [x], 5` yazdın. `mov dword [x], 5` gibi boyut ekle                                                                      |
| `error: invalid combination of opcode and operands`                           | Bellekten belleğe `mov`, boyutları uyuşmayan register'lar veya geçersiz operand kombinasyonu                                 |
| `div` sonrası **Floating point exception**                                    | Sıfıra bölme, ya da `rdx`'i sıfırlamadan / `cqo` yapmadan bölme (bölüm 64 bite sığmadı)                                      |
| Değişkenin adresi yerine değeri / tersi                                       | `mov rax, x` (adres) ile `mov rax, [x]` (değer) karışıklığı                                                                  |
| Döngü sayacı bozuluyor                                                        | `syscall` komutu `rcx` ve `r11`'i bozar. Sayaçları `r12`–`r15`'te tut                                                        |
| Yazdırılan çıktı uzunluğu yanlış                                              | Elle sayılmış uzunluk. `equ $ - etiket` kullan                                                                               |
| Karakter yazdırırken `5` yerine garip karakter                                | Rakam yerine sayı değeri yazıyorsun. `add al, '0'` ile ASCII'ye çevir                                                        |
| `printf` çağrısında segfault                                                  | Stack hizalaması (`call`'dan önce `rsp % 16 == 0` olmalı), ya da `xor eax, eax` unutulmuş                                    |
| `relocation R_X86_64_32 against ... can not be used when making a PIE object` | gcc ile bağlarken `mov rdi, fmt` gibi mutlak adres kullanmışsın. `default rel` ve `lea rdi, [fmt]` kullan                    |
| `ld: warning: missing .note.GNU-stack section implies executable stack`       | Zararsız uyarı. Dosyanın sonuna `section .note.GNU-stack noalloc noexec nowrite progbits` satırını ekleyerek susturabilirsin |
| `ld: warning: cannot find entry symbol _start`                                | `global _start` yazmamışsın veya `gcc` ile bağlayıp `main` kullanman gerekir                                                 |
| Yazılan ve beklenen çıktı farklı, nedeni anlaşılmıyor                         | **GDB** ile komut komut izle ve `strace` ile syscall'lara bak                                                                |

---

[← Ek B: Komut kartı](ek-b-komut-karti.md) · [Ek D: Kaynaklar →](ek-d-kaynaklar.md)
