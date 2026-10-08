# Pohon Biner (Binary Tree) dalam Paradigma Fungsional dan Haskell

Catatan persiapan praktikum pohon biner IF2110. Semua contoh kode di bawah sudah dites dengan GHC 9.10.3, dan output yang ditulis adalah output aslinya.

> Materi lambda dan HOF ada di [`penjelasanLambda.md`](penjelasanLambda.md). Contoh fold pada pohon juga ada di [`Praktikum5_2025/TreeFold.hs`](Praktikum5_2025/TreeFold.hs) dan pembahasannya di [`Praktikum5_2025/penjelasansoal.md`](Praktikum5_2025/penjelasansoal.md).

---

## Daftar Isi

1. [Konsep pohon biner](#1-konsep-pohon-biner)
2. [Istilah penting](#2-istilah-penting)
3. [Pohon biner dalam notasi fungsional](#3-pohon-biner-dalam-notasi-fungsional)
4. [Mendefinisikan pohon biner di Haskell](#4-mendefinisikan-pohon-biner-di-haskell)
5. [Membuat dan menampilkan pohon](#5-membuat-dan-menampilkan-pohon)
6. [Dua gaya menulis: pattern matching vs selektor](#6-dua-gaya-menulis-pattern-matching-vs-selektor)
7. [Pola rekursi pohon](#7-pola-rekursi-pohon)
8. [Fungsi-fungsi dasar](#8-fungsi-fungsi-dasar)
9. [Traversal: pre-order, in-order, post-order](#9-traversal-pre-order-in-order-post-order)
10. [Fungsi yang menghasilkan pohon](#10-fungsi-yang-menghasilkan-pohon)
11. [Binary Search Tree (BST)](#11-binary-search-tree-bst)
12. [Pohon dan HOF: `mapTree` dan `foldTree`](#12-pohon-dan-hof-maptree-dan-foldtree)
13. [Strategi mengerjakan soal pohon](#13-strategi-mengerjakan-soal-pohon)
14. [Kesalahan umum](#14-kesalahan-umum)
15. [Cheat sheet](#15-cheat-sheet)
16. [Latihan soal](#16-latihan-soal)

---

## 1. Konsep pohon biner

**Pohon** adalah struktur data bertingkat: ada satu simpul paling atas (akar), dan setiap simpul bisa punya anak. **Pohon biner** adalah pohon yang setiap simpulnya punya **paling banyak dua anak**, yaitu anak kiri dan anak kanan.

```
        4          ← akar
       / \
      2   6        ← anak dari 4
     / \   \
    1   3   7      ← 1, 3, 7 adalah daun (tidak punya anak)
```

Hal terpenting untuk pemrograman fungsional: pohon biner didefinisikan secara **rekursif**.

> Sebuah pohon biner adalah salah satu dari:
> 1. **pohon kosong**, atau
> 2. sebuah **akar** beserta **subpohon kiri** dan **subpohon kanan**, yang masing-masing juga pohon biner.

Bandingkan dengan list yang sudah kamu kenal:

| | List | Pohon biner |
|---|---|---|
| Kasus kosong | `[]` | pohon kosong |
| Kasus tidak kosong | elemen pertama + **satu** sisa list | akar + **dua** subpohon (kiri dan kanan) |
| Rekursi | ke `tail` (sekali) | ke kiri **dan** ke kanan (dua kali) |

Karena definisinya rekursif, hampir semua fungsi pada pohon juga ditulis secara rekursif dengan pola yang sama: tangani pohon kosong, lalu proses akar dan gabungkan hasil rekursi dari kedua subpohon.

---

## 2. Istilah penting

Menggunakan pohon contoh di atas:

| Istilah | Arti | Contoh |
|---|---|---|
| **Akar** (*root*) | simpul paling atas | `4` |
| **Simpul** (*node*) / elemen | setiap titik di pohon | `4, 2, 6, 1, 3, 7` |
| **Anak** (*child*) | simpul langsung di bawah | anak `2` adalah `1` dan `3` |
| **Orang tua** (*parent*) | simpul langsung di atas | orang tua `7` adalah `6` |
| **Daun** (*leaf*) | simpul tanpa anak | `1, 3, 7` |
| **Subpohon kiri / kanan** | pohon yang berakar di anak kiri / kanan | subpohon kiri `4` berakar di `2` |
| **Level** | tingkat simpul, akar = level 1 | `2` dan `6` ada di level 2 |
| **Kedalaman / tinggi** (*depth*) | banyak level pada pohon | `3` |
| **Pohon uner kiri** | akar hanya punya subpohon kiri | |
| **Pohon uner kanan** | akar hanya punya subpohon kanan | subpohon berakar `6` |
| **Pohon biner** (dalam arti predikat `isBiner`) | akar punya subpohon kiri **dan** kanan | pohon berakar `4` |

> ⚠️ **Konvensi kedalaman bisa berbeda.** Di catatan ini pohon kosong punya kedalaman `0` dan pohon satu elemen punya kedalaman `1`, sama seperti contoh di `TreeFold.hs`. Ada buku yang menghitung banyaknya *sisi* sehingga pohon satu elemen bernilai `0`. Selalu ikuti contoh di soal.

---

## 3. Pohon biner dalam notasi fungsional

Di kuliah, tipe bentukan ditulis lengkap dengan **konstruktor**, **selektor**, dan **predikat**. Bentuknya kira-kira seperti ini (format sama dengan `NotasiFungsional/`):

```
DEFINISI DAN SPESIFIKASI TYPE
    type BinTree : [ ] atau < A : elemen, L : BinTree, R : BinTree >
    { Pohon biner kosong ditulis [ ].
      Pohon tidak kosong terdiri dari akar A, subpohon kiri L, dan subpohon kanan R. }

DEFINISI DAN SPESIFIKASI KONSTRUKTOR
    makeBinTree : elemen, BinTree, BinTree -> BinTree
    { makeBinTree(A, L, R) membentuk pohon dengan akar A, subpohon kiri L, subpohon kanan R }

DEFINISI DAN SPESIFIKASI SELEKTOR
    akar  : BinTree tidak kosong -> elemen      { akar(P) adalah akar dari P }
    left  : BinTree tidak kosong -> BinTree     { left(P) adalah subpohon kiri P }
    right : BinTree tidak kosong -> BinTree     { right(P) adalah subpohon kanan P }

DEFINISI DAN SPESIFIKASI PREDIKAT
    isTreeEmpty : BinTree -> boolean   { true jika P kosong }
    isOneElmt   : BinTree -> boolean   { true jika P hanya terdiri dari satu elemen (daun) }
    isUnerLeft  : BinTree -> boolean   { true jika P hanya punya subpohon kiri }
    isUnerRight : BinTree -> boolean   { true jika P hanya punya subpohon kanan }
    isBiner     : BinTree -> boolean   { true jika P punya subpohon kiri dan kanan }
```

Pola pikirnya:
- **Konstruktor** membangun pohon dari bagian-bagiannya.
- **Selektor** mengambil bagian dari pohon. Selektor hanya boleh dipakai pada pohon **tidak kosong**.
- **Predikat** mengecek bentuk pohon, dan dipakai sebagai syarat di analisis kasus.

Fungsi-fungsi soal (misalnya `nbElmt`, `depth`) kemudian ditulis **hanya** dengan memakai konstruktor, selektor, dan predikat ini. Begitulah template `TreeFold.hs` disusun.

### Pohon boleh kosong vs pohon tidak kosong

Soal kadang memberi **prasyarat "pohon tidak kosong"**. Ini mengubah cara menulis rekursi:

| | Pohon boleh kosong | Pohon tidak kosong |
|---|---|---|
| Basis | pohon kosong | pohon **satu elemen** (daun) |
| Kasus rekursi | selalu rekursi ke kiri dan kanan | perlu cek dulu: uner kiri, uner kanan, atau biner? |

Contoh lengkapnya ada di `maxTree` di [bagian 8](#maxtree-contoh-pohon-tidak-kosong).

---

## 4. Mendefinisikan pohon biner di Haskell

Haskell punya kata kunci `data` untuk membuat tipe bentukan sendiri:

```haskell
data BinTree a = Empty | Node a (BinTree a) (BinTree a)
    deriving (Show, Eq)
```

Baca bagian per bagian:

| Bagian | Arti |
|---|---|
| `data BinTree a` | membuat tipe baru bernama `BinTree`. `a` adalah **variabel tipe**: isinya bisa `Int`, `Char`, `String`, dan lain-lain |
| `=` ... `\|` ... | tipe ini punya **dua bentuk** (konstruktor), dipisah `\|` ("atau") |
| `Empty` | konstruktor pertama: pohon kosong, tidak membawa data |
| `Node a (BinTree a) (BinTree a)` | konstruktor kedua: simpul yang membawa **akar** bertipe `a`, **subpohon kiri**, dan **subpohon kanan** |
| `deriving (Show, Eq)` | minta Haskell otomatis membuat cara **menampilkan** pohon (`Show`) dan **membandingkan** dua pohon dengan `==` (`Eq`) |

Definisi ini sama persis dengan definisi rekursif di bagian 1: `Node` berisi `BinTree a` lagi di dalamnya.

Konstruktor di Haskell sebenarnya juga **fungsi**:

```haskell
-- ghci> :t Node
-- Node :: a -> BinTree a -> BinTree a -> BinTree a
```

Jadi `Node` sudah berperan sebagai konstruktor `makeBinTree` dari notasi fungsional.

### Tipe konkret vs polimorfik

- `BinTree Int`: pohon yang isinya `Int`.
- `BinTree a`: pohon dengan isi tipe apa saja (polimorfik).

Kalau soal meminta pohon `Int`, tulis `BinTree Int`. **`BinTree` saja tanpa parameter tidak valid** sebagai tipe nilai.

### Konstruktor, selektor, dan predikat versi Haskell

Ini isi template seperti di `TreeFold.hs`, ditulis polimorfik:

```haskell
-- KONSTRUKTOR
makeBinTree :: a -> BinTree a -> BinTree a -> BinTree a
makeBinTree a l r = Node a l r

-- SELEKTOR
akar :: BinTree a -> a
akar (Node x _ _) = x
akar Empty        = error "akar: pohon kosong"

left :: BinTree a -> BinTree a
left (Node _ l _) = l
left Empty        = error "left: pohon kosong"

right :: BinTree a -> BinTree a
right (Node _ _ r) = r
right Empty        = error "right: pohon kosong"

-- PREDIKAT
isTreeEmpty :: BinTree a -> Bool
isTreeEmpty Empty = True
isTreeEmpty _     = False

isOneElmt :: BinTree a -> Bool
isOneElmt (Node _ Empty Empty) = True
isOneElmt _                    = False
```

Perhatikan pola `(Node x _ _)`. Ini **pattern matching** pada konstruktor: Haskell mengecek apakah pohonnya berbentuk `Node`, lalu sekaligus memberi nama pada isinya. `_` berarti "bagian ini tidak dipakai".

---

## 5. Membuat dan menampilkan pohon

### Menulis pohon secara langsung

Pohon contoh dari bagian 1:

```
        4
       / \
      2   6
     / \   \
    1   3   7
```

```haskell
t1 :: BinTree Int
t1 = Node 4
        (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty))
        (Node 6 Empty (Node 7 Empty Empty))
```

Setiap subpohon yang menjadi argumen **wajib dikurung**. Tanpa kurung, Haskell mengira `Node`, `1`, `Empty`, ... adalah argumen terpisah.

Menulis `Node x Empty Empty` berulang-ulang itu melelahkan, jadi biasanya dibuat fungsi pembantu:

```haskell
daun :: a -> BinTree a
daun x = Node x Empty Empty

t1 :: BinTree Int
t1 = Node 4
        (Node 2 (daun 1) (daun 3))
        (Node 6 Empty (daun 7))
```

Satu pohon lagi untuk contoh-contoh berikutnya (bukan BST):

```
      5
     / \
    8   3
   /
  9
```

```haskell
t2 :: BinTree Int
t2 = Node 5 (Node 8 (daun 9) Empty) (daun 3)
```

### Cara Haskell menampilkan pohon

Karena ada `deriving Show`, ghci bisa mencetak pohon. Formatnya sama dengan cara menulisnya:

```haskell
-- ghci> daun 1
-- Node 1 Empty Empty
-- ghci> t1
-- Node 4 (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty)) (Node 6 Empty (Node 7 Empty Empty))
```

> 💡 Cara membaca output panjang: setiap `Node` diikuti **akar**, lalu **kurung pertama = subpohon kiri**, **kurung kedua = subpohon kanan**. Menggambar ulang di kertas sangat membantu.

---

## 6. Dua gaya menulis: pattern matching vs selektor

Fungsi yang sama bisa ditulis dengan dua gaya. Keduanya benar, pilih sesuai template soal.

**Contoh:** `nbElmt t` menghitung banyak simpul di pohon `t`.

### Gaya A: pattern matching (gaya Haskell)

```haskell
nbElmt :: BinTree a -> Int
nbElmt Empty        = 0
nbElmt (Node _ l r) = 1 + nbElmt l + nbElmt r
```

### Gaya B: selektor + predikat (gaya notasi fungsional kuliah)

```haskell
nbElmt :: BinTree a -> Int
nbElmt t
    | isTreeEmpty t = 0
    | otherwise     = 1 + nbElmt (left t) + nbElmt (right t)
```

```haskell
-- ghci> nbElmt t1
-- 6
```

| | Pattern matching | Selektor + predikat |
|---|---|---|
| Kelebihan | singkat, tidak mungkin memanggil `akar Empty` | mirip notasi kuliah, mirip jawaban `TreeFold.hs` |
| Perhatikan | urutan pola penting (dicek dari atas) | predikat **harus** dicek sebelum selektor |
| Kapan dipakai | template tidak menyediakan selektor | template menyediakan `akar`, `left`, `right`, `isTreeEmpty` |

Di sisa catatan ini kebanyakan contoh memakai gaya A karena lebih pendek. Mengubahnya ke gaya B cukup mekanis:

| Gaya A | Gaya B |
|---|---|
| pola `Empty` | guard `isTreeEmpty t` |
| pola `(Node x l r)` | `x` → `akar t`, `l` → `left t`, `r` → `right t` |
| pola `(Node x Empty Empty)` | guard `isOneElmt t` |

---

## 7. Pola rekursi pohon

Hampir semua soal pohon mengikuti template ini:

```haskell
f :: BinTree a -> hasil
f Empty        = ...                          -- (1) BASIS: jawaban untuk pohon kosong
f (Node x l r) = gabung x (f l) (f r)         -- (2) REKURENS: akar + hasil kiri + hasil kanan
```

Jadi setiap kali menulis fungsi pohon, jawab tiga pertanyaan:

1. **Apa jawaban untuk pohon kosong?** Biasanya `0`, `[]`, `False`, `Empty`, atau `1` untuk perkalian.
2. **Kalau aku sudah tahu jawaban untuk subpohon kiri dan kanan, bagaimana menggabungkannya dengan akar?**
3. **Apakah ada kasus khusus**, misalnya daun yang harus diperlakukan berbeda?

Pertanyaan kedua adalah inti rekursi: **percaya** bahwa `f l` dan `f r` sudah benar, lalu pikirkan satu langkah saja. Ini sama seperti menulis rekursi list, hanya saja hasil rekursinya ada dua.

### Jejak eksekusi `nbElmt`

Pada pohon kecil:

```
    2
   / \
  1   3
```

```
nbElmt (Node 2 (daun 1) (daun 3))
= 1 + nbElmt (daun 1) + nbElmt (daun 3)
= 1 + (1 + nbElmt Empty + nbElmt Empty) + (1 + nbElmt Empty + nbElmt Empty)
= 1 + (1 + 0 + 0) + (1 + 0 + 0)
= 3
```

Setiap `Empty` di ujung pohon menjadi basis yang menghentikan rekursi.

---

## 8. Fungsi-fungsi dasar

Semua contoh menggunakan `t1` dan `t2` dari bagian 5.

### `sumTree`: jumlah semua elemen

```haskell
sumTree :: BinTree Int -> Int
sumTree Empty        = 0
sumTree (Node x l r) = x + sumTree l + sumTree r

-- ghci> sumTree t1
-- 23
```

Polanya sama dengan `nbElmt`, hanya `1` diganti `x`.

### `depth`: kedalaman pohon

```haskell
depth :: BinTree a -> Int
depth Empty        = 0
depth (Node _ l r) = 1 + max (depth l) (depth r)

-- ghci> depth t1
-- 3
```

Kedalaman pohon = 1 (untuk akar) + kedalaman subpohon yang **lebih dalam**. Di sini `max`, bukan `+`, karena kita mencari cabang terpanjang, bukan total.

### `nbDaun`: banyak daun

```haskell
nbDaun :: BinTree a -> Int
nbDaun Empty                = 0
nbDaun (Node _ Empty Empty) = 1                       -- daun
nbDaun (Node _ l r)         = nbDaun l + nbDaun r     -- bukan daun: akar tidak dihitung

-- ghci> nbDaun t1
-- 3
```

Ini contoh soal yang butuh **kasus khusus**: daun dihitung `1`, simpul lain tidak dihitung.

> ⚠️ **Urutan pola penting.** Pola `(Node _ Empty Empty)` harus ditulis **sebelum** `(Node _ l r)`. Kalau terbalik, daun ikut cocok dengan `(Node _ l r)` lebih dulu, dan hasilnya selalu `0`.

### `isMember`: apakah sebuah nilai ada di pohon?

```haskell
isMember :: Eq a => a -> BinTree a -> Bool
isMember _ Empty        = False
isMember y (Node x l r) = y == x || isMember y l || isMember y r

-- ghci> isMember 3 t1
-- True
-- ghci> isMember 5 t1
-- False
```

`Eq a =>` artinya "tipe `a` harus bisa dibandingkan dengan `==`". Ini wajib ditulis kalau fungsi polimorfik memakai `==`. Kalau tipenya konkret (`Int -> BinTree Int -> Bool`), tidak perlu.

### `maxTree`: contoh pohon tidak kosong

Soal: nilai terbesar di pohon. **Prasyarat: pohon tidak kosong.**

Godaan pertama adalah menulis `maxTree Empty = 0`. Tapi itu **salah** kalau semua elemen negatif: pohon berisi `-5` saja akan menghasilkan `0`. Pohon kosong memang tidak punya nilai maksimum, jadi basisnya harus **daun**, dan kita perlu membedakan bentuk pohon:

```haskell
maxTree :: BinTree Int -> Int
maxTree (Node x Empty Empty) = x                                 -- daun
maxTree (Node x l Empty)     = max x (maxTree l)                 -- uner kiri
maxTree (Node x Empty r)     = max x (maxTree r)                 -- uner kanan
maxTree (Node x l r)         = maximum [x, maxTree l, maxTree r] -- biner
maxTree Empty                = error "maxTree: pohon kosong"     -- melanggar prasyarat

-- ghci> maxTree t1
-- 7
-- ghci> maxTree t2
-- 9
```

Empat kasus ini sama dengan predikat `isOneElmt`, `isUnerLeft`, `isUnerRight`, `isBiner` dari notasi fungsional. Versi gaya selektor:

```haskell
maxTree t
    | isOneElmt t   = akar t
    | isUnerLeft t  = max (akar t) (maxTree (left t))
    | isUnerRight t = max (akar t) (maxTree (right t))
    | otherwise     = maximum [akar t, maxTree (left t), maxTree (right t)]
```

> 💡 Setiap kali soal bilang **"pohon tidak kosong"**, periksa apakah kamu memanggil rekursi pada subpohon yang mungkin kosong. Kalau iya, pecah menjadi kasus daun / uner kiri / uner kanan / biner.

---

## 9. Traversal: pre-order, in-order, post-order

**Traversal** berarti mengunjungi semua simpul dan mengumpulkannya (biasanya ke dalam list). Ketiganya hanya berbeda di **kapan akar diproses**:

| Traversal | Urutan | Cara ingat |
|---|---|---|
| **Pre-order** | **akar** → kiri → kanan | akar di depan (*pre*) |
| **In-order** | kiri → **akar** → kanan | akar di tengah (*in*) |
| **Post-order** | kiri → kanan → **akar** | akar di belakang (*post*) |

```haskell
preorder :: BinTree a -> [a]
preorder Empty        = []
preorder (Node x l r) = [x] ++ preorder l ++ preorder r

inorder :: BinTree a -> [a]
inorder Empty        = []
inorder (Node x l r) = inorder l ++ [x] ++ inorder r

postorder :: BinTree a -> [a]
postorder Empty        = []
postorder (Node x l r) = postorder l ++ postorder r ++ [x]
```

Ketiga fungsi itu identik, hanya posisi `[x]` yang dipindah. Ini juga yang membedakan `treeFoldPre`, `treeFoldIn`, dan `treeFoldPost` di `TreeFold.hs`.

```haskell
-- ghci> preorder t1
-- [4,2,1,3,6,7]
-- ghci> inorder t1
-- [1,2,3,4,6,7]
-- ghci> postorder t1
-- [1,3,2,7,6,4]

-- ghci> preorder t2
-- [5,8,9,3]
-- ghci> inorder t2
-- [9,8,5,3]
-- ghci> postorder t2
-- [9,8,3,5]
```

> ⚠️ `++` menyambung **dua list**. `x` adalah satu elemen, jadi harus dibungkus jadi `[x]`. Menulis `inorder l ++ x ++ inorder r` akan error tipe. Alternatifnya: `inorder l ++ (x : inorder r)`.

### Jejak `inorder` pada pohon kecil

```
    2
   / \
  1   3

inorder (Node 2 (daun 1) (daun 3))
= inorder (daun 1) ++ [2] ++ inorder (daun 3)
= ([] ++ [1] ++ []) ++ [2] ++ ([] ++ [3] ++ [])
= [1] ++ [2] ++ [3]
= [1,2,3]
```

---

## 10. Fungsi yang menghasilkan pohon

Tidak semua fungsi pohon menghasilkan angka atau list. Ada juga yang **membangun pohon baru**. Polanya tetap sama, hanya bagian "gabung" memakai konstruktor `Node` lagi.

> ⚠️ Di Haskell tidak ada nilai yang bisa diubah. "Mengubah pohon" selalu berarti **membuat pohon baru**, sementara pohon lama tetap utuh.

### `mapTree`: terapkan fungsi ke setiap elemen

Versi pohon dari `map`. Bentuk pohon tetap, hanya isinya yang berubah.

```haskell
mapTree :: (a -> b) -> BinTree a -> BinTree b
mapTree _ Empty        = Empty
mapTree f (Node x l r) = Node (f x) (mapTree f l) (mapTree f r)

-- ghci> mapTree (* 10) (Node 2 (daun 1) (daun 3))
-- Node 20 (Node 10 Empty Empty) (Node 30 Empty Empty)
-- ghci> mapTree even (Node 2 (daun 1) (daun 3))
-- Node True (Node False Empty Empty) (Node False Empty Empty)
```

`mapTree` adalah **higher order function** karena parameter pertamanya fungsi. Kamu bisa mengirim lambda atau section, sama seperti `map`.

### `mirror`: cerminkan pohon

```haskell
mirror :: BinTree a -> BinTree a
mirror Empty        = Empty
mirror (Node x l r) = Node x (mirror r) (mirror l)   -- kiri dan kanan ditukar

-- ghci> mirror (Node 2 (daun 1) (daun 3))
-- Node 2 (Node 3 Empty Empty) (Node 1 Empty Empty)
-- ghci> inorder (mirror t1)
-- [7,6,4,3,2,1]
```

Perhatikan: subpohon **juga** dicerminkan secara rekursif, tidak hanya ditukar posisinya di akar.

---

## 11. Binary Search Tree (BST)

**BST** adalah pohon biner dengan aturan tambahan untuk **setiap** simpul:
- semua elemen di subpohon **kiri** lebih **kecil** dari akar,
- semua elemen di subpohon **kanan** lebih **besar** dari akar.

`t1` adalah BST:

```
        4
       / \
      2   6        kiri dari 4: {2, 1, 3}, semuanya < 4
     / \   \       kanan dari 4: {6, 7},   semuanya > 4
    1   3   7
```

Akibat penting: **in-order traversal dari BST selalu terurut naik**. Lihat `inorder t1 = [1,2,3,4,6,7]`.

### Mencari di BST

Karena sudah terurut, kita tidak perlu memeriksa kedua subpohon. Cukup pilih satu arah:

```haskell
searchBST :: Int -> BinTree Int -> Bool
searchBST _ Empty = False
searchBST y (Node x l r)
    | y == x    = True
    | y < x     = searchBST y l      -- pasti di kiri, kalau ada
    | otherwise = searchBST y r      -- pasti di kanan, kalau ada

-- ghci> searchBST 3 t1
-- True
-- ghci> searchBST 5 t1
-- False
```

Bandingkan dengan `isMember` yang harus memeriksa kiri **dan** kanan.

### Menyisipkan ke BST

```haskell
insertBST :: Int -> BinTree Int -> BinTree Int
insertBST y Empty = daun y                              -- tempat kosong ditemukan
insertBST y (Node x l r)
    | y < x     = Node x (insertBST y l) r              -- sisip ke kiri, kanan tetap
    | y > x     = Node x l (insertBST y r)              -- sisip ke kanan, kiri tetap
    | otherwise = Node x l r                            -- sudah ada, tidak disisipkan lagi

-- ghci> insertBST 5 t1
-- Node 4 (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty)) (Node 6 (Node 5 Empty Empty) (Node 7 Empty Empty))
```

```
        4                    4
       / \                  / \
      2   6      →         2   6
     / \   \              / \ / \
    1   3   7            1  3 5  7
```

Jejaknya: `5 > 4` ke kanan, `5 < 6` ke kiri, ketemu `Empty`, jadi `daun 5` dipasang di situ.

### Membangun BST dari list (pakai `foldl`)

Sisipkan elemen satu per satu dari kiri, mulai dari pohon kosong:

```haskell
buildBST :: [Int] -> BinTree Int
buildBST li = foldl (\t y -> insertBST y t) Empty li

-- ghci> buildBST [4,2,6,1,3,7] == t1
-- True
-- ghci> inorder (buildBST [5,3,8,1,4,9,2])
-- [1,2,3,4,5,8,9]
```

Polanya sama dengan `buatEnkripsi` di `AturanMcBucket.hs`: akumulator (`t`, pohon sejauh ini) dibawa dari kiri ke kanan dan diperbarui oleh setiap elemen. Ingat urutan parameter lambda `foldl`: **akumulator dulu**, lalu elemen.

Bonus: `inorder (buildBST li)` adalah algoritma pengurutan (*tree sort*), dan duplikat ikut terbuang.

### Elemen terkecil di BST

Elemen terkecil selalu ada di simpul **paling kiri**:

```haskell
minBST :: BinTree Int -> Int
minBST (Node x Empty _) = x           -- tidak ada kiri lagi
minBST (Node _ l _)     = minBST l
minBST Empty            = error "minBST: pohon kosong"

-- ghci> minBST t1
-- 1
```

---

## 12. Pohon dan HOF: `mapTree` dan `foldTree`

Kalau kamu perhatikan, `nbElmt`, `sumTree`, `depth`, dan `inorder` punya bentuk yang sama:

```haskell
f Empty        = basis
f (Node x l r) = gabung (f l) x (f r)
```

Yang berbeda hanya `basis` dan cara `gabung`. Jadi pola ini bisa ditulis **sekali** sebagai HOF, lalu bagian yang berbeda dikirim sebagai parameter. Ini versi pohon dari `foldr`:

```haskell
foldTree :: (b -> a -> b -> b) -> b -> BinTree a -> b
foldTree _ z Empty        = z
foldTree f z (Node x l r) = f (foldTree f z l) x (foldTree f z r)
```

Lalu fungsi-fungsi sebelumnya bisa ditulis ulang dengan lambda:

```haskell
nbElmtF :: BinTree a -> Int
nbElmtF t = foldTree (\l _ r -> l + 1 + r) 0 t

depthF :: BinTree a -> Int
depthF t = foldTree (\l _ r -> 1 + max l r) 0 t

inorderF :: BinTree a -> [a]
inorderF t = foldTree (\l x r -> l ++ [x] ++ r) [] t

-- ghci> nbElmtF t1
-- 6
-- ghci> depthF t1
-- 3
-- ghci> inorderF t1
-- [1,2,3,4,6,7]
-- ghci> foldTree (\l x r -> l + x + r) 0 t1
-- 23
```

Cara membaca lambda di dalam `foldTree`: `l` adalah **hasil yang sudah jadi** dari subpohon kiri, `x` adalah akar, dan `r` adalah hasil dari subpohon kanan. Tugasmu hanya menggabungkan ketiganya.

`foldTree` di sini sama dengan `treeFoldIn` di `TreeFold.hs`. Versi `Post` dan `Pre` hanya mengubah urutan argumen lambda.

> 💡 `foldTree` tidak cocok untuk fungsi yang butuh kasus khusus daun (seperti `nbDaun`), karena lambdanya tidak tahu apakah `l` dan `r` berasal dari pohon kosong. Untuk soal seperti itu, tulis rekursi biasa.

---

## 13. Strategi mengerjakan soal pohon

### Langkah-langkah

1. **Gambar pohon contoh dari soal di kertas.** Output `Node ... (Node ...) ...` sulit dibaca kalau tidak digambar.
2. **Tentukan tipe hasil.** Angka? `Bool`? List? Pohon baru? Ini menentukan bentuk basis dan cara menggabungkan.
3. **Tentukan basis.**
   - Pohon boleh kosong → basis `Empty`.
   - Prasyarat "tidak kosong" → basis daun, lalu tangani uner kiri / uner kanan / biner.
4. **Tulis rekurens dengan "percaya pada rekursi".** Anggap `f l` dan `f r` sudah benar. Pikirkan hanya cara menggabungkannya dengan akar.
5. **Cek apakah daun perlu diperlakukan khusus.** Contohnya `nbDaun` dan `listDaun`. Kalau iya, tulis pola daun **sebelum** pola umum.
6. **Tes di ghci** dengan pohon kosong, satu elemen, uner, dan pohon contoh soal.

### Tabel cepat: soal → basis dan gabung

| Soal | Basis (`Empty`) | Gabung untuk `Node x l r` |
|---|---|---|
| banyak elemen | `0` | `1 + f l + f r` |
| jumlah elemen | `0` | `x + f l + f r` |
| hasil kali elemen | `1` | `x * f l * f r` |
| kedalaman | `0` | `1 + max (f l) (f r)` |
| cari elemen | `False` | `y == x \|\| f l \|\| f r` |
| semua memenuhi `p` | `True` | `p x && f l && f r` |
| list elemen (traversal) | `[]` | `f l ++ [x] ++ f r` (atau urutan lain) |
| pohon baru (map/mirror) | `Empty` | `Node (...) (f l) (f r)` |

### Template file gaya praktikum

```haskell
module JumlahDaun where

data BinTree a = Empty | Node a (BinTree a) (BinTree a)
    deriving (Show, Eq)

-- JUMLAH DAUN
-- DEFINISI DAN SPESIFIKASI
nbDaun :: BinTree a -> Int
-- nbDaun t menghasilkan banyaknya daun pada pohon t.
-- Daun adalah simpul yang tidak memiliki subpohon kiri maupun kanan.
-- Jika t kosong, hasilnya 0.

-- REALISASI
nbDaun Empty                = 0
nbDaun (Node _ Empty Empty) = 1
nbDaun (Node _ l r)         = nbDaun l + nbDaun r

-- APLIKASI
-- > nbDaun (Node 4 (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty)) (Node 6 Empty (Node 7 Empty Empty)))
-- 3
-- > nbDaun Empty
-- 0
```

---

## 14. Kesalahan umum

| Kesalahan | Penyebab | Perbaikan |
|---|---|---|
| `Non-exhaustive patterns` saat dijalankan | lupa menulis kasus `Empty` (atau kasus lain) | pastikan setiap konstruktor punya pola |
| `akar: pohon kosong` / crash | memanggil selektor pada `Empty` | cek `isTreeEmpty` di guard **sebelum** memakai `akar`, `left`, `right` |
| `nbDaun` selalu `0` | pola `(Node _ l r)` ditulis sebelum `(Node _ Empty Empty)` | tulis pola yang lebih spesifik **di atas** |
| `depth` hasilnya sama dengan `nbElmt` | memakai `depth l + depth r` | pakai `1 + max (depth l) (depth r)` |
| `maxTree` salah untuk bilangan negatif | basis `maxTree Empty = 0` | basis daun + kasus uner, lihat bagian 8 |
| error tipe di traversal | `inorder l ++ x ++ inorder r` | bungkus jadi `[x]` |
| `Node 2 Node 1 Empty Empty Empty` error | subpohon tidak dikurung | `Node 2 (Node 1 Empty Empty) Empty` |
| `Expecting one more argument to 'BinTree'` | menulis tipe `BinTree` saja | `BinTree Int` atau `BinTree a` |
| `No instance for (Show (BinTree Int))` | lupa `deriving (Show)` | tambahkan `deriving (Show, Eq)` |
| `No instance for (Eq a)` | fungsi polimorfik memakai `==` | tambahkan `Eq a =>` di signature |
| `isBST` salah menerima pohon yang bukan BST | hanya membandingkan akar dengan **anak langsung** | lihat latihan 8 |
| `mirror` hanya menukar level pertama | `Node x r l` tanpa rekursi | `Node x (mirror r) (mirror l)` |

Contoh jebakan `isBST` (kasus ke-11 di tabel):

```
      5
     / \
    3   9          3 < 5 ✔, 9 > 5 ✔, 1 < 3 ✔, 8 > 3 ✔
   / \             tapi 8 ada di subpohon KIRI dari 5, padahal 8 > 5
  1   8            → BUKAN BST
```

Kalau hanya mengecek anak langsung, pohon ini lolos sebagai BST. Aturan BST berlaku untuk **seluruh** subpohon, bukan hanya anaknya.

---

## 15. Cheat sheet

```haskell
-- DEFINISI
data BinTree a = Empty | Node a (BinTree a) (BinTree a)
    deriving (Show, Eq)

-- MEMBUAT POHON
daun x = Node x Empty Empty
Node 2 (daun 1) (daun 3)               -- subpohon WAJIB dikurung

-- POLA PATTERN MATCHING
f Empty                = ...           -- pohon kosong
f (Node x Empty Empty) = ...           -- daun        (tulis sebelum pola umum)
f (Node x l Empty)     = ...           -- uner kiri
f (Node x Empty r)     = ...           -- uner kanan
f (Node x l r)         = ...           -- umum

-- GAYA SELEKTOR
f t
    | isTreeEmpty t = ...
    | otherwise     = ... (akar t) ... f (left t) ... f (right t) ...

-- TRAVERSAL
preorder  (Node x l r) = [x] ++ preorder l ++ preorder r
inorder   (Node x l r) = inorder l ++ [x] ++ inorder r
postorder (Node x l r) = postorder l ++ postorder r ++ [x]

-- BST: kiri < akar < kanan, inorder selalu terurut naik
searchBST y (Node x l r) | y == x = True | y < x = searchBST y l | otherwise = searchBST y r

-- HOF
mapTree  f (Node x l r) = Node (f x) (mapTree f l) (mapTree f r)
foldTree f z (Node x l r) = f (foldTree f z l) x (foldTree f z r)
```

**Cara cepat memilih bentuk jawaban:**

- Hasilnya **angka** dari seluruh pohon → basis `0` (atau `1` untuk kali), gabung dengan `+` / `*` / `max`
- Hasilnya **Bool** → basis `False` (ada?) atau `True` (semua?), gabung dengan `||` / `&&`
- Hasilnya **list** → basis `[]`, gabung dengan `++`
- Hasilnya **pohon** → basis `Empty`, gabung dengan `Node`
- Soal menyebut **daun** → tambahkan pola `(Node x Empty Empty)`
- Soal bilang **pohon tidak kosong** → basis daun + kasus uner kiri / uner kanan
- Soal menyebut **BST** → cukup telusuri satu sisi (kiri atau kanan), jangan dua-duanya

---

## 16. Latihan soal

Gunakan definisi `BinTree`, `daun`, `t1`, dan `t2` dari bagian 4 dan 5. Untuk mengetes, salin definisi itu ke file `.hs` bersama jawabanmu lalu buka dengan `ghci`.

```
t1:     4               t2:     5
       / \                     / \
      2   6                   8   3
     / \   \                 /
    1   3   7               9
```

Coba kerjakan sendiri dulu sebelum membuka jawaban.

### Level 1: pola dasar

**1. `productTree :: BinTree Int -> Int`**
Hasil kali semua elemen. Pohon kosong menghasilkan `1`.
```
productTree t1     => 1008
productTree Empty  => 1
```

**2. `hitungJika :: (a -> Bool) -> BinTree a -> Int`**
Banyaknya elemen yang memenuhi predikat `p`. (HOF!)
```
hitungJika even t1     => 3      -- 4, 2, 6
hitungJika (> 3) t1    => 3      -- 4, 6, 7
```

**3. `listDaun :: BinTree a -> [a]`**
Daftar semua daun, dari kiri ke kanan.
```
listDaun t1  => [1,3,7]
listDaun t2  => [9,3]
```

**4. `hapusDaun :: BinTree a -> BinTree a`**
Pohon baru dengan semua daun dibuang.
```
hapusDaun t1  => Node 4 (Node 2 Empty Empty) (Node 6 Empty Empty)
```

<details>
<summary>Jawaban level 1</summary>

```haskell
productTree :: BinTree Int -> Int
productTree Empty        = 1
productTree (Node x l r) = x * productTree l * productTree r

hitungJika :: (a -> Bool) -> BinTree a -> Int
hitungJika _ Empty        = 0
hitungJika p (Node x l r) = (if p x then 1 else 0) + hitungJika p l + hitungJika p r

listDaun :: BinTree a -> [a]
listDaun Empty                = []
listDaun (Node x Empty Empty) = [x]
listDaun (Node _ l r)         = listDaun l ++ listDaun r

hapusDaun :: BinTree a -> BinTree a
hapusDaun Empty                = Empty
hapusDaun (Node _ Empty Empty) = Empty
hapusDaun (Node x l r)         = Node x (hapusDaun l) (hapusDaun r)
```

Catatan:
- `productTree`: basis `1`, bukan `0`, karena `1` adalah elemen netral perkalian. Sama seperti `amplifikasiQuantum`.
- `listDaun` dan `hapusDaun`: pola daun **harus** di atas pola umum.

</details>

### Level 2: menengah

**5. `elmtLevel :: Int -> BinTree a -> [a]`**
Semua elemen di level `k` (akar = level 1), dari kiri ke kanan.
```
elmtLevel 1 t1  => [4]
elmtLevel 2 t1  => [2,6]
elmtLevel 3 t1  => [1,3,7]
elmtLevel 4 t1  => []
```
*Petunjuk:* level `k` dari pohon = level `k-1` dari subpohon kiri dan kanan.

**6. `samaBentuk :: BinTree a -> BinTree b -> Bool`**
`True` jika kedua pohon punya bentuk yang sama persis (isinya boleh berbeda).
```
samaBentuk t1 (mapTree show t1)  => True
samaBentuk t1 t2                 => False
```
*Petunjuk:* lakukan pattern matching pada **dua** pohon sekaligus.

**7. `jumlahJalurMaks :: BinTree Int -> Int`**
Jumlah terbesar dari jalur akar sampai daun. **Prasyarat: pohon tidak kosong.**
```
jumlahJalurMaks t1  => 17      -- 4 + 6 + 7
jumlahJalurMaks t2  => 22      -- 5 + 8 + 9
```
*Petunjuk:* lihat `maxTree`. Jalur harus berakhir di daun, jadi untuk pohon uner kamu **tidak boleh** memilih sisi yang kosong.

**8. `isBST :: BinTree Int -> Bool`**
`True` jika pohon adalah BST (tanpa elemen kembar). Hati-hati dengan jebakan di bagian 14.
```
isBST t1  => True
isBST t2  => False
isBST (Node 5 (Node 3 (daun 1) (daun 8)) (daun 9))  => False
```
*Petunjuk:* manfaatkan sifat in-order BST.

<details>
<summary>Jawaban level 2</summary>

```haskell
elmtLevel :: Int -> BinTree a -> [a]
elmtLevel _ Empty        = []
elmtLevel 1 (Node x _ _) = [x]
elmtLevel k (Node _ l r) = elmtLevel (k - 1) l ++ elmtLevel (k - 1) r

samaBentuk :: BinTree a -> BinTree b -> Bool
samaBentuk Empty Empty                   = True
samaBentuk (Node _ l1 r1) (Node _ l2 r2) = samaBentuk l1 l2 && samaBentuk r1 r2
samaBentuk _ _                           = False     -- satu kosong, satu tidak

jumlahJalurMaks :: BinTree Int -> Int
jumlahJalurMaks (Node x Empty Empty) = x
jumlahJalurMaks (Node x l Empty)     = x + jumlahJalurMaks l
jumlahJalurMaks (Node x Empty r)     = x + jumlahJalurMaks r
jumlahJalurMaks (Node x l r)         = x + max (jumlahJalurMaks l) (jumlahJalurMaks r)
jumlahJalurMaks Empty                = error "jumlahJalurMaks: pohon kosong"

isBST :: BinTree Int -> Bool
isBST t = naikKetat (inorder t)
    where
        naikKetat (a:b:sisa) = a < b && naikKetat (b:sisa)
        naikKetat _          = True    -- list kosong atau satu elemen
```

Catatan:
- `samaBentuk` memakai dua variabel tipe (`a` dan `b`) supaya pohon `Int` bisa dibandingkan dengan pohon `String`.
- `isBST`: BST ⇔ in-order-nya naik ketat. Pola `(a:b:sisa)` mengambil dua elemen pertama sekaligus.

</details>

### Level 3: lanjut

**9. `deleteBST :: Int -> BinTree Int -> BinTree Int`**
Hapus sebuah nilai dari BST, hasilnya harus tetap BST. Kalau nilai tidak ada, pohon tidak berubah.
```
inorder (deleteBST 4 t1)  => [1,2,3,6,7]
deleteBST 4 t1            => Node 6 (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty)) (Node 7 Empty Empty)
inorder (deleteBST 2 t1)  => [1,3,4,6,7]
```
*Petunjuk:* kalau simpul yang dihapus punya dua anak, gantikan akarnya dengan elemen **terkecil** dari subpohon kanan (`minBST`), lalu hapus elemen itu dari subpohon kanan.

**10. `bangunSeimbang :: [a] -> BinTree a`**
Bangun pohon dari list yang sudah terurut sehingga pohonnya seimbang: elemen tengah jadi akar, separuh kiri jadi subpohon kiri, separuh kanan jadi subpohon kanan.
```
bangunSeimbang [1..7]
=> Node 4 (Node 2 (Node 1 Empty Empty) (Node 3 Empty Empty)) (Node 6 (Node 5 Empty Empty) (Node 7 Empty Empty))
depth (bangunSeimbang [1..7])  => 3
```
*Petunjuk:* pakai `take`, `drop`, dan `!!` (ambil elemen ke-n).

**11. `levelOrder :: BinTree a -> [a]`**
Elemen pohon dibaca level demi level, dari kiri ke kanan (*breadth-first*).
```
levelOrder t1  => [4,2,6,1,3,7]
levelOrder t2  => [5,8,3,9]
```
*Petunjuk:* bisa digabung dari `elmtLevel` dan `depth`, atau pakai antrean (list of pohon).

**12. Tulis ulang dengan `foldTree`** (definisi di bagian 12): `sumTree`, `preorder`, dan `mirror`.

<details>
<summary>Jawaban level 3</summary>

```haskell
deleteBST :: Int -> BinTree Int -> BinTree Int
deleteBST _ Empty = Empty
deleteBST y (Node x l r)
    | y < x     = Node x (deleteBST y l) r
    | y > x     = Node x l (deleteBST y r)
    | otherwise = hapusAkar l r              -- y == x, simpul ini yang dihapus
    where
        hapusAkar Empty r' = r'              -- tidak punya kiri: naikkan kanan
        hapusAkar l' Empty = l'              -- tidak punya kanan: naikkan kiri
        hapusAkar l' r'    = Node (minBST r') l' (deleteBST (minBST r') r')

bangunSeimbang :: [a] -> BinTree a
bangunSeimbang [] = Empty
bangunSeimbang li = Node tengah (bangunSeimbang kiri) (bangunSeimbang kanan)
    where
        n      = length li `div` 2
        kiri   = take n li
        tengah = li !! n
        kanan  = drop (n + 1) li

-- Cara 1: pakai elmtLevel untuk setiap level
levelOrder :: BinTree a -> [a]
levelOrder t = concat (map (\k -> elmtLevel k t) [1 .. depth t])

-- Cara 2: antrean
levelOrder' :: BinTree a -> [a]
levelOrder' t = proses [t]
    where
        proses []                  = []
        proses (Empty : sisa)      = proses sisa
        proses (Node x l r : sisa) = x : proses (sisa ++ [l, r])   -- anak masuk antrean belakang

sumTreeF :: BinTree Int -> Int
sumTreeF t = foldTree (\l x r -> l + x + r) 0 t

preorderF :: BinTree a -> [a]
preorderF t = foldTree (\l x r -> [x] ++ l ++ r) [] t

mirrorF :: BinTree a -> BinTree a
mirrorF t = foldTree (\l x r -> Node x r l) Empty t
```

Catatan:
- `deleteBST`: di kasus dua anak, `minBST r'` adalah pengganti yang aman karena ia lebih besar dari semua elemen kiri dan lebih kecil dari sisa elemen kanan.
- `levelOrder'`: `proses` memegang antrean pohon yang belum dikunjungi. Simpul diambil dari depan dan anaknya ditaruh di belakang, sehingga simpul level atas selalu keluar lebih dulu.
- `mirrorF`: `l` dan `r` di lambda **sudah** berupa subpohon yang dicerminkan, jadi cukup ditukar posisinya.

</details>

---

> 💡 **Tips terakhir sebelum praktikum:** kalau buntu, tulis dulu kasus `Empty` dan kasus daun, lalu coba fungsi itu di ghci pada pohon satu elemen dan dua elemen. Hampir semua bug pohon kelihatan di pohon kecil.
