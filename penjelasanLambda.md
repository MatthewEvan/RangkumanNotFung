# Fungsi sebagai Nilai, Lambda, dan Higher Order Function

Catatan persiapan Praktikum 5 IF2110. Semua contoh kode di bawah sudah dites dengan GHC 9.10.3, dan output yang ditulis adalah output aslinya.

---

## Daftar Isi

1. [Fungsi adalah nilai](#1-fungsi-adalah-nilai)
2. [Currying dan partial application](#2-currying-dan-partial-application)
3. [Fungsi lambda (fungsi anonim)](#3-fungsi-lambda-fungsi-anonim)
4. [Operator section](#4-operator-section)
5. [Higher Order Function (HOF)](#5-higher-order-function-hof)
6. [HOF bawaan: map, filter, fold, zipWith, dll.](#6-hof-bawaan)
7. [Menulis HOF sendiri dengan rekursi](#7-menulis-hof-sendiri-dengan-rekursi)
8. [Komposisi fungsi (`.`) dan operator `$`](#8-komposisi-fungsi--dan-operator-)
9. [Menulis ulang soal praktikum lama dengan HOF](#9-menulis-ulang-soal-praktikum-lama-dengan-hof)
10. [Contoh soal lengkap gaya praktikum](#10-contoh-soal-lengkap-gaya-praktikum)
11. [Kesalahan umum](#11-kesalahan-umum)
12. [Cheat sheet](#12-cheat-sheet)

---

## 1. Fungsi adalah nilai

Di Haskell, fungsi diperlakukan **sama seperti nilai lain** (`Int`, `Bool`, list, dan seterusnya). Istilahnya fungsi adalah *first-class citizen*. Artinya fungsi bisa:

| Bisa... | Contoh |
|---|---|
| diberi nama | `tambahLima = tambah 5` |
| dikirim sebagai **parameter** ke fungsi lain | `map tambahLima [1,2,3]` |
| dikembalikan sebagai **hasil** fungsi lain | `pembuatPengali 3` menghasilkan fungsi |
| disimpan dalam list/tuple | `[(+1), (*2), abs]` |

Tipe sebuah fungsi juga ditulis dengan `->`. Misalnya `Int -> Int` adalah tipe "fungsi yang menerima `Int` dan menghasilkan `Int`". Tipe fungsi bisa muncul **di dalam** signature fungsi lain, dengan **tanda kurung**:

```haskell
applyTwice :: (Int -> Int) -> Int -> Int
--            ^^^^^^^^^^^^
--            parameter pertama adalah sebuah FUNGSI
```

---

## 2. Currying dan partial application

Kamu sudah sering menulis fungsi dengan banyak parameter:

```haskell
tambah :: Int -> Int -> Int
tambah a b = a + b
```

Sebenarnya Haskell membaca tipe itu sebagai:

```haskell
tambah :: Int -> (Int -> Int)
```

yaitu **fungsi yang menerima satu `Int`, lalu mengembalikan fungsi lain** yang menerima `Int` berikutnya. Ini disebut **currying**.

Akibatnya, sebuah fungsi boleh dipanggil dengan **sebagian** argumennya saja (*partial application*), dan hasilnya adalah fungsi baru:

```haskell
tambahLima :: Int -> Int
tambahLima = tambah 5        -- belum lengkap, jadi hasilnya fungsi "tambah 5 ke sesuatu"

-- ghci> tambahLima 3
-- 8
```

Inilah alasan `map (tambah 5) [1,2,3]` bisa ditulis: `tambah 5` sendiri sudah berupa fungsi.

---

## 3. Fungsi lambda (fungsi anonim)

**Lambda** adalah fungsi **tanpa nama** yang ditulis langsung di tempat ia dipakai. Bentuknya:

```
\parameter1 parameter2 ... -> ekspresi
```

Simbol `\` dibaca "lambda" (karena mirip huruf λ).

### Perbandingan dengan fungsi biasa

```haskell
-- Fungsi biasa (punya nama)
kuadrat :: Int -> Int
kuadrat x = x * x

-- Lambda yang setara (tanpa nama)
\x -> x * x
```

### Contoh pemakaian

```haskell
-- Langsung dipanggil
-- ghci> (\x -> x * 2) 5
-- 10

-- Dua parameter
-- ghci> (\a b -> a + b) 3 4
-- 7

-- Paling sering: dikirim ke HOF
-- ghci> map (\x -> x * 2) [1,2,3]
-- [2,4,6]

-- ghci> filter (\x -> x `mod` 3 == 0) [1..12]
-- [3,6,9,12]
```

### Lambda dengan pattern matching

Parameter lambda juga bisa berupa pola, misalnya tuple:

```haskell
-- ambil nama mahasiswa yang nilainya >= 60
lulus :: [(String, Int)] -> [String]
lulus daftar = map fst (filter (\(_, nilai) -> nilai >= 60) daftar)

-- ghci> lulus [("Ani", 80), ("Budi", 55), ("Caca", 60)]
-- ["Ani","Caca"]
```

> ⚠️ Lambda hanya punya **satu** pola. Lambda tidak bisa punya beberapa kasus seperti fungsi biasa. Kalau butuh percabangan, pakai `if-then-else` di dalam lambda, atau buat fungsi bernama dengan guard.

```haskell
map (\x -> if x < 0 then 0 else x) [-2, 5, -1, 3]
-- [0,5,0,3]
```

### Kapan memakai lambda vs fungsi bernama?

| Pakai **lambda** jika... | Pakai **fungsi bernama** jika... |
|---|---|
| logikanya pendek (1 baris) | logikanya panjang / banyak kasus |
| hanya dipakai sekali | dipakai di beberapa tempat |
| | perlu spesifikasi/komentar sendiri (sesuai format DEFINISI DAN SPESIFIKASI) |

### Fungsi yang mengembalikan lambda

```haskell
pembuatPengali :: Int -> (Int -> Int)
-- pembuatPengali n menghasilkan FUNGSI yang mengalikan input dengan n
pembuatPengali n = \x -> x * n

-- ghci> (pembuatPengali 3) 7
-- 21
-- ghci> map (pembuatPengali 10) [1,2,3]
-- [10,20,30]
```

---

## 4. Operator section

Operator seperti `+`, `*`, `==`, `>` juga fungsi. Dengan memberi kurung dan **satu** operand, kita mendapat fungsi baru. Ini cara singkat menulis lambda sederhana:

| Section | Setara dengan lambda | Contoh |
|---|---|---|
| `(+1)` | `\x -> x + 1` | `map (+1) [1,2,3]` → `[2,3,4]` |
| `(*2)` | `\x -> x * 2` | |
| `(2^)` | `\x -> 2 ^ x` | `map (2^) [1,2,3]` → `[2,4,8]` |
| `(^2)` | `\x -> x ^ 2` | |
| `(> 0)` | `\x -> x > 0` | `all (> 0) [1,2,3]` → `True` |
| `(== 7)` | `\x -> x == 7` | |
| `` (`div` 2) `` | `\x -> x `div` 2` | `map (`div` 2) [10,11,12]` → `[5,5,6]` |
| `(+)` | `\a b -> a + b` | `foldr (+) 0 [1,2,3]` → `6` |

> ⚠️ **Jebakan minus:** `(-1)` **bukan** fungsi, melainkan angka negatif satu. Untuk "kurangi 1" pakai `subtract 1` atau `\x -> x - 1`.
> ```haskell
> map (subtract 1) [5,6,7]   -- [4,5,6]
> ```

---

## 5. Higher Order Function (HOF)

**Higher Order Function** adalah fungsi yang memenuhi minimal satu dari syarat berikut:

1. **menerima fungsi sebagai parameter**, atau
2. **mengembalikan fungsi sebagai hasil**.

Contoh paling sederhana:

```haskell
applyTwice :: (Int -> Int) -> Int -> Int
-- applyTwice f x menerapkan fungsi f sebanyak dua kali pada x
applyTwice f x = f (f x)

-- ghci> applyTwice (+3) 10
-- 16
-- ghci> applyTwice (\x -> x * x) 3
-- 81
```

`pembuatPengali` di bagian 3 juga HOF karena mengembalikan fungsi.

**Kenapa berguna?** Banyak fungsi rekursi list di praktikum sebelumnya punya **pola yang sama persis** dan hanya berbeda di "apa yang dilakukan ke tiap elemen". HOF memisahkan **pola rekursinya** (ditulis sekali) dari **aksinya** (dikirim sebagai fungsi). Jadi kode menjadi lebih pendek dan lebih jarang salah.

---

## 6. HOF bawaan

Semua fungsi ini sudah tersedia di Prelude (tidak perlu `import`).

### `map`: terapkan fungsi ke setiap elemen

```haskell
map :: (a -> b) -> [a] -> [b]
```

```haskell
map (*2) [1,2,3]              -- [2,4,6]
map (\x -> x * x) [1,2,3]     -- [1,4,9]
map head [[1,2],[3,4],[5,6]]  -- [1,3,5]
map length ["ab", "cde"]      -- [2,3]
```

Panjang list hasil **selalu sama** dengan list awal.

### `filter`: ambil elemen yang memenuhi syarat

```haskell
filter :: (a -> Bool) -> [a] -> [a]
```

Fungsi yang mengembalikan `Bool` disebut **predikat**.

```haskell
filter even [1..10]                    -- [2,4,6,8,10]
filter odd [1..10]                     -- [1,3,5,7,9]
filter (\x -> x `mod` 3 == 0) [1..12]  -- [3,6,9,12]
filter (/= ' ') "a b c"                -- "abc"
```

### `foldr` dan `foldl`: "melipat" list menjadi satu nilai

```haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
--       ^^^^^^^^^^^^^    ^
--       fungsi gabung    nilai awal (basis)
```

`foldr` adalah **pola rekursi list yang paling umum**. Cara membacanya:

```
foldr f z [x1, x2, x3]  =  f x1 (f x2 (f x3 z))      -- dikerjakan dari KANAN
foldl f z [x1, x2, x3]  =  f (f (f z x1) x2) x3      -- dikerjakan dari KIRI
```

Contoh:

```haskell
foldr (+) 0 [1,2,3,4]     -- 1 + (2 + (3 + (4 + 0))) = 10
foldr (*) 1 [1,2,3,4]     -- 24

-- Untuk operasi tidak komutatif, arah berpengaruh!
foldl (-) 10 [1,2,3]      -- ((10 - 1) - 2) - 3 = 4
foldr (-) 10 [1,2,3]      -- 1 - (2 - (3 - 10)) = -8
```

Lambda di dalam fold menerima **dua** parameter: elemen dan akumulator. Urutannya berbeda untuk `foldr` dan `foldl`:

```haskell
foldr (\elemen acc -> ...) awal list    -- elemen DULU, lalu acc
foldl (\acc elemen -> ...) awal list    -- acc DULU, lalu elemen
```

```haskell
panjang :: [a] -> Int
panjang li = foldr (\_ acc -> acc + 1) 0 li
-- panjang "haskell" = 7

balik :: [a] -> [a]
balik li = foldl (\acc x -> x : acc) [] li
-- balik [1,2,3] = [3,2,1]
```

Ada juga `foldr1`/`foldl1` yang **tidak butuh nilai awal**. Fungsi ini memakai elemen pertama/terakhir sebagai nilai awal, jadi list **tidak boleh kosong**:

```haskell
maksList :: [Int] -> Int
maksList li = foldr1 (\a b -> if a > b then a else b) li
-- maksList [3,9,2] = 9
```

### `zipWith`: gabungkan dua list elemen demi elemen

```haskell
zipWith :: (a -> b -> c) -> [a] -> [b] -> [c]
```

```haskell
zipWith (+) [1,2,3] [10,20,30]            -- [11,22,33]
zipWith (\a b -> a * b) [1,2,3] [4,5,6]   -- [4,10,18]
```

Kalau panjangnya berbeda, hasilnya mengikuti list yang **lebih pendek**.

### HOF lain yang sering muncul

| Fungsi | Arti | Contoh → hasil |
|---|---|---|
| `any p l` | ada elemen yang memenuhi `p`? | `any even [1,3,5]` → `False` |
| `all p l` | semua elemen memenuhi `p`? | `all (> 0) [1,2,3]` → `True` |
| `takeWhile p l` | ambil dari depan **selama** `p` benar | `takeWhile (< 4) [1..10]` → `[1,2,3]` |
| `dropWhile p l` | buang dari depan selama `p` benar | `dropWhile (< 4) [1..6]` → `[4,5,6]` |
| `flip f a b` | tukar urutan argumen: `f b a` | `flip (-) 1 10` → `9` |
| `uncurry f (a,b)` | panggil `f a b` dari tuple | `map (uncurry (+)) [(1,2),(3,4)]` → `[3,7]` |

Fungsi non-HOF yang sering dipasangkan dengan HOF: `sum`, `product`, `length`, `maximum`, `minimum`, `fst`, `snd`.

---

## 7. Menulis HOF sendiri dengan rekursi

Supaya HOF tidak terasa seperti sulap, lihat isinya. Semuanya hanya **rekursi list biasa** seperti yang kamu tulis di Praktikum 3, ditambah satu parameter fungsi.

Notasi `(x:xs)` adalah pattern matching list: `x` = `head`, `xs` = `tail`.

```haskell
myMap :: (a -> b) -> [a] -> [b]
myMap _ []     = []                    -- basis
myMap f (x:xs) = f x : myMap f xs      -- terapkan f ke head, rekursi ke tail

myFilter :: (a -> Bool) -> [a] -> [a]
myFilter _ [] = []
myFilter p (x:xs)
    | p x       = x : myFilter p xs    -- lolos syarat → simpan
    | otherwise = myFilter p xs        -- tidak lolos → buang

myFoldr :: (a -> b -> b) -> b -> [a] -> b
myFoldr _ z []     = z
myFoldr f z (x:xs) = f x (myFoldr f z xs)
```

Versi yang sama dengan gaya `isEmpty`/`head`/`tail` seperti di `Matrix.hs`:

```haskell
mapIntLama :: (Int -> Int) -> [Int] -> [Int]
mapIntLama f li
    | null li   = []
    | otherwise = f (head li) : mapIntLama f (tail li)
```

> 💡 **Huruf kecil `a`, `b` di tipe** adalah *variabel tipe* (polimorfisme). Artinya "tipe apa saja". `myMap :: (a -> b) -> [a] -> [b]` bisa dipakai untuk `[Int]`, `[Char]`, `[[Int]]`, dan lainnya. Kalau soal meminta tipe konkret (misalnya `[Int]`), tulis saja tipe konkretnya.

---

## 8. Komposisi fungsi (`.`) dan operator `$`

### Komposisi `.`

`(f . g) x` sama dengan `f (g x)`: jalankan `g` dulu, lalu `f`. Dibaca **dari kanan ke kiri**, seperti komposisi fungsi di matematika (f ∘ g).

```haskell
(map (* 2) . filter odd) [1..6]
-- filter odd dulu → [1,3,5], lalu map (*2) → [2,6,10]
```

### Operator `$`

`f $ x` sama dengan `f x`, tetapi `$` punya prioritas paling rendah. Gunanya **mengurangi tanda kurung**:

```haskell
sum (map (* 2) [1,2,3])
sum $ map (* 2) [1,2,3]     -- sama saja, hasilnya 12
```

### Tiga cara menulis fungsi yang sama

Soal: *jumlahkan kuadrat dari semua bilangan genap dalam list.*

```haskell
-- (1) Kurung biasa: paling eksplisit, paling aman untuk pemula
jumlahKuadratGenap :: [Int] -> Int
jumlahKuadratGenap li = foldr (+) 0 (map (\x -> x * x) (filter even li))

-- (2) Dengan komposisi (gaya "point-free", parameter li tidak ditulis)
jumlahKuadratGenap' :: [Int] -> Int
jumlahKuadratGenap' = sum . map (^2) . filter even

-- ghci> jumlahKuadratGenap [1..6]
-- 56          -- 2² + 4² + 6² = 4 + 16 + 36
```

Bacanya dari dalam/kanan ke luar/kiri: **filter** genap → **map** kuadrat → **jumlahkan**.

---

## 9. Menulis ulang soal praktikum lama dengan HOF

Ini menunjukkan seberapa banyak kode yang bisa dipangkas.

### `count` (Praktikum3_2025/CountOccurrence.hs)

Versi lama butuh dua fungsi rekursif (`countInList` dan `count`). Dengan HOF cukup satu baris:

```haskell
count :: [[Int]] -> Int -> Int
-- count m n menghitung berapa kali n muncul di dalam list of list m
count m n = sum (map (\li -> length (filter (== n) li)) m)

-- ghci> count [[1,2,1],[3],[1,4]] 1
-- 3
```

Cara membaca: untuk **setiap** baris `li` (`map`), ambil elemen yang `== n` (`filter`), hitung banyaknya (`length`), lalu **jumlahkan** semuanya (`sum`).

### `transposeMatrix` (Praktikum3_2025/TransposeMatrix.hs)

Fungsi ini sebenarnya sudah memakai HOF (`map head`, `map tail`):

```haskell
transposeMatrix :: [[Int]] -> [[Int]]
transposeMatrix m
    | null m || null (head m) = []
    | otherwise = map head m : transposeMatrix (map tail m)

-- ghci> transposeMatrix [[1,2,3],[4,5,6],[7,8,9]]
-- [[1,4,7],[2,5,8],[3,6,9]]
```

### Operasi matriks dengan HOF bersarang

```haskell
-- kalikan setiap elemen matriks dengan k
skalarMatrix :: Int -> [[Int]] -> [[Int]]
skalarMatrix k m = map (map (* k)) m
-- map luar: untuk setiap baris; map dalam: untuk setiap elemen

-- jumlahkan dua matriks berukuran sama
tambahMatrix :: [[Int]] -> [[Int]] -> [[Int]]
tambahMatrix m1 m2 = zipWith (zipWith (+)) m1 m2

-- ghci> skalarMatrix 2 [[1,2],[3,4]]
-- [[2,4],[6,8]]
-- ghci> tambahMatrix [[1,2],[3,4]] [[10,20],[30,40]]
-- [[11,22],[33,44]]
```

### `hitungDigit` (Praktikum3/DigitAnomali.hs)

Bilangan diubah dulu menjadi list digit, lalu memakai `filter`:

```haskell
hitungDigitHOF :: Int -> Int -> Int
hitungDigitHOF n d = length (filter (== d) (digit n))
    where
        digit x
            | x < 10    = [x]
            | otherwise = digit (x `div` 10) ++ [x `mod` 10]

-- ghci> hitungDigitHOF 707070 7
-- 3
```

---

## 10. Contoh soal lengkap gaya praktikum

Template yang sama dengan file-file praktikummu, dan bisa langsung disalin ke file `.hs`:

```haskell
module NilaiMahasiswa where

-- NILAI MAHASISWA
-- DEFINISI DAN SPESIFIKASI
naikkanNilai :: Int -> [Int] -> [Int]
-- naikkanNilai k li menambahkan k ke setiap nilai di li,
-- dengan nilai maksimum 100

jumlahLulus :: [Int] -> Int
-- jumlahLulus li menghasilkan banyaknya nilai yang >= 60

rataRata :: [Int] -> Float
-- rataRata li menghasilkan rata-rata nilai di li. Prasyarat: li tidak kosong

-- REALISASI
naikkanNilai k li = map (\x -> if x + k > 100 then 100 else x + k) li

jumlahLulus li = length (filter (>= 60) li)

rataRata li = fromIntegral (foldr (+) 0 li) / fromIntegral (length li)

-- APLIKASI
-- naikkanNilai 10 [50, 95, 70]   => [60,100,80]
-- jumlahLulus [50, 95, 70]       => 2
-- rataRata [50, 95, 70]          => 71.666664
```

### Latihan mandiri

Coba kerjakan dengan HOF + lambda (jawaban ada di bawah, jangan diintip dulu 😄):

1. `kuadratkanSemua :: [Int] -> [Int]`, mengkuadratkan setiap elemen.
2. `hapusNegatif :: [Int] -> [Int]`, membuang elemen negatif.
3. `hasilKali :: [Int] -> Int`, mengalikan semua elemen dengan `foldr`.
4. `jarak :: [Int] -> [Int] -> [Int]`, selisih mutlak elemen-elemen yang seposisi.
5. `adaKelipatan :: Int -> [Int] -> Bool`, apakah ada elemen yang kelipatan `k`.
6. `sumKolom :: [[Int]] -> [Int]`, jumlah setiap kolom matriks (petunjuk: gunakan `transposeMatrix` lalu `map`).

<details>
<summary>Jawaban</summary>

```haskell
kuadratkanSemua li = map (\x -> x * x) li
hapusNegatif li    = filter (\x -> x >= 0) li
hasilKali li       = foldr (*) 1 li
jarak l1 l2        = zipWith (\a b -> abs (a - b)) l1 l2
adaKelipatan k li  = any (\x -> x `mod` k == 0) li
sumKolom m         = map sum (transposeMatrix m)
```

</details>

---

## 11. Kesalahan umum

| Kesalahan | Penyebab | Perbaikan |
|---|---|---|
| `applyTwice :: Int -> Int -> Int -> Int` | lupa kurung pada tipe parameter fungsi | `(Int -> Int) -> Int -> Int` |
| `map (-1) [1,2,3]` error | `(-1)` dibaca angka negatif | `map (subtract 1) ...` atau `map (\x -> x - 1) ...` |
| `map \x -> x+1 [1,2]` error | lambda tidak dikurung | `map (\x -> x + 1) [1,2]` |
| `foldl (\x acc -> ...)` hasilnya aneh | urutan parameter lambda tertukar | `foldl` → `\acc x`, `foldr` → `\x acc` |
| `foldr1 max []` crash | `foldr1`/`foldl1`/`head` pada list kosong | cek `null` dulu atau pakai `foldr` dengan nilai awal |
| `sum li / length li` error tipe | `/` butuh `Float`, `length` menghasilkan `Int` | `fromIntegral (sum li) / fromIntegral (length li)` |
| Komposisi salah urutan | `f . g` menjalankan `g` **dulu** | baca dari kanan ke kiri |

> 💡 Di GHC 9.10, memakai `head`/`tail` akan memunculkan **warning** `[-Wx-partial]`. Itu hanya peringatan (bukan error) bahwa fungsi tersebut crash pada list kosong. Kode tetap jalan. Pattern matching `(x:xs)` adalah alternatif yang tidak memunculkan warning.

---

## 12. Cheat sheet

```haskell
-- LAMBDA
\x -> x * 2                    -- satu parameter
\a b -> a + b                  -- dua parameter
\(a, b) -> a + b               -- pattern tuple
\_ -> 0                        -- abaikan parameter

-- SECTION
(+1)  (*2)  (^2)  (2^)  (> 0)  (== x)  (`div` 2)  (+)

-- HOF BAWAAN
map     f   l                  -- ubah setiap elemen
filter  p   l                  -- saring elemen
foldr   f z l                  -- lipat dari kanan,  f :: elemen -> acc -> acc
foldl   f z l                  -- lipat dari kiri,   f :: acc -> elemen -> acc
zipWith f l1 l2                -- gabung dua list
any p l / all p l              -- ada / semua
takeWhile p l / dropWhile p l

-- KOMPOSISI
(f . g) x  ==  f (g x)
f $ g x    ==  f (g x)

-- TIPE FUNGSI SEBAGAI PARAMETER: WAJIB DIKURUNG
hof :: (Int -> Int) -> [Int] -> [Int]
```

**Cara cepat memilih HOF:**

- Hasilnya list **sepanjang** input, tiap elemen diubah → `map`
- Hasilnya list **lebih pendek**, sebagian elemen dibuang → `filter`
- Hasilnya **satu nilai** (jumlah, hasil kali, maks, panjang) → `foldr` / `foldl` (atau `sum`, `product`, `maximum`)
- Dua list diproses **berpasangan** → `zipWith`
- Hasilnya **Bool** tentang isi list → `any` / `all`
