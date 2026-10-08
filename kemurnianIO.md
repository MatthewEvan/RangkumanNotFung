# Kemurnian (Purity) dan I/O di Haskell

Catatan persiapan materi kemurnian dan I/O IF2110. Semua program di bawah sudah dijalankan dengan GHC 9.10.3 memakai input yang tertulis, dan output yang ditulis adalah output aslinya.

> Catatan sebelumnya: [`penjelasanLambda.md`](penjelasanLambda.md) (lambda dan HOF) dan [`binarytree.md`](binarytree.md) (pohon biner).

---

## Daftar Isi

1. [Fungsi murni](#1-fungsi-murni)
2. [Transparansi referensial](#2-transparansi-referensial)
3. [Kenapa Haskell memaksa kemurnian?](#3-kenapa-haskell-memaksa-kemurnian)
4. [Masalahnya: program butuh I/O](#4-masalahnya-program-butuh-io)
5. [Tipe `IO`](#5-tipe-io)
6. [`main` dan cara menjalankan program](#6-main-dan-cara-menjalankan-program)
7. [Notasi `do`: `<-`, `let`, dan aksi](#7-notasi-do---let-dan-aksi)
8. [Output: `putStr`, `putStrLn`, `print`, `show`](#8-output-putstr-putstrln-print-show)
9. [Input: `getLine`, `read`, `readLn`, `words`, `lines`](#9-input-getline-read-readln-words-lines)
10. [Pola terpenting: inti murni, kulit IO](#10-pola-terpenting-inti-murni-kulit-io)
11. [`return` bukan "keluar dari fungsi"](#11-return-bukan-keluar-dari-fungsi)
12. [Percabangan dan pengulangan di IO](#12-percabangan-dan-pengulangan-di-io)
13. [Menyimpan "keadaan" tanpa variabel yang berubah](#13-menyimpan-keadaan-tanpa-variabel-yang-berubah)
14. [Kesalahan umum](#14-kesalahan-umum)
15. [Cheat sheet](#15-cheat-sheet)
16. [Latihan soal](#16-latihan-soal)

---

## 1. Fungsi murni

**Fungsi murni** (*pure function*) adalah fungsi yang memenuhi dua syarat:

1. **Input yang sama selalu menghasilkan output yang sama.** Hasilnya hanya bergantung pada parameter, tidak pada jam, isi file, ketikan user, atau variabel global.
2. **Tidak punya efek samping** (*side effect*). Fungsi tidak mencetak ke layar, tidak membaca keyboard, tidak menulis file, dan tidak mengubah variabel apa pun. Satu-satunya hasil kerjanya adalah **nilai yang dikembalikan**.

Semua fungsi yang kamu tulis di praktikum sejauh ini murni: `sigma`, `gnomeTerlihat`, `buatEnkripsi`, `nbElmt`. `nbElmt t1` dipanggil kapan pun dan berapa kali pun, hasilnya selalu `6`.

### Contoh fungsi yang TIDAK murni (di bahasa lain)

Python, karena bergantung pada variabel luar dan mengubahnya:

```python
counter = 0
def tambah(x):
    global counter
    counter += 1          # mengubah variabel luar → efek samping
    return x + counter    # hasil bergantung pada berapa kali sudah dipanggil

tambah(5)   # 6
tambah(5)   # 7  ← input sama, output beda
```

Python, karena membaca dan menulis ke dunia luar:

```python
def sapa():
    nama = input()        # hasil bergantung pada ketikan user
    print("Halo", nama)   # mencetak ke layar → efek samping
```

Contoh lain yang tidak murni: fungsi yang mengembalikan waktu sekarang, angka acak, atau isi sebuah file.

### Cek cepat: murni atau tidak?

| Fungsi | Murni? | Alasan |
|---|---|---|
| `kuadrat x = x * x` | ✔ | hanya bergantung pada `x` |
| `max a b` | ✔ | hanya bergantung pada `a` dan `b` |
| "kembalikan jam sekarang" | ✘ | hasil berubah tiap dipanggil |
| "cetak x lalu kembalikan x" | ✘ | mencetak adalah efek samping |
| "tambahkan x ke variabel global total" | ✘ | mengubah keadaan di luar fungsi |
| "angka acak antara 1 dan 6" | ✘ | input sama (tidak ada), output beda-beda |

---

## 2. Transparansi referensial

Konsekuensi terpenting dari kemurnian adalah **transparansi referensial** (*referential transparency*):

> Pemanggilan fungsi murni selalu boleh **diganti dengan hasilnya** tanpa mengubah arti program.

Kalau `nbElmt t1 = 6`, maka di mana pun `nbElmt t1` muncul, kamu boleh menulis `6` saja:

```haskell
nbElmt t1 + nbElmt t1
= 6 + nbElmt t1
= 6 + 6
= 12
```

Inilah yang membuat **dry run** seperti di `gnomeTerlihat` dan `buatEnkripsi` bisa dikerjakan: kita tinggal mengganti ekspresi dengan hasilnya, langkah demi langkah, seperti menyederhanakan rumus matematika.

Di bahasa yang tidak murni, ini tidak berlaku. Dengan `tambah` di bagian 1, `tambah(5) + tambah(5)` **tidak** sama dengan `2 * tambah(5)`, karena setiap pemanggilan mengubah `counter`.

---

## 3. Kenapa Haskell memaksa kemurnian?

Di Haskell, **semua fungsi biasa wajib murni**. Ini bukan anjuran, melainkan aturan bahasa:

- **Tidak ada variabel yang bisa diubah.** `x = 5` artinya `x` *adalah* 5 selamanya, bukan "isi kotak `x` dengan 5". Karena itu record update di `Portfolio.hs` dan `insertBST` di `binarytree.md` selalu **membuat nilai baru**.
- **Tidak bisa mencetak atau membaca input** di dalam fungsi biasa. Kamu tidak bisa menyelipkan `print` di dalam `nbElmt`.

Manfaatnya:

| Manfaat | Penjelasan |
|---|---|
| Mudah dites | cukup cek pasangan input dan output, persis seperti bagian APLIKASI di file praktikummu |
| Mudah dinalar | untuk memahami fungsi, cukup baca fungsi itu sendiri. Tidak ada keadaan tersembunyi |
| Tidak ada bug "urutan pemanggilan" | hasil tidak bergantung pada fungsi lain yang dipanggil sebelumnya |
| Evaluasi *lazy* aman | Haskell boleh menghitung sesuatu kapan saja (atau tidak sama sekali kalau tidak dibutuhkan), karena hasilnya pasti sama |

---

## 4. Masalahnya: program butuh I/O

Program yang tidak bisa membaca input dan menampilkan output tidak berguna. Padahal I/O pada dasarnya **tidak murni**:

- membaca input memberi hasil berbeda tergantung apa yang diketik user,
- mencetak ke layar adalah efek samping.

Jadi bagaimana bahasa yang "semua fungsinya murni" bisa melakukan I/O?

Jawaban Haskell: **I/O tetap boleh, tapi ditandai dan dipisahkan lewat tipe.**

---

## 5. Tipe `IO`

`IO a` artinya:

> **Sebuah aksi** yang, kalau dijalankan, boleh berinteraksi dengan dunia luar, lalu menghasilkan nilai bertipe `a`.

| Aksi | Tipe | Arti |
|---|---|---|
| `getLine` | `IO String` | aksi membaca satu baris, hasilnya `String` |
| `readLn` | `IO a` | aksi membaca satu baris lalu mengubahnya ke tipe `a` (misalnya `Int`) |
| `putStrLn "halo"` | `IO ()` | aksi mencetak, tidak menghasilkan nilai yang berguna |
| `print 42` | `IO ()` | aksi mencetak nilai apa pun yang punya `Show` |

`()` dibaca **unit**, yaitu tipe yang hanya punya satu nilai, juga ditulis `()`. Aksi bertipe `IO ()` dijalankan demi **efeknya**, bukan demi hasilnya. Mirip `void` di C.

### `getLine` bukan `String`

Ini konsep paling penting di materi ini:

> `getLine` **bukan** sebuah string. `getLine` adalah **resep** untuk mendapatkan sebuah string dari keyboard.

Karena itu, kamu tidak bisa memperlakukannya seperti string biasa:

```haskell
length getLine         -- ERROR: getLine bertipe IO String, bukan String
getLine ++ "!"         -- ERROR: sama
```

Analogi: `getLine` itu seperti **resep kue**, bukan kuenya. Kamu tidak bisa memakan resep. Kamu harus **menjalankan** resepnya dulu (di dalam blok `do`, bagian 7), baru mendapat kuenya.

### Tipe sebagai penanda

Dengan aturan ini, tipe memberi tahu kamu apakah sebuah fungsi murni:

```haskell
nbElmt   :: BinTree a -> Int     -- tidak ada IO → DIJAMIN murni
bacaInt  :: IO Int               -- ada IO → aksi yang berinteraksi dengan dunia luar
cetakPohon :: BinTree Int -> IO ()   -- menerima pohon, menghasilkan aksi mencetak
```

Fungsi murni **tidak bisa** memanggil aksi IO dan mengambil hasilnya. Yang sebaliknya boleh: kode IO bebas memanggil fungsi murni. Jadi kemurnian di bagian program yang tidak ber-`IO` tetap terjaga.

---

## 6. `main` dan cara menjalankan program

Program Haskell dimulai dari `main`, yang bertipe `IO ()`:

```haskell
main :: IO ()
main = putStrLn "Halo, dunia!"
```

`main` adalah **satu aksi besar**. Saat program dijalankan, aksi inilah yang dieksekusi.

### Tiga cara menjalankan

Misalkan programnya ada di file `Sapa.hs`.

**1. `runghc`**: langsung jalankan tanpa membuat file `.exe`. Paling praktis untuk latihan.

```bash
runghc Sapa.hs
```

**2. `ghci`**: muat file, lalu panggil `main`.

```bash
ghci Sapa.hs
```

Lalu di dalam ghci ketik `main`. Di ghci kamu juga bisa memanggil fungsi murni satu per satu untuk dites, jadi ini cara terbaik untuk debugging.

**3. `ghc`**: kompilasi jadi file `.exe`, lalu jalankan.

```bash
ghc Sapa.hs
```

```bash
.\Sapa.exe
```

> ⚠️ Untuk dikompilasi menjadi `.exe`, file harus **tanpa** baris `module ... where`, atau ditulis `module Main where`. File seperti `module KalkulatorDeret where` hanya bisa dimuat di ghci, tidak bisa jadi program mandiri.

### Mengetes dengan input dari file

Supaya tidak perlu mengetik input berulang-ulang, simpan input di file `input.txt`, lalu arahkan ke program. Di PowerShell:

```powershell
Get-Content input.txt | runghc Sapa.hs
```

---

## 7. Notasi `do`: `<-`, `let`, dan aksi

Beberapa aksi dirangkai **berurutan** dengan notasi `do`:

```haskell
main :: IO ()
main = do
    putStrLn "Siapa namamu?"
    nama <- getLine
    putStrLn ("Halo, " ++ nama ++ "!")
```

Input:
```
Budi
```
Output:
```
Siapa namamu?
Halo, Budi!
```

Di dalam `do`, setiap baris adalah salah satu dari tiga bentuk ini:

| Sintaks | Arti | Contoh | Tipe |
|---|---|---|---|
| `x <- aksi` | **jalankan** aksi IO, simpan **hasilnya** ke `x` | `nama <- getLine` | `getLine :: IO String`, `nama :: String` |
| `let x = ekspresi` | beri nama pada nilai **murni**, tanpa menjalankan aksi | `let n = length nama` | `n :: Int` |
| `aksi` | jalankan aksi, abaikan hasilnya | `putStrLn "halo"` | |

Cara membedakan `<-` dan `let`:
- Sisi kanan bertipe `IO ...` dan kamu ingin hasilnya → **`<-`**
- Sisi kanan nilai biasa (hasil fungsi murni) → **`let`**

`<-` adalah satu-satunya cara "membuka" `IO String` menjadi `String`, dan itu **hanya bisa dilakukan di dalam blok `do`**. Itulah yang menjaga kemurnian: fungsi biasa tidak punya blok `do`, jadi tidak bisa membuka hasil IO.

> ⚠️ **Indentasi penting.** Semua baris dalam satu blok `do` harus dimulai di kolom yang sama.

---

## 8. Output: `putStr`, `putStrLn`, `print`, `show`

| Fungsi | Tipe | Arti |
|---|---|---|
| `putStr s` | `String -> IO ()` | cetak `s` **tanpa** pindah baris |
| `putStrLn s` | `String -> IO ()` | cetak `s` lalu pindah baris |
| `print x` | `Show a => a -> IO ()` | cetak nilai **apa pun** yang punya `Show`, lalu pindah baris |
| `show x` | `Show a => a -> String` | **fungsi murni**: ubah nilai menjadi `String` |

`print x` sama dengan `putStrLn (show x)`.

```haskell
main :: IO ()
main = do
    putStr "satu "
    putStr "dua"
    putStrLn ""
    putStrLn "tiga"
    print 42
    print "halo"
    putStrLn "halo"
    print [1,2,3]
    print (3, True)
    putStrLn (show 42 ++ " ekor")
```

Output:
```
satu dua
tiga
42
"halo"
halo
[1,2,3]
(3,True)
42 ekor
```

> ⚠️ Perhatikan `print "halo"` mencetak **dengan tanda kutip**, karena `show "halo"` menghasilkan `"\"halo\""`. Untuk mencetak teks apa adanya, pakai `putStrLn`.

### Merapikan output

Soal sering meminta output dipisah spasi atau per baris, bukan format list `[17,5,2]`:

| Fungsi | Arti | Contoh → hasil |
|---|---|---|
| `unwords` | gabung list string dengan spasi | `unwords ["17","5","2"]` → `"17 5 2"` |
| `unlines` | gabung list string, setiap elemen diakhiri baris baru | `unlines ["a","b"]` → `"a\nb\n"` |
| `mapM_ print xs` | cetak setiap elemen di baris sendiri | |

```haskell
main :: IO ()
main = do
    mapM_ print [3,2,1]
    putStrLn (unwords (map show [17,5,2]))
    putStr (unlines ["baris 1", "baris 2"])
```

Output:
```
3
2
1
17 5 2
baris 1
baris 2
```

`unwords (map show xs)` adalah pola yang sangat sering dipakai: `map show` mengubah setiap angka jadi string, lalu `unwords` menggabungkannya dengan spasi. Lagi-lagi HOF dari praktikum sebelumnya.

---

## 9. Input: `getLine`, `read`, `readLn`, `words`, `lines`

### `read`: dari `String` ke nilai

`getLine` selalu menghasilkan `String`. Untuk angka, pakai `read` (fungsi murni, kebalikan dari `show`):

```haskell
main :: IO ()
main = do
    s <- getLine
    let n = read s :: Int
    print (n * 2)
```

Input `12`, output:
```
24
```

`:: Int` memberi tahu Haskell tipe hasil `read`. Kalau tipenya sudah jelas dari pemakaian (misalnya hasilnya dikirim ke fungsi bertipe `Int -> ...`), anotasi ini boleh dihilangkan.

### `readLn`: `getLine` + `read` sekaligus

```haskell
n <- readLn :: IO Int        -- sama dengan: s <- getLine; let n = read s :: Int
```

### Beberapa angka dalam satu baris: `words`

`words` memecah string berdasarkan spasi:

```haskell
-- words "3 5 7" = ["3","5","7"]

main :: IO ()
main = do
    baris <- getLine
    let angka = map read (words baris) :: [Int]
    print (sum angka)
```

Input `3 5 7`, output:
```
15
```

Bacanya: `words` memecah jadi `["3","5","7"]`, `map read` mengubah tiap potongan jadi angka `[3,5,7]`, lalu `sum` menjumlahkan.

### Seluruh input sekaligus: `getContents` dan `lines`

`getContents` membaca **semua** input sampai habis sebagai satu `String`. `lines` memecahnya per baris:

```haskell
main :: IO ()
main = do
    isi <- getContents
    let angka = map read (lines isi) :: [Int]
    putStrLn ("Banyak data: " ++ show (length angka))
    putStrLn ("Maksimum: " ++ show (maximum angka))
```

Input:
```
4
9
1
```
Output:
```
Banyak data: 3
Maksimum: 9
```

Ini berguna kalau banyaknya baris input tidak diketahui.

### Ringkasan pola membaca input

| Bentuk input | Pola |
|---|---|
| satu teks | `s <- getLine` |
| satu angka | `n <- readLn :: IO Int` |
| beberapa angka dalam satu baris | `xs <- fmap (map read . words) getLine`, atau `getLine` lalu `let xs = map read (words s)` |
| N baris, N diketahui | `replicateM n getLine` (bagian 12) |
| sampai input habis | `isi <- getContents` lalu `lines isi` |
| sampai bertemu penanda (misal `0`) | rekursi IO (bagian 12) |

---

## 10. Pola terpenting: inti murni, kulit IO

Cara menulis program Haskell yang baik:

> **Kulit IO yang tipis** di sekeliling **inti murni yang tebal.**

Semua logika ditulis sebagai fungsi murni. Bagian IO hanya **membaca**, **memanggil fungsi murni**, lalu **mencetak**.

```haskell
module Main where

-- INTI MURNI: seluruh logika di sini, bisa dites di ghci tanpa input
gnomeTerlihat :: [Integer] -> [Integer]
gnomeTerlihat l = foldr gabung [] l
    where
        gabung x [] = [x]
        gabung x (y:ys)
            | x > y     = x : y : ys
            | otherwise = y : ys

-- KULIT IO: baca → olah → cetak
main :: IO ()
main = do
    baris <- getLine                              -- 1. baca
    let l = map read (words baris) :: [Integer]   -- 2. ubah teks jadi data
    print (gnomeTerlihat l)                       -- 3. olah (murni), lalu cetak
```

Input `16 17 4 3 5 2`, output:
```
[17,5,2]
```

Kenapa pola ini bagus:

- `gnomeTerlihat` bisa dites di ghci tanpa mengetik input: `gnomeTerlihat [16,17,4,3,5,2]`.
- Kalau output salah, kamu langsung tahu masalahnya ada di **parsing input** (bagian IO) atau di **logika** (bagian murni).
- Fungsi murninya bisa dipakai ulang di program lain.

Driver Olympia yang disebut di template `KalkulatorDeret.hs` bekerja dengan cara ini juga: **driver** memegang bagian IO (membaca test case, mencetak hasil), dan **kamu** menulis inti murninya.

---

## 11. `return` bukan "keluar dari fungsi"

Di bahasa lain, `return` menghentikan fungsi. **Di Haskell tidak.**

`return x` artinya: **bungkus nilai murni `x` menjadi aksi IO** yang tidak melakukan apa-apa selain menghasilkan `x`.

```haskell
return :: a -> IO a       -- (versi khusus IO)
```

```haskell
main :: IO ()
main = do
    return 5
    putStrLn "masih jalan"
```

Output:
```
masih jalan
```

`return 5` tidak menghentikan apa pun. Ia hanya aksi kosong yang hasilnya dibuang.

Kegunaan `return` yang sebenarnya: menjadi baris terakhir sebuah fungsi IO yang perlu **menghasilkan nilai**:

```haskell
bacaInt :: IO Int
bacaInt = do
    s <- getLine
    return (read s)      -- read s :: Int adalah nilai murni, dibungkus jadi IO Int
```

Hubungan `<-` dan `return`:
- `<-` **membuka** `IO a` menjadi `a`
- `return` **membungkus** `a` menjadi `IO a`

---

## 12. Percabangan dan pengulangan di IO

### `if-then-else` di dalam `do`

Kedua cabang harus berupa aksi IO dengan tipe yang sama:

```haskell
import Control.Monad (when)

main :: IO ()
main = do
    n <- readLn :: IO Int
    if even n
        then putStrLn "genap"
        else putStrLn "ganjil"
    when (n > 100) (putStrLn "besar sekali!")
```

| Input | Output |
|---|---|
| `7` | `ganjil` |
| `200` | `genap` lalu `besar sekali!` |

`when kondisi aksi` adalah `if` tanpa `else`: aksi hanya dijalankan kalau kondisinya `True`. Fungsi ini perlu `import Control.Monad (when)` di baris paling atas file.

Kalau satu cabang butuh **lebih dari satu aksi**, buka `do` baru di cabang itu:

```haskell
if x == 0
    then return []
    else do
        ...
        ...
```

### Pengulangan dengan rekursi

Haskell tidak punya `for` atau `while`. Untuk mengulang aksi IO, pakai **rekursi**, sama seperti fungsi biasa:

```haskell
hitungMundur :: Int -> IO ()
hitungMundur 0 = putStrLn "Selesai!"
hitungMundur n = do
    print n
    hitungMundur (n - 1)

main :: IO ()
main = do
    n <- bacaInt          -- bacaInt dari bagian 11
    hitungMundur n
```

Input `3`, output:
```
3
2
1
Selesai!
```

### Membaca sampai bertemu penanda

Pola umum di soal: "baca angka satu per baris sampai bertemu 0".

```haskell
bacaSampaiNol :: IO [Int]
bacaSampaiNol = do
    x <- readLn
    if x == 0
        then return []                -- penanda: berhenti, hasilnya list kosong
        else do
            sisa <- bacaSampaiNol     -- rekursi: baca sisanya
            return (x : sisa)         -- gabungkan x dengan sisanya

main :: IO ()
main = do
    xs <- bacaSampaiNol
    putStrLn ("Total: " ++ show (sum xs))
```

Input:
```
5
8
2
0
```
Output:
```
Total: 15
```

Strukturnya sama persis dengan rekursi list: basis (`0` → `[]`) dan rekurens (`x : sisa`). Bedanya, setiap langkah perlu `<-` untuk membuka hasil IO dan `return` untuk membungkus hasil akhirnya.

### `mapM_`: jalankan aksi untuk setiap elemen

`mapM_` adalah "`map` untuk aksi IO". Fungsi ini menjalankan aksi untuk setiap elemen list dan membuang hasilnya:

```haskell
mapM_ print [3,2,1]                              -- cetak tiap elemen per baris
mapM_ (\x -> putStrLn ("Halo, " ++ x)) nama      -- dengan lambda
```

> ⚠️ `map print [1,2,3]` **tidak mencetak apa-apa**. Hasilnya hanya list berisi tiga *resep* aksi yang tidak pernah dijalankan. Untuk menjalankannya, pakai `mapM_`.

### `replicateM`: ulangi aksi N kali

```haskell
import Control.Monad (replicateM)

main :: IO ()
main = do
    n <- readLn :: IO Int
    xs <- replicateM n readLn :: IO [Int]    -- baca n baris, kumpulkan jadi list
    print (filter even xs)
```

Input:
```
4
10
7
8
3
```
Output:
```
[10,8]
```

Baris pertama adalah banyak data (`4`), lalu empat baris berikutnya adalah datanya. Format input seperti ini sangat umum di soal.

---

## 13. Menyimpan "keadaan" tanpa variabel yang berubah

Bagaimana membuat program yang "mengingat" sesuatu, misalnya saldo, kalau variabel tidak bisa diubah?

Jawabannya: **bawa keadaan sebagai parameter** dari satu pemanggilan rekursi ke pemanggilan berikutnya. Ini sama dengan akumulator di `foldl`.

```haskell
-- saldo dibawa sebagai PARAMETER, bukan variabel yang diubah
loop :: Int -> IO ()
loop saldo = do
    perintah <- getLine
    case words perintah of
        ["tambah", x] -> loop (saldo + read x)
        ["ambil", x]  -> loop (saldo - read x)
        ["lihat"]     -> do
            putStrLn ("Saldo: " ++ show saldo)
            loop saldo
        ["keluar"]    -> putStrLn "Sampai jumpa!"
        _             -> do
            putStrLn "Perintah tidak dikenal"
            loop saldo

main :: IO ()
main = loop 0
```

Input:
```
tambah 50
ambil 20
lihat
halo
tambah 5
lihat
keluar
```
Output:
```
Saldo: 30
Perintah tidak dikenal
Saldo: 35
Sampai jumpa!
```

Tidak ada `saldo = saldo + x`. Yang terjadi adalah `loop` **dipanggil lagi dengan nilai saldo yang baru**. Setiap pemanggilan punya `saldo`-nya sendiri yang tidak pernah berubah.

`case ... of` adalah pattern matching di dalam ekspresi. Di sini dipakai untuk mencocokkan hasil `words perintah` dengan bentuk-bentuk perintah yang dikenal. `_` menangkap semua bentuk lain.

---

## 14. Kesalahan umum

| Kesalahan | Penyebab | Perbaikan |
|---|---|---|
| `let nama = getLine` lalu `length nama` error | `let` tidak menjalankan aksi, `nama` masih bertipe `IO String` | `nama <- getLine` |
| `x <- length s` error | `length s` nilai murni, bukan aksi IO | `let x = length s` |
| `read s` error *ambiguous type* atau `no parse` | Haskell tidak tahu `s` mau dibaca jadi tipe apa | `read s :: Int` |
| `Prelude.read: no parse` saat dijalankan | teks tidak sesuai tipe, misalnya `read "3 5" :: Int` | pecah dulu dengan `words`, lalu `map read` |
| `putStrLn 42` error | `putStrLn` butuh `String` | `print 42` atau `putStrLn (show 42)` |
| `putStrLn "Nilai: " ++ show n` error | `putStrLn` hanya menerima `"Nilai: "`, sisanya dianggap terpisah | `putStrLn ("Nilai: " ++ show n)` atau `putStrLn $ "Nilai: " ++ show n` |
| output ada tanda kutip `"halo"` | memakai `print` untuk string | pakai `putStrLn` |
| `map print xs` tidak mencetak apa-apa | `map` hanya membuat list aksi, tidak menjalankannya | `mapM_ print xs` |
| mengira `return` menghentikan fungsi | `return` hanya membungkus nilai ke `IO` | pakai `if-then-else` untuk menghentikan alur |
| mencoba `print` di dalam fungsi murni | fungsi murni tidak boleh punya efek samping | tes fungsi murninya langsung di ghci |
| error *parse* di blok `do` | baris dalam `do` tidak rata | semua aksi dalam satu `do` mulai di kolom yang sama |
| `if` tanpa `else` di `do` error | di Haskell `if` **wajib** punya `else` | tambahkan `else return ()`, atau pakai `when` |
| `ghc File.hs` tidak menghasilkan `.exe` | file diawali `module NamaLain where` | hapus baris `module`, atau ganti jadi `module Main where` |
| prompt dari `putStr` tidak muncul sebelum input (program hasil kompilasi) | output ditahan di *buffer* sampai ada baris baru | pakai `putStrLn`, atau tambahkan `hFlush stdout` (dari `import System.IO`) setelah `putStr` |

---

## 15. Cheat sheet

```haskell
-- KEMURNIAN
-- fungsi tanpa IO di tipenya → PASTI murni
-- input sama → output sama, tidak ada efek samping
-- variabel tidak bisa diubah; "keadaan" dibawa lewat parameter

-- PROGRAM
main :: IO ()
main = do
    s  <- getLine                    -- jalankan aksi, ambil hasil
    n  <- readLn :: IO Int           -- baca satu angka
    let xs = map read (words s) :: [Int]   -- nilai murni pakai let
    print (sum xs)                   -- jalankan aksi
    putStrLn ("n = " ++ show n)

-- OUTPUT
putStr s          -- tanpa baris baru
putStrLn s        -- dengan baris baru
print x           -- = putStrLn (show x); string ikut dikutip
unwords (map show xs)    -- "1 2 3"
mapM_ print xs           -- satu elemen per baris

-- INPUT
getLine                  -- IO String, satu baris
readLn                   -- IO a, satu baris jadi angka/nilai
getContents              -- IO String, semua input
words / lines            -- pecah per spasi / per baris
read s :: Int            -- String → Int (murni)

-- PENGULANGAN
import Control.Monad (replicateM, when)
xs <- replicateM n getLine        -- baca n baris
mapM_ aksi xs                     -- jalankan aksi untuk tiap elemen
when kondisi aksi                 -- if tanpa else
-- atau rekursi: f n = do { ...; f (n - 1) }

-- RETURN
return x          -- bungkus nilai murni jadi IO; TIDAK menghentikan fungsi
```

**Cara cepat memutuskan `<-` atau `let`:**

- Sisi kanan bertipe `IO ...` → `<-`
- Sisi kanan nilai biasa → `let`

**Pola program:** baca (IO) → ubah teks jadi data (`words`, `read`) → olah (fungsi murni) → rapikan (`show`, `unwords`) → cetak (IO).

---

## 16. Latihan soal

Untuk soal program, simpan jawaban di file `.hs`, lalu jalankan dengan `runghc NamaFile.hs`. Ketik inputnya, atau arahkan dari file seperti di bagian 6.

### Level 1: konsep

**1. Murni atau tidak?** Untuk setiap fungsi, tentukan apakah murni, dan jelaskan alasannya.

```haskell
a) luasLingkaran :: Double -> Double
   luasLingkaran r = 3.14 * r * r

b) bacaNama :: IO String
   bacaNama = getLine

c) sapa :: String -> String
   sapa nama = "Halo, " ++ nama

d) cetakSapa :: String -> IO ()
   cetakSapa nama = putStrLn ("Halo, " ++ nama)
```

e) Fungsi Python berikut:
```python
total = 0
def catat(x):
    global total
    total = total + x
    return total
```

**2. Benar atau salah?** Kalau salah, perbaiki.

```haskell
main = do
    nama <- getLine
    let n <- length nama
    putStrLn "Panjang nama: " ++ show n
```

<details>
<summary>Jawaban level 1</summary>

**1.**
- a) **Murni.** Hasil hanya bergantung pada `r`.
- b) **Tidak murni** (aksi IO). Tipenya `IO String`, hasilnya bergantung pada ketikan user.
- c) **Murni.** Hanya mengolah string, tanpa mencetak apa pun.
- d) **Menghasilkan aksi IO.** Tipenya `... -> IO ()`, dan aksinya mencetak ke layar (efek samping). Perhatikan bedanya dengan c): fungsi c) hanya **membuat** teks sapaan, sedangkan d) **mencetak**-nya.
- e) **Tidak murni.** Fungsi mengubah variabel global `total`, sehingga `catat(5)` memberi hasil berbeda setiap kali dipanggil.

**2.** Ada dua kesalahan:
- `let n <- length nama` salah: `length nama` nilai murni, jadi pakai `let n = length nama`.
- `putStrLn "Panjang nama: " ++ show n` salah: `putStrLn` hanya menerima `"Panjang nama: "`. Harus dikurung.

```haskell
main = do
    nama <- getLine
    let n = length nama
    putStrLn ("Panjang nama: " ++ show n)
```

</details>

### Level 2: program dasar

**3. Ulang tahun.** Baca nama (baris 1) dan umur (baris 2), lalu cetak sapaan.

| Input | Output |
|---|---|
| `Budi`<br>`19` | `Halo Budi, tahun depan kamu berumur 20 tahun.` |

**4. Statistik satu baris.** Baca satu baris berisi beberapa bilangan bulat, lalu cetak jumlah, maksimum, dan minimum.

| Input | Output |
|---|---|
| `4 9 1 7` | `Jumlah: 21`<br>`Maksimum: 9`<br>`Minimum: 1` |

**5. Segitiga bintang.** Baca `n`, lalu cetak segitiga bintang setinggi `n`. Buat bagian murninya sebagai fungsi `segitiga :: Int -> [String]`.

| Input | Output |
|---|---|
| `4` | `*`<br>`**`<br>`***`<br>`****` |

<details>
<summary>Jawaban level 2</summary>

```haskell
-- Soal 3
main :: IO ()
main = do
    nama <- getLine
    umur <- readLn :: IO Int
    putStrLn ("Halo " ++ nama ++ ", tahun depan kamu berumur " ++ show (umur + 1) ++ " tahun.")
```

```haskell
-- Soal 4
main :: IO ()
main = do
    baris <- getLine
    let xs = map read (words baris) :: [Int]
    putStrLn ("Jumlah: " ++ show (sum xs))
    putStrLn ("Maksimum: " ++ show (maximum xs))
    putStrLn ("Minimum: " ++ show (minimum xs))
```

```haskell
-- Soal 5
segitiga :: Int -> [String]
segitiga n = map (\i -> replicate i '*') [1 .. n]

main :: IO ()
main = do
    n <- readLn
    mapM_ putStrLn (segitiga n)
```

Catatan soal 5: `replicate i '*'` membuat string berisi `i` bintang. Seluruh logika ada di `segitiga` (murni), dan `main` hanya mencetak. Kamu bisa mengetes `segitiga 4` di ghci tanpa input.

</details>

### Level 3: pengulangan

**6. Sapa N orang.** Baris pertama berisi `N`, lalu `N` baris berisi nama. Sapa setiap orang.

| Input | Output |
|---|---|
| `3`<br>`Ani`<br>`Budi`<br>`Caca` | `Halo, Ani!`<br>`Halo, Budi!`<br>`Halo, Caca!` |

**7. Balik urutan.** Baca baris demi baris sampai bertemu baris `STOP` (tidak ikut dihitung). Cetak banyaknya baris, lalu cetak semua baris dalam urutan terbalik.

| Input | Output |
|---|---|
| `satu`<br>`dua`<br>`tiga`<br>`STOP` | `Banyak baris: 3`<br>`tiga`<br>`dua`<br>`satu` |

**8. Gnome rapi.** Baca satu baris tinggi gnome, cetak gnome yang terlihat (pakai `gnomeTerlihat` dari Praktikum 5) **dipisah spasi**. Kalau barisnya kosong, cetak `kosong`.

| Input | Output |
|---|---|
| `16 17 4 3 5 2` | `17 5 2` |
| *(baris kosong)* | `kosong` |

<details>
<summary>Jawaban level 3</summary>

```haskell
-- Soal 6
import Control.Monad (replicateM)

main :: IO ()
main = do
    n <- readLn :: IO Int
    nama <- replicateM n getLine
    mapM_ (\x -> putStrLn ("Halo, " ++ x ++ "!")) nama
```

```haskell
-- Soal 7
bacaSampaiStop :: IO [String]
bacaSampaiStop = do
    s <- getLine
    if s == "STOP"
        then return []
        else do
            sisa <- bacaSampaiStop
            return (s : sisa)

main :: IO ()
main = do
    xs <- bacaSampaiStop
    putStrLn ("Banyak baris: " ++ show (length xs))
    mapM_ putStrLn (reverse xs)
```

```haskell
-- Soal 8
gnomeTerlihat :: [Integer] -> [Integer]
gnomeTerlihat l = foldr gabung [] l
    where
        gabung x [] = [x]
        gabung x (y:ys)
            | x > y     = x : y : ys
            | otherwise = y : ys

formatHasil :: [Integer] -> String
formatHasil [] = "kosong"
formatHasil xs = unwords (map show xs)

main :: IO ()
main = do
    baris <- getLine
    let l = map read (words baris)
    putStrLn (formatHasil (gnomeTerlihat l))
```

Catatan soal 8: `words ""` menghasilkan `[]`, jadi baris kosong otomatis menjadi list kosong. `formatHasil` juga fungsi murni, jadi bisa dites terpisah.

</details>

### Level 4: gabungan

**9. Matriks.** Baris pertama berisi `R C` (banyak baris dan kolom), lalu `R` baris berisi `C` bilangan. Cetak jumlah setiap baris, lalu jumlah setiap kolom.

| Input | Output |
|---|---|
| `2 3`<br>`1 2 3`<br>`4 5 6` | `Jumlah baris: 6 15`<br>`Jumlah kolom: 5 7 9` |

*Petunjuk:* `transpose` dari `Data.List` mengubah baris menjadi kolom: `transpose [[1,2,3],[4,5,6]] = [[1,4],[2,5],[3,6]]`. Bisa juga pakai `transposeMatrix` dari `Praktikum3_2025`.

**10. ATM sederhana.** Kembangkan program saldo di bagian 13 supaya:
- perintah `ambil x` ditolak dengan pesan `Saldo tidak cukup` kalau `x` lebih besar dari saldo,
- perintah `keluar` mencetak saldo akhir sebelum `Sampai jumpa!`.

| Input | Output |
|---|---|
| `tambah 50`<br>`ambil 80`<br>`ambil 20`<br>`keluar` | `Saldo tidak cukup`<br>`Saldo akhir: 30`<br>`Sampai jumpa!` |

<details>
<summary>Jawaban level 4</summary>

```haskell
-- Soal 9
import Control.Monad (replicateM)
import Data.List (transpose)

main :: IO ()
main = do
    ukuran <- getLine
    let [r, _] = map read (words ukuran) :: [Int]
    barisBaris <- replicateM r getLine
    let m = map (map read . words) barisBaris :: [[Int]]
    putStrLn ("Jumlah baris: " ++ unwords (map show (map sum m)))
    putStrLn ("Jumlah kolom: " ++ unwords (map show (map sum (transpose m))))
```

Catatan soal 9:
- `let [r, _] = ...` membongkar list dua elemen. `_` untuk `C` yang tidak dipakai, karena banyak kolom sudah terlihat dari isi baris.
- `map read . words` adalah komposisi fungsi: pecah baris jadi kata, lalu ubah setiap kata jadi angka. Ini diterapkan ke setiap baris dengan `map` luar.

```haskell
-- Soal 10
loop :: Int -> IO ()
loop saldo = do
    perintah <- getLine
    case words perintah of
        ["tambah", x] -> loop (saldo + read x)
        ["ambil", x]  ->
            if read x > saldo
                then do
                    putStrLn "Saldo tidak cukup"
                    loop saldo
                else loop (saldo - read x)
        ["lihat"]     -> do
            putStrLn ("Saldo: " ++ show saldo)
            loop saldo
        ["keluar"]    -> do
            putStrLn ("Saldo akhir: " ++ show saldo)
            putStrLn "Sampai jumpa!"
        _             -> do
            putStrLn "Perintah tidak dikenal"
            loop saldo

main :: IO ()
main = loop 0
```

Catatan soal 10: pada `ambil` yang ditolak, `loop` dipanggil lagi dengan `saldo` yang **sama**. Saldo "tidak berubah" artinya parameter yang sama diteruskan.

</details>

---

> 💡 **Ringkasan satu kalimat:** fungsi murni menghitung, aksi `IO` berinteraksi dengan dunia luar, dan `do` + `<-` adalah satu-satunya jembatan dari dunia `IO` ke nilai biasa.
