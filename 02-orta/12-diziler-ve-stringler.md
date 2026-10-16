[Ana sayfa / İçindekiler](../README.md)

# Bölüm 12: Diziler ve stringler

### Diziler

Bir dizi, bellekte ardışık duran aynı boyutlu elemanlardır. `i`. elemana adres: `taban + i * eleman_boyutu`.

```nasm
section .data
    dizi dq 3, 8, 1, 9, 4
    n    equ ($ - dizi) / 8    ; eleman sayısı = toplam bayt / 8
```

**Elemanların toplamı:**

```nasm
section .text
    global _start

_start:
    xor rax, rax                ; toplam
    xor rcx, rcx                ; indeks
.dongu:
    add rax, [dizi + rcx*8]     ; dizi[rcx]
    inc rcx
    cmp rcx, n
    jl .dongu

    mov rdi, rax                ; 25
    mov rax, 60
    syscall
```

**En büyük eleman:**

```nasm
    mov rax, [dizi]             ; en büyük = dizi[0]
    mov rcx, 1
.dongu:
    cmp rcx, n
    jge .bitti
    mov rbx, [dizi + rcx*8]
    cmp rbx, rax
    cmovg rax, rbx              ; rbx > rax ise rax = rbx
    inc rcx
    jmp .dongu
.bitti:
```

**Dizi elemanını değiştirme:**

```nasm
mov qword [dizi + 2*8], 100     ; dizi[2] = 100
```

### Stringler

Assembly'de string, **bayt dizisidir**. C tarzı string'ler sonu `0` baytı ile biter (null-terminated).

```nasm
metin db "Merhaba", 0
```

#### strlen: uzunluk bulma

```nasm
; rdi = string adresi → rax = uzunluk
    xor rax, rax
.dongu:
    cmp byte [rdi + rax], 0
    je .bitti
    inc rax
    jmp .dongu
.bitti:
```

#### atoi: yazıyı sayıya çevirme

`"1234"` → `1234`:

```nasm
; rdi = string adresi → rax = sayı
; ilk rakam olmayan karakterde durur
atoi:
    xor rax, rax
.dongu:
    movzx rcx, byte [rdi]
    sub rcx, '0'
    cmp rcx, 9
    ja .bitti                ; '0'..'9' dışındaysa (işaretsiz karşılaştırma hilesi)
    imul rax, rax, 10
    add rax, rcx
    inc rdi
    jmp .dongu
.bitti:
    ret
```

Hilesi: `'0'`'dan küçük bir karakter çıkarılınca sonuç, işaretsiz olarak çok büyük sayı olur. Böylece tek `ja` ile hem "küçük" hem "büyük" durumunu yakalarız. Satır sonu karakteri (10) da bu şekilde döngüyü bitirir.

### String komutları

x86'da bellek bloğu işleyen özel komutlar vardır. Bunlar `rsi` (kaynak) ve `rdi` (hedef) kullanır ve her seferinde bunları otomatik ilerletir (`DF=0` ise artırır).

| Komut                                 | İşlem                                       |
| ------------------------------------- | ------------------------------------------- |
| `movsb` / `movsw` / `movsd` / `movsq` | `[rdi] ← [rsi]`, ikisini de ilerlet         |
| `stosb` / `stosw` / `stosd` / `stosq` | `[rdi] ← al/ax/eax/rax`, `rdi` ilerlet      |
| `lodsb` / ...                         | `al/ax/eax/rax ← [rsi]`, `rsi` ilerlet      |
| `scasb` / ...                         | `al` ile `[rdi]` karşılaştır, `rdi` ilerlet |
| `cmpsb` / ...                         | `[rsi]` ile `[rdi]` karşılaştır             |

**`rep` öneki** komutu `rcx` kez tekrarlar:

```nasm
cld                       ; yönü ileri yap (DF=0) - alışkanlık olarak yaz

; kaynaktan hedefe uzunluk bayt kopyala
lea rsi, [kaynak]
lea rdi, [hedef]
mov rcx, uzunluk
rep movsb

; tamponu 'A' ile doldur
lea rdi, [tampon]
mov al, 'A'
mov rcx, 10
rep stosb
```

`repe cmpsb` ve `repne scasb` ile string karşılaştırma / karakter arama yapılır.

### Örnek: string kopyalama ve yazdırma

```nasm
section .data
    kaynak db "Assembly guzeldir", 10
    uzun   equ $ - kaynak

section .bss
    hedef resb 64

section .text
    global _start

_start:
    cld
    lea rsi, [kaynak]
    lea rdi, [hedef]
    mov rcx, uzun
    rep movsb

    mov rax, 1
    mov rdi, 1
    lea rsi, [hedef]
    mov rdx, uzun
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall
```

### Alıştırmalar

1. Dizideki en küçük elemanı bul.
2. Bir dizinin elemanlarını ters sırada yaz (yerinde ters çevir).
3. Bir string içinde belirli bir karakterin kaç kez geçtiğini say.
4. `atoi` ile klavyeden bir sayı oku (`read` ile) ve çıkış kodu olarak ver.
5. Bir string'i büyük harfe çeviren program yaz.

---

[← Bölüm 11: Bit operasyonları](11-bit-operasyonlari.md) · [Bölüm 13: Stack (Yığın) →](13-stack-yigin.md)
