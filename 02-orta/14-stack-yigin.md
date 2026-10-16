[Ana sayfa / İçindekiler](../README.md)

# Bölüm 14: Stack (Yığın)

### Stack nedir?

Stack, programın **geçici veri** (yerel değişkenler, dönüş adresleri, kaydedilen register'lar) sakladığı bellek bölgesidir. **LIFO** (son giren ilk çıkar) çalışır. Tabaklar üst üste dizilir, en üsttekini alırsın.

x86'da stack **aşağı doğru büyür**: yeni veri koyunca `rsp` küçülür.

### rsp: stack pointer

`rsp` her zaman stack'in **tepesini** (en son konulan veriyi) gösterir.

```
Yüksek adres
   ┌────────────┐
   │   ...      │
   ├────────────┤
   │   değer A  │  ← ilk push
   ├────────────┤
   │   değer B  │  ← ikinci push
   ├────────────┤
   │   değer C  │  ← rsp burada (en son push)
   └────────────┘
Düşük adres        stack bu yöne büyür ↓
```

### push ve pop

```nasm
push rax          ; rsp -= 8; [rsp] = rax
pop rbx           ; rbx = [rsp]; rsp += 8
```

x64'te push/pop **8 baytlık (64 bit)** birimlerle çalışır.

Örnek:

```nasm
mov rax, 10
mov rbx, 20
push rax          ; stack: [10]
push rbx          ; stack: [10, 20]  (tepe: 20)
pop rcx           ; rcx = 20
pop rdx           ; rdx = 10
```

Ters sırada pop yaptığına dikkat et. **LIFO.**

**push/pop ile iki register'ı takas etme:**

```nasm
push rax
push rbx
pop rax           ; rax artık eski rbx
pop rbx           ; rbx artık eski rax
```

### Stack'i elle yönetmek

`push` ve `pop` aslında `rsp` üzerinde işlemdir:

```nasm
; push rax  ile eşdeğeri:
sub rsp, 8
mov [rsp], rax

; pop rbx  ile eşdeğeri:
mov rbx, [rsp]
add rsp, 8
```

Bu sayede stack'te yerel değişken alanı açabilirsin:

```nasm
sub rsp, 32           ; 32 bayt yer aç
mov qword [rsp], 5    ; ilk yerel değişken
mov qword [rsp+8], 7  ; ikinci yerel değişken
; ...
add rsp, 32           ; yeri geri ver (stack'i dengele!)
```

**Stack'i dengelemeyi unutma.** Girişte kaç bayt ayırdıysan çıkışta o kadarını geri vermelisin. Aksi halde fonksiyon yanlış dönüş adresine döner ve program çöker.

### Programın başlangıcında stack

Program `_start`'ta başladığında stack şu bilgileri içerir:

```
[rsp]       = argc (argüman sayısı)
[rsp + 8]   = argv[0] (program adı, string adresi)
[rsp + 16]  = argv[1]
...
```

Bunu Bölüm 18'de kullanacağız.

### GDB ile stack'i izle

```
(gdb) x/8xg $rsp       # stack'in tepesinden 8 quadword
(gdb) p $rsp
(gdb) stepi            # push yap, rsp'nin 8 azaldığını gör
```

### Stack taşması (overflow)

Stack sonsuz değildir (Linux'ta varsayılan 8 MB). Sonsuz özyineleme veya devasa yerel diziler stack'i taşırır ve segfault verir. **Stack üzerindeki bir tamponun sınırını aşarak yazmak** ise "stack buffer overflow" güvenlik açığıdır. Dönüş adresinin üzerine yazılabildiği için klasik saldırıların temelidir. Bölüm 15'ten sonra nedenini tam anlayacaksın.

### Alıştırmalar

1. Üç register'ı (`rax`, `rbx`, `rcx`) push edip ters sırada pop et.
2. `push rax` komutunu `sub` ve `mov` ile elle yaz. GDB'de her ikisinin `rsp`'ye etkisini karşılaştır.
3. Bir string'i stack kullanarak ters çevir (her karakteri push et, sonra pop ederek yaz).

---

[← Bölüm 13: Diziler ve stringler](13-diziler-ve-stringler.md) · [Bölüm 15: Fonksiyonlar ve çağrı sözleşmesi →](15-fonksiyonlar-ve-cagri-sozlesmesi.md)
