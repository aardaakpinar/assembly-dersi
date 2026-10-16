[Ana sayfa / İçindekiler](../README.md)

# Bölüm 8: Sistem çağrıları (syscall)

Bu noktaya kadar programlarımız yalnızca register'larda ve bellekte hesap yaptı, sonucu da çıkış kodu olarak aldık. Gerçek programlar ekrana yazar, klavyeden okur, dosya açar. Bunların hiçbirini işlemci tek başına yapamaz. **İşletim sistemi çekirdeğinin (kernel)** yardımı gerekir.

### Kernel ile konuşmak

Programlar çekirdeğe **sistem çağrısı (system call, syscall)** ile istek gönderir. Assembly'de libc gibi bir kütüphane olmadan bunu doğrudan biz yaparız.

Aslında sen bunu zaten yaptın. Bölüm 2'den beri her programın sonunda şu satırlar vardı:

```nasm
mov rax, 60     ; hangi sistem çağrısı: 60 = exit
mov rdi, 42     ; 1. argüman: çıkış kodu
syscall         ; çekirdeği çağır
```

Artık bunlar "kalıp" değil. Bu, **`exit(42)` sistem çağrısıdır.**

### Linux x86-64 syscall kuralları

```
rax  = syscall numarası      (hangi iş?)
rdi  = 1. argüman
rsi  = 2. argüman
rdx  = 3. argüman
r10  = 4. argüman            (dikkat: rcx değil!)
r8   = 5. argüman
r9   = 6. argüman

syscall komutu çalıştırılır

rax  = dönüş değeri          (hata varsa -errno)
```

⚠️ `syscall` komutu **`rcx` ve `r11` register'larını bozar** (çekirdek bunları kendi işi için kullanır). Bu register'lardaki değerlere sonra ihtiyacın varsa onları başka yerde sakla. Genelde `r12`–`r15` güvenli seçimdir.

En önemli syscall'lar (tam liste için [Ek A](../ek/ek-a-syscall-tablosu.md)):

| Numara | Ad       | rdi        | rsi           | rdx         |
| ------ | -------- | ---------- | ------------- | ----------- |
| 0      | `read`   | fd         | tampon adresi | bayt sayısı |
| 1      | `write`  | fd         | veri adresi   | bayt sayısı |
| 39     | `getpid` |            |               |             |
| 60     | `exit`   | çıkış kodu |               |             |

### Dosya tanımlayıcıları (file descriptor, fd)

Linux'ta her açık "kaynak" (dosya, klavye, ekran...) bir sayıyla temsil edilir. Program başlarken üçü hazırdır:

| fd  | Ad     | Ne için                   |
| --- | ------ | ------------------------- |
| 0   | stdin  | klavye (standart girdi)   |
| 1   | stdout | ekran (standart çıktı)    |
| 2   | stderr | hata mesajları için ekran |

### write ve read nasıl çağrılır?

Henüz çalıştırmayacağız ama imzalarına bakalım. C'deki `write(1, mesaj, uzunluk)` assembly'de şöyle olur:

```nasm
mov rax, 1          ; write
mov rdi, 1          ; fd: 1 = stdout (ekran)
mov rsi, mesaj      ; ne yazılacak: verinin ADRESİ
mov rdx, uzunluk    ; kaç bayt yazılacak
syscall
```

Klavyeden okumak (`read`):

```nasm
mov rax, 0          ; read
mov rdi, 0          ; fd: 0 = stdin (klavye)
mov rsi, tampon     ; okunanlar nereye yazılsın: tamponun ADRESİ
mov rdx, 64         ; en fazla kaç bayt
syscall             ; rax = gerçekte okunan bayt sayısı
```

Dikkat: `write` ve `read` **bellekteki bir adresi** bekler, yani yazılacak veriyi önce `.data`'da tanımlamamız gerekir. Bölüm 5'te gördüğün `mesaj` ile `[mesaj]` farkını hatırla: burada **adres** (köşeli parantezsiz) veriyoruz. Bir sonraki bölümde bunları birlikte çalıştıracağız.

### Çıkış kodu ile deneyelim: getpid

`write` kullanmadan başka bir syscall'ı deneyebiliriz. `getpid` (numara 39) çalışan programın süreç kimliğini (PID) döndürür:

```nasm
section .text
    global _start

_start:
    mov rax, 39         ; getpid
    syscall             ; rax = bu sürecin PID'i

    mov rdi, rax        ; PID'i çıkış kodu yap
    mov rax, 60         ; exit
    syscall
```

```bash
./pid; echo $?
./pid; echo $?
```

Her çalıştırdığında farklı bir sayı görürsün (PID'in alt 8 biti). İşletim sistemi sana gerçek bir değer verdi ve sen bunu `rax` içinden okudun.

### Hata kontrolü

Başarısız syscall'lar `rax`'te **negatif** bir değer döndürür (`-errno`). Örneğin `-2` = `ENOENT` (dosya yok), `-13` = `EACCES` (izin yok). Bunu Bölüm 10'da `test rax, rax` + `js hata` ile kontrol etmeyi öğreneceksin.

### Eski yöntem: `int 0x80`

32 bit Linux'tan kalma bir syscall yöntemi daha var (`int 0x80`). 64 bit programlarda **kullanma**; farklı numaralar ve 32 bit register'lar kullanır ve tuzaklarla doludur.

### Alıştırmalar

1. `getpid` programını birkaç kez çalıştır ve çıkış kodunun her seferinde değiştiğini doğrula.
2. `getppid` (numara 110) syscall'ı ile üst sürecin (kabuğun) PID'ini çıkış kodu olarak döndür. Arka arkaya çalıştırdığında bu sayı sabit mi kalıyor? Neden?
3. Terminalde `man 2 write` komutunu çalıştır. Parametreleri yukarıdaki tabloyla karşılaştır (`fd`, `buf`, `count`).
4. `rcx` ve `r11`'in `syscall` sonrası değiştiğini GDB ile gör: `mov rcx, 1234` yaz, `syscall`'dan önce ve sonra `info registers rcx` komutuyla bak. (Bölüm 7)

---

[← Bölüm 7: GDB ile hata ayıklama](07-gdb-ile-hata-ayiklama.md) · [Bölüm 9: Merhaba Dünya →](09-merhaba-dunya.md)
