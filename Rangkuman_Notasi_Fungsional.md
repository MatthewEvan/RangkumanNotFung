# Rangkuman Kuis Notasi Fungsional — IF2110 Algoritma & Pemrograman 2

> Cakupan: Notasi Fungsional · Ekspresi Dasar · Ekspresi Kondisional · Type Bentukan · Ekspresi Rekursif · List
> Sumber: slide kuliah IF2110 Sem. I 2026/2027 (diturunkan dari *Diktat Pemrograman Fungsional*, S. A. Rukmono)

---

## Daftar Isi

0. [Peta Materi & Cara Belajar](#0-peta-materi--cara-belajar)
1. [Paradigma & Notasi Fungsional](#1-paradigma--notasi-fungsional)
2. [Ekspresi Dasar](#2-ekspresi-dasar)
3. [Ekspresi Kondisional](#3-ekspresi-kondisional)
4. [Type Bentukan](#4-type-bentukan)
5. [Ekspresi Rekursif](#5-ekspresi-rekursif)
6. [List](#6-list)
7. [Cheat Sheet Akhir](#7-cheat-sheet-akhir)

---

## 0. Peta Materi & Cara Belajar

Materinya bertumpuk. Setiap bab memakai ulang yang sebelumnya:

```
Notasi (4 bagian)
   └─► Ekspresi Dasar (operator, prasyarat, let, evaluasi)
          └─► Ekspresi Kondisional (depend on, lengkap & disjoint, and then)
                 └─► Type Bentukan (produk, invarian, alternatif)
                        └─► Ekspresi Rekursif (basis + rekurens menuju basis)
                               └─► List (type rekursif: Nil | Cons)
```

**Bentuk soal kuis biasanya ada lima macam** (sesuai latihan di slide):

| Jenis | Yang diminta | Kunci menjawab |
|---|---|---|
| **Membaca** | Diberi realisasi, tuliskan definisi + spesifikasi + nama | Spesifikasi menyatakan **arti**, bukan mengulang rumus |
| **Menulis** | Tulis 4 bagian lengkap | Definisi → Spesifikasi (prasyarat, basis) → Realisasi → Aplikasi (nilai batas) |
| **Menelusuri** | Trace evaluasi | Hanya dua langkah: **ekspansi** & **reduksi** |
| **Memperbaiki** | Temukan bug | Cari nilai batas, cek lengkap/disjoint, cek invarian, cek konvergensi |
| **Merancang** | Pilih & beri alasan | Tidak ada jawaban tunggal, yang dinilai alasannya |

---

## 1. Paradigma & Notasi Fungsional

### 1.1 Konsep inti

**Paradigma** adalah sudut pandang terhadap persoalan. Ia memusatkan perhatian pada beberapa atribut dan mengabaikan yang lain. Tidak ada paradigma yang cocok untuk semua persoalan.

| Imperatif / Prosedural | Fungsional |
|---|---|
| Ada **state** dan **waktu** | Hanya **ekspresi** dan **nilainya** |
| Nilai sebuah nama bisa berubah | Nama **terikat sekali saja** |
| Instruksi dijalankan berurutan | **Tidak ada urutan instruksi** |
| Arti bergantung pada riwayat | Arti bergantung pada ekspresi |
| Memori dipikirkan pemrogram | Memori bukan urusan pemrogram |

**Yang tidak ada di paradigma fungsional:** variabel yang berubah nilai, urutan instruksi (sebelum/sesudah), alokasi memori, dan pemisahan tegas antara data dan program.

**Yang tersisa hanya dua:**
- **Type**: himpunan nilai beserta operasi yang terdefinisi atasnya.
- **Fungsi**: pemetaan dari satu type ke type lain.

**Transparansi referensial** adalah sifat kunci. Sebuah ekspresi boleh diganti oleh nilainya di mana pun dan kapan pun ia muncul (`3+4` selalu boleh diganti `7`). Akibatnya penalaran terhadap program menjadi jauh lebih sederhana.

Satu persoalan bisa ditulis dengan dua cara:
```
{ Notasi algoritmik: BAGAIMANA menghitung (urutan langkah) }
s ← 0
i traversal [1..n]: s ← s + i
→ s

{ Notasi fungsional: APA yang dihitung }
totalN (n) : n * (n + 1) div 2
```

### 1.2 Empat bagian teks fungsi (urutannya selalu sama)

| # | Bagian | Pertanyaan yang dijawab |
|---|---|---|
| 1 | **Definisi** | Memetakan apa ke apa? (nama, domain, range) |
| 2 | **Spesifikasi** | Apa artinya? (makna + prasyarat) |
| 3 | **Realisasi** | Bagaimana menghitungnya? (ekspresinya) |
| 4 | **Aplikasi** | Bagaimana dipakai? (contoh + hasil) |

Definisi dan spesifikasi ditulis berdampingan. Definisi menyatakan **bentuk**, spesifikasi menyatakan **makna**.

**Abstraksi:** pemakai hanya membaca definisi + spesifikasi, sedangkan penulis bertanggung jawab atas realisasi. Karena itu **satu spesifikasi boleh punya beberapa realisasi yang sah**.

### 1.3 Acuan notasi

```
namaFungsi : A → B              { definisi, 1 parameter }
namaFungsi : A, B → C           { definisi, 2 parameter (dipisah koma) }
namaKonstanta : → A             { konstanta, tanpa parameter }
namaFungsi (x) : e              { realisasi }
⇒ namaFungsi (3)                { aplikasi }
{ ... }                         { komentar / spesifikasi }
```

- Tanda **`:`** dipakai untuk **definisi**.
- Tanda **`=`** dipakai untuk **perbandingan**.
- Keduanya tidak pernah bermakna ganda.
- `x` pada `square (x)` disebut **parameter**; `5` pada `square (5)` disebut **argumen**.

### 1.4 Konvensi penamaan (camelCase / "punukUnta")

| Jenis | Aturan | Contoh |
|---|---|---|
| Fungsi | camelCase | `square`, `divMod` |
| Type | Huruf besar di depan | `Integer`, `Point` |
| **Predikat** (hasil Boolean) | awalan **`is`**, tanpa tanda tanya | `isEmpty`, `isOrigin` |
| **Konstruktor** | awalan **`make`** | `makePoint` |
| **Konversi** | awalan **`to`** | `toReal`, `toSeconds` |
| **Selektor** | sama dengan nama komponen | `x`, `numerator` |
| Nama antara | pendek, lokal | `dx`, `dy` |

> ⚠️ Bahan lama menulis `IsOrigin?`, `Absis`, `MakePoint`. Yang berlaku **sekarang** adalah aturan di tabel atas.

### 1.5 Alur kerja untuk setiap soal

1. **Definisi**: nama, domain, range. Sudah tepatkah domainnya?
2. **Spesifikasi**: makna, prasyarat, kasus batas.
3. **Realisasi**: baru di tahap ini ekspresinya ditulis.
4. **Uji**: tulis hasil yang diharapkan lebih dulu, baru dijalankan.

### 1.6 Contoh kasus

#### Kasus 1: Pangkat dua & pangkat tiga (satu spesifikasi, dua realisasi)

```
JUDUL Pangkat Dua dan Pangkat Tiga

DEFINISI DAN SPESIFIKASI
  square : Integer → Integer
  { square (x) adalah nilai x dipangkatkan dua }

  cube : Integer → Integer
  { cube (x) adalah nilai x dipangkatkan tiga }

REALISASI
  square (x) : x * x

  cube (x) : x * square (x)      { v1: memakai fungsi antara }
  cube (x) : x * x * x           { v2: langsung }

APLIKASI
  ⇒ square (-3)     9
  ⇒ cube (2)        8
  ⇒ cube (-2)       -8
```
Kedua realisasi `cube` benar. Memilih salah satunya adalah **keputusan rancangan**.

#### Kasus 2: Celcius → Fahrenheit (latihan slide)

```
JUDUL Konversi Suhu

DEFINISI DAN SPESIFIKASI
  toFahrenheit : Real → Real
  { toFahrenheit (c) adalah suhu dalam derajat Fahrenheit
    yang setara dengan suhu c derajat Celcius }

REALISASI
  toFahrenheit (c) : 9.0 / 5.0 * c + 32.0

APLIKASI
  ⇒ toFahrenheit (0.0)      32.0
  ⇒ toFahrenheit (100.0)    212.0
  ⇒ toFahrenheit (-40.0)    -40.0
```
Catatan jawaban:
- Domain dipilih **Real**, bukan Integer, karena suhu bisa pecahan.
- Nama diberi awalan `to` karena fungsinya melakukan konversi.
- Spesifikasi yang baik menyatakan **makna** ("suhu setara dalam Fahrenheit"). Spesifikasi yang buruk hanya mengulang rumus ("9/5 kali c tambah 32").

#### Kasus 3: Latihan membaca (tentukan definisi & spesifikasi)

```
f (a, b) : (a + b) div 2
g (x)    : x - (x div 10) * 10
h (a, b) : a mod b = 0
```
Jawaban:
```
average2 : Integer, Integer → Integer
{ average2 (a, b) adalah rata-rata a dan b, dibulatkan ke bawah }

lastDigit : Integer ≥ 0 → Integer [0..9]
{ lastDigit (x) adalah digit satuan x (digit paling kanan) }

isDivisible : Integer, Integer → Boolean
{ isDivisible (a, b) true bila a habis dibagi b. Prasyarat: b ≠ 0 }
```
Perhatikan bahwa `h` menghasilkan Boolean, jadi namanya **wajib** diawali `is`. Fungsi ini juga perlu prasyarat karena `mod` dengan b = 0 tidak terdefinisi.

---

## 2. Ekspresi Dasar

### 2.1 Ekspresi

Ekspresi adalah teks yang **bila dievaluasi menghasilkan satu nilai**. Ia tersusun dari nama, konstanta, operator, aplikasi fungsi, dan kurung. Realisasi setiap fungsi adalah sebuah ekspresi.

### 2.2 Operator

**Aritmatika** (hasil bertype sama dengan operan):

| Operator | Berlaku untuk |
|---|---|
| `+ - *` | Integer atau Real |
| `-x` (uner) | negasi |
| `/` | **hanya Real** (pembagian riil) |
| `div`, `mod` | **hanya Integer** (hasil bagi & sisa bagi) |

Aturan type:
- Kedua operan harus bertype sama: Integer dengan Integer, Real dengan Real.
- **Tidak ada konversi otomatis** Integer ↔ Real. Konversi ditulis sendiri dengan `asReal (x)`.
- Kesalahan type ketahuan saat menulis, bukan saat program berjalan.

**Relasional** (hasil selalu Boolean): `= ≠` untuk dua nilai bertype sama, `< > ≤ ≥` untuk Integer, Real, Character.

**Boolean**: `and`, `or`, `not`.
- **Operan `not` selalu dalam kurung**: `not (x < y)`, supaya jangkauan negasinya jelas.

### 2.3 Fungsi dasar yang tersedia (hanya ini, sengaja sedikit)

```
abs    : Integer → Integer     { nilai mutlak }
sqrt   : Real → Real           { akar kuadrat. Prasyarat: x ≥ 0.0 }
asReal : Integer → Real        { satu-satunya "jembatan" Integer → Real }
pi     : → Real                { konstanta }
```
`max2` dan fungsi lain harus ditulis sendiri.

### 2.4 Prasyarat

Prasyarat adalah **bagian dari kontrak** fungsi, bukan catatan tambahan. Prasyarat wajib ditulis kalau ada bagian domain di mana fungsinya tidak terdefinisi:
- `a div b` dan `a mod b` tidak terdefinisi bila `b = 0`
- `a / b` tidak terdefinisi bila `b = 0.0`
- `sqrt (x)` tidak terdefinisi bila `x < 0.0`

```
average2 : Integer, Integer → Integer
{ rata-rata a dan b, dibulatkan }
average2 (a, b) : (a + b) div 2          { pembagi konstanta → aman, tanpa prasyarat }

averagePart : Integer, Integer → Integer
{ rata-rata n nilai yang jumlahnya `jumlah`. Prasyarat: n > 0 }
averagePart (jumlah, n) : jumlah div n   { pembagi parameter → PERLU prasyarat }
```
Mengapa prasyaratnya `n > 0` dan bukan `n ≠ 0`? Karena "banyaknya nilai" yang negatif juga tidak bermakna. Prasyarat harus sesuai **arti**, bukan sekadar menghindari error.

### 2.5 Nama antara (`let`) vs fungsi antara

```
distance (x1, y1, x2, y2) :
  let
    dx : x1 - x2
    dy : y1 - y2
  in
    sqrt (dx * dx + dy * dy)
```

| Nama antara (`let`) | Fungsi antara |
|---|---|
| Hidup hanya dalam satu realisasi | Dapat dipakai di mana saja |
| Tidak perlu definisi | Perlu definisi + spesifikasi |
| Tidak menambah perbendaharaan | Menambah perbendaharaan |
| Type disimpulkan sendiri | Konsepnya berdiri sendiri |
| Contoh: `dx`, `dy` | Contoh: `sqrDif` |

### 2.6 Penelusuran evaluasi: hanya dua langkah

- **Ekspansi**: aplikasi fungsi diganti dengan realisasinya.
- **Reduksi**: operator atau fungsi dasar dihitung.
- Evaluasi **berhenti** ketika tidak ada lagi yang bisa disederhanakan.

Karena transparansi referensial, **urutan evaluasi tidak mengubah hasil**, dan bagian yang saling bebas boleh dihitung bersamaan. Notasi ini memang tidak menetapkan urutan evaluasi.

### 2.7 Padanan Haskell

| Notasi | Haskell |
|---|---|
| `A, B → C` | `A -> B -> C` |
| `f (x, y) : e` | `f x y = e` |
| `= ≠ ≤` | `== /= <=` |
| `{ komentar }` | `-- komentar` |
| Real / Boolean / Character | `Double` / `Bool` / `Char` |

### 2.8 Contoh kasus

#### Kasus 1: Jarak dua titik + penelusuran lengkap

```
JUDUL Jarak Dua Titik

DEFINISI DAN SPESIFIKASI
  sqrDif : Real, Real → Real
  { sqrDif (a, b) adalah kuadrat dari selisih a dan b }

  distance : Real, Real, Real, Real → Real
  { distance (x1, y1, x2, y2) adalah jarak titik (x1,y1) ke (x2,y2) }

REALISASI
  sqrDif (a, b) : (a - b) * (a - b)
  distance (x1, y1, x2, y2) : sqrt (sqrDif (x1, x2) + sqrDif (y1, y2))

APLIKASI
  ⇒ distance (1.0, 3.0, 5.0, 6.0)   5.0
```
Penelusuran:
```
⇒ distance (1.0, 3.0, 5.0, 6.0)
→ sqrt (sqrDif (1.0, 5.0) + sqrDif (3.0, 6.0))        { ekspansi distance }
→ sqrt ((1.0-5.0)*(1.0-5.0) + sqrDif (3.0, 6.0))      { ekspansi sqrDif }
→ sqrt ((-4.0)*(-4.0) + sqrDif (3.0, 6.0))            { reduksi - }
→ sqrt (16.0 + sqrDif (3.0, 6.0))                     { reduksi * }
→ sqrt (16.0 + (3.0-6.0)*(3.0-6.0))                   { ekspansi sqrDif }
→ sqrt (16.0 + (-3.0)*(-3.0))                         { reduksi - }
→ sqrt (16.0 + 9.0)                                   { reduksi * }
→ sqrt (25.0)                                         { reduksi + }
→ 5.0                                                 { reduksi sqrt }
```
Pertanyaan yang sering muncul: *mengapa `sqrt` di sini tidak perlu prasyarat?* Karena jumlah dua kuadrat selalu ≥ 0. Lalu kenapa `square` dari Bab 2 tidak bisa dipakai? Karena domain `square` adalah Integer, sedangkan di sini yang dibutuhkan Real.

#### Kasus 2: Tahun kabisat + penelusuran `isLeapYear (1900)`

```
JUDUL Tahun Kabisat

DEFINISI DAN SPESIFIKASI
  isLeapYear : Integer > 0 → Boolean
  { isLeapYear (y) true bila y adalah tahun kabisat menurut kalender
    Gregorian: habis dibagi 4 tetapi tidak habis dibagi 100,
    atau habis dibagi 400 }

REALISASI
  isLeapYear (y) : ((y mod 4 = 0) and (y mod 100 ≠ 0)) or (y mod 400 = 0)

APLIKASI
  ⇒ isLeapYear (2024)   true     { habis /4, tidak habis /100 }
  ⇒ isLeapYear (2023)   false    { tidak habis /4 }
  ⇒ isLeapYear (1900)   false    { habis /100 tapi tidak /400 }
  ⇒ isLeapYear (2000)   true     { habis /400 }
```
Pemilihan contoh aplikasinya disengaja: **satu contoh untuk setiap kemungkinan**.

Penelusuran:
```
⇒ isLeapYear (1900)
→ ((1900 mod 4 = 0) and (1900 mod 100 ≠ 0)) or (1900 mod 400 = 0)   { ekspansi }
→ ((0 = 0) and (1900 mod 100 ≠ 0)) or (1900 mod 400 = 0)            { reduksi mod }
→ (true and (1900 mod 100 ≠ 0)) or (1900 mod 400 = 0)               { reduksi = }
→ (true and (0 ≠ 0)) or (1900 mod 400 = 0)                          { reduksi mod }
→ (true and false) or (1900 mod 400 = 0)                            { reduksi ≠ }
→ false or (1900 mod 400 = 0)                                       { reduksi and }
→ false or (300 = 0)                                                { reduksi mod }
→ false or false                                                    { reduksi = }
→ false                                                             { reduksi or }
```
Pengamatan: begitu `(0 ≠ 0)` tereduksi menjadi `false`, bagian kiri `and` sudah pasti `false`. Tetapi karena `false or X` bergantung pada X, hasil akhir baru pasti setelah `1900 mod 400 = 0` dievaluasi.

#### Kasus 3: Durasi jam–menit (latihan menulis)

```
JUDUL Konversi Durasi

DEFINISI DAN SPESIFIKASI
  toMinutes : Integer ≥ 0, Integer [0..59] → Integer ≥ 0
  { toMinutes (h, m) adalah banyaknya menit dalam durasi h jam m menit }

  fullHours : Integer ≥ 0 → Integer ≥ 0
  { fullHours (m) adalah banyaknya jam penuh dalam m menit }

REALISASI
  toMinutes (h, m) : h * 60 + m
  fullHours (m) : m div 60

APLIKASI
  ⇒ toMinutes (2, 30)   150
  ⇒ toMinutes (0, 0)    0
  ⇒ fullHours (150)     2
  ⇒ fullHours (59)      0      { nilai batas: belum satu jam }
  ⇒ fullHours (60)      1      { nilai batas: tepat satu jam }
```

#### Kasus 4: Kuadrat sempurna (memikirkan prasyarat `sqrt`)

```
DEFINISI DAN SPESIFIKASI
  isPerfectSquare : Integer ≥ 0 → Boolean
  { isPerfectSquare (n) true bila ada bilangan bulat k dengan k * k = n }

REALISASI
  isPerfectSquare (n) :
    let
      k : floor (sqrt (asReal (n)))     { lihat catatan }
    in
      k * k = n
```
Catatan: fungsi pembulatan (`floor`) **tidak** termasuk daftar fungsi dasar di slide. Kalau di kuis tidak disediakan, tulis sebagai fungsi antara lengkap dengan definisi dan spesifikasinya. Domain `Integer ≥ 0` sekaligus memenuhi prasyarat `sqrt`.

#### Kasus 5: Latihan membaca (Bab 3)

```
p (a, b, c) : (a + b + c) div 3
q (x)       : (x mod 2 = 0) and (x mod 3 = 0)
r (a, b)    : let s : a + b in s*s - 4.0*a*b
t (n)       : abs (n) = n
```
Jawaban:
```
average3 : Integer, Integer, Integer → Integer
{ rata-rata tiga bilangan bulat, dibulatkan ke bawah }

isMultipleOf6 : Integer → Boolean
{ true bila x habis dibagi 2 dan 3, artinya kelipatan 6 }

sqrDiffReal : Real, Real → Real
{ (a + b)² - 4ab = (a - b)², kuadrat selisih a dan b }
  { perhatikan 4.0 → a, b bertype Real }

isNonNegative : Integer → Boolean
{ true bila n ≥ 0 }
```
Trik membaca: **sederhanakan secara aljabar** dulu. Bentuk `(a+b)² − 4ab` sebenarnya sama dengan `(a−b)²`.

---

## 3. Ekspresi Kondisional

### 3.1 Analisis kasus

Ekspresi kondisional dipakai ketika nilai bergantung pada kasus. Domainnya **dipilah** menjadi beberapa bagian, lalu setiap bagian diberi ekspresinya sendiri.

```
namaFungsi (x) :
  depend on ⟨deskripsi domain⟩
    ⟨kondisi-1⟩ : ⟨ekspresi-1⟩
    ⟨kondisi-2⟩ : ⟨ekspresi-2⟩
    ⟨kondisi-3⟩ : ⟨ekspresi-3⟩
```
Nilai ekspresi kondisional adalah nilai ekspresi yang kondisinya benar. **Tepat satu kondisi harus benar.**

### 3.2 Dua syarat wajib: LENGKAP & DISJOINT

| Lengkap | Disjoint |
|---|---|
| Setiap nilai domain memenuhi **≥ 1** kondisi | Tidak ada nilai yang memenuhi **> 1** kondisi |
| `k1 or k2 or … ≡ true` | `not (ki and kj)` untuk setiap i ≠ j |
| Kalau dilanggar, ada masukan tanpa hasil (dan itu bukan fungsi) | Kalau dilanggar, ekspresinya ambigu |
| Cek dengan nilai ekstrem & nilai batas | Cek setiap pasangan kasus |

**Urutan penulisan kasus TIDAK berarti apa-apa.** Tidak ada aturan "yang pertama cocok, dipakai". Justru karena itu disjoint **wajib**, bukan sekadar anjuran.

### 3.3 `else` dan `if-then-else`

- `else` berarti **negasi dari semua kondisi sebelumnya**. Dengan `else`, syarat lengkap otomatis terpenuhi.
- Kelemahannya: kasus yang diwakili `else` tidak terlihat di teks.
- Pakai `else` hanya bila sisanya adalah satu kelompok kasus yang wajar.

```
if kondisi then e1 else e2
  ≡
depend on
  kondisi       : e1
  not (kondisi) : e2
```
- `if-then-else` hanya untuk **dua kasus**. Untuk tiga kasus atau lebih, `depend on` jauh lebih mudah dibaca.
- **`if c then e` tanpa `else` TIDAK BERARTI** di notasi ini. Kalau c false, tidak ada nilai yang dihasilkan. Di imperatif "tidak melakukan apa-apa" adalah aksi sah, tetapi di fungsional "tidak menghasilkan nilai" bukan pilihan.

### 3.4 Prasyarat mempersempit kewajiban "lengkap"

Syarat lengkap hanya dituntut atas domain **yang tersisa setelah prasyarat**. Kasus yang tidak bisa dijawab dikeluarkan lewat prasyarat, bukan dipaksa dijawab.

> ⚠️ **Jangan pakai `else` untuk menutupi pelanggaran prasyarat.** `else` bisa mengubah pelanggaran prasyarat menjadi jawaban keliru yang kelihatan wajar (lihat Kasus 2).

### 3.5 Pembatasan domain/range

- Selang terbatas: `Integer [1..12]`, `Integer [1..4]`
- Selang tak terbatas: `Integer > 0`, `Real ≤ 13.0`
- Pembatasan ini **tidak diperiksa mesin**. Ia bagian dari spesifikasi, bukan kode.

### 3.6 Operator hubung-singkat (short-circuit)

| `A and then B` | `A or else B` |
|---|---|
| ≡ `if A then B else false` | ≡ `if A then true else B` |
| B dievaluasi **hanya bila A true** | B dievaluasi **hanya bila A false** |
| Dipakai bila B tidak selalu terdefinisi | Bentuk cerminnya |
| **Tidak komutatif** | **Tidak komutatif** |

`and` dan `or` biasa **menuntut kedua operannya bernilai**. Operator hubung-singkat adalah **satu-satunya tempat urutan bermakna** dalam notasi ini.

### 3.7 Evaluasi ekspresi kondisional

Evaluasinya dua tahap: (1) kondisi dievaluasi sampai ditemukan yang benar, (2) ekspresi pasangannya dievaluasi. Ekspresi pada kasus lain **tidak dievaluasi**. Karena kasusnya disjoint, penelusuran boleh berhenti di kondisi benar pertama. Karena urutan tidak bermakna, pemeriksaan boleh dimulai dari kondisi mana saja.

### 3.8 Beda dengan Haskell

- Guard Haskell (`|`) diperiksa **berurutan**; guard pertama yang True dipakai.
- Haskell **tidak memaksa disjoint**; hasilnya ditentukan urutan penulisan.
- `&&` dan `||` di Haskell sudah berperilaku seperti `and then` dan `or else`.

### 3.9 Daftar periksa kesalahan

| Kesalahan | Gejala | Cara memeriksa |
|---|---|---|
| Kasus tidak lengkap | Ada masukan tanpa hasil | Coba nilai ekstrem & batas |
| Kasus tumpang tindih | Dua kasus benar bersamaan | Periksa setiap pasangan kasus |
| Batas salah dimiliki | Salah **hanya** tepat di nilai batas | Uji setiap nilai batas |
| `else` menutupi lubang | Kasus yang terlupa menghasilkan jawaban yang tampak wajar | Tulis kondisi terakhir secara lengkap |
| `if` tanpa `else` | Sebagian masukan tanpa nilai | Tidak berlaku dalam notasi ini |
| `and` untuk operan yang tak selalu terdefinisi | Tidak terdefinisi padahal harusnya bernilai | Apakah operan kedua terdefinisi di seluruh domain? |

### 3.10 Contoh kasus

#### Kasus 1: Wujud air (menentukan pemilik nilai batas)

```
JUDUL Wujud Air pada Tekanan 1 atm

DEFINISI DAN SPESIFIKASI
  stateOfWater : Real → Character
  { stateOfWater (t) adalah wujud air pada suhu t °C dan tekanan 1 atm:
    'S' bila padat, 'L' bila cair, 'G' bila uap.
    Nilai di perbatasan dimiliki oleh wujud pada suhu yang lebih rendah:
    t = 0.0 → 'S', t = 100.0 → 'L'. }

REALISASI
  stateOfWater (t) :
    depend on t
      t ≤ 0.0                   : 'S'
      (t > 0.0) and (t ≤ 100.0) : 'L'
      t > 100.0                 : 'G'

APLIKASI
  ⇒ stateOfWater (-10.0)    'S'     { di dalam kasus 1 }
  ⇒ stateOfWater (0.0)      'S'     { tepat di batas 1 }
  ⇒ stateOfWater (25.0)     'L'     { di dalam kasus 2 }
  ⇒ stateOfWater (100.0)    'L'     { tepat di batas 2 }
  ⇒ stateOfWater (150.0)    'G'     { di dalam kasus 3 }
```
Poin penting:
- Soal tidak menyebutkan nilai batas masuk ke kasus mana, jadi **kita yang memutuskan lalu menuliskannya di spesifikasi**.
- Setiap kondisi ditulis utuh, misalnya `(t > 0.0) and (t ≤ 100.0)` dan bukan hanya `t ≤ 100.0`.
- Aplikasi mencakup satu nilai di dalam tiap kasus dan satu nilai tepat di tiap batas.

Penelusuran `stateOfWater (25.0)`:
```
⇒ stateOfWater (25.0)
→ depend on 25.0                                  { ekspansi }
    25.0 ≤ 0.0                       : 'S'
    (25.0 > 0.0) and (25.0 ≤ 100.0)  : 'L'
    25.0 > 100.0                     : 'G'
→ depend on 25.0                                  { reduksi ≤ }
    false                            : 'S'
    (25.0 > 0.0) and (25.0 ≤ 100.0)  : 'L'
    25.0 > 100.0                     : 'G'
→ depend on 25.0                                  { reduksi >, ≤, and }
    false : 'S'
    true  : 'L'
    25.0 > 100.0 : 'G'
→ 'L'
```

#### Kasus 2: Kuadran (prasyarat membuat "tidak lengkap" menjadi sah)

```
DEFINISI DAN SPESIFIKASI
  quadrant : Real, Real → Integer [1..4]
  { quadrant (x, y) adalah nomor kuadran tempat titik (x, y) berada.
    Prasyarat: x ≠ 0.0 and y ≠ 0.0 (titik tidak di sumbu mana pun) }

REALISASI
  quadrant (x, y) :
    depend on x, y
      (x > 0.0) and (y > 0.0) : 1
      (x < 0.0) and (y > 0.0) : 2
      (x < 0.0) and (y < 0.0) : 3
      (x > 0.0) and (y < 0.0) : 4
```
❌ **Versi keliru** dengan `else : 4`: `quadrant (0.0, 5.0)` menghasilkan `4`, padahal titik itu ada di sumbu Y dan tidak masuk kuadran mana pun. Jawaban ini keliru tetapi kelihatan wajar, sehingga sulit ditemukan.

#### Kasus 3: Pembagi habis (`and` vs `and then`)

```
JUDUL Pembagi Habis

DEFINISI DAN SPESIFIKASI
  isDivisor : Integer, Integer → Boolean
  { isDivisor (a, b) true bila b membagi habis a. False bila b = 0 }

REALISASI
  isDivisor (a, b) : (b ≠ 0) and then (a mod b = 0)

APLIKASI
  ⇒ isDivisor (12, 3)   true
  ⇒ isDivisor (12, 5)   false
  ⇒ isDivisor (12, 0)   false
```
Kalau ditulis `(b ≠ 0) and (a mod b = 0)`, hasil untuk b = 0 **tidak terdefinisi**, karena `and` tetap menuntut `12 mod 0` bernilai.

Penelusuran `isDivisor (12, 0)`:
```
⇒ isDivisor (12, 0)
→ (0 ≠ 0) and then (12 mod 0 = 0)     { ekspansi }
→ false and then (12 mod 0 = 0)       { reduksi ≠ }
→ false                                { and then: operan kanan tidak dievaluasi }
```
Kalau urutannya dibalik, `(12 mod 0 = 0) and then (0 ≠ 0)` **tidak terdefinisi**, karena operan kiri dievaluasi lebih dulu dan `12 mod 0` tidak punya nilai. Inilah alasan `and then` tidak komutatif.

#### Kasus 4: Ongkos kirim yang keliru (latihan memperbaiki)

```
deliveryFee : Real → Integer
{ ringan ≤ 1 kg → 10000; sedang > 1 s.d. 5 kg → 20000; berat > 5 kg → 35000.
  Prasyarat: b > 0.0 }

deliveryFee (b) :
  depend on b
    b < 1.0                 : 10000
    (b > 1.0) and (b ≤ 5.0) : 20000
    else                    : 35000
```
Jawaban:
1. **Masukan yang salah:** `deliveryFee (1.0)` menghasilkan `35000`, padahal seharusnya `10000`.
2. **Kenapa lengkap & disjoint tetap salah?** `else` menelan nilai b = 1.0 yang terlupa. Syarat lengkap terpenuhi secara teknis, tetapi kasusnya salah tempat.
3. **Yang diperbaiki realisasinya**, karena spesifikasi adalah kontrak dengan pemakai:
```
deliveryFee (b) :
  depend on b
    b ≤ 1.0                 : 10000
    (b > 1.0) and (b ≤ 5.0) : 20000
    b > 5.0                 : 35000
```
Kondisi terakhir sebaiknya ditulis lengkap, bukan `else`, supaya lubang serupa kelihatan.

#### Kasus 5: Latihan membaca (lengkap & disjoint?)

```
p (n) : depend on n          q (x) : depend on x        r (a, b) : depend on a, b
  n < 0 : -1                   x ≤ 10 : 'A'               a > b : a - b
  n = 0 : 0                    x ≥ 5  : 'B'               a < b : b - a
  n > 0 : 1                    else   : 'C'
```
| Fungsi | Arti | Lengkap? | Disjoint? |
|---|---|---|---|
| `p` → `sign` | tanda bilangan | ✅ | ✅ |
| `q` | — | ✅ (`else` tidak pernah tercapai, karena setiap x memenuhi x ≤ 10 atau x ≥ 5) | ❌ x ∈ [5..10] memenuhi dua kasus sekaligus |
| `r` → `absDif` | selisih mutlak | ❌ **a = b tidak tercakup** | ✅ |

Perbaikan `r`: ubah kasus pertama menjadi `a ≥ b : a - b`.

#### Kasus 6: Tarif parkir (latihan menulis)

```
JUDUL Tarif Parkir

DEFINISI DAN SPESIFIKASI
  parkingFee : Integer > 0 → Integer
  { parkingFee (j) adalah tarif parkir dalam rupiah untuk lama parkir j jam:
    dua jam pertama Rp5.000 (sekaligus), setiap jam berikutnya Rp3.000,
    dengan tarif maksimum sehari Rp50.000.
    Lama parkir dihitung dalam jam bulat. }

REALISASI
  parkingFee (j) :
    let
      normal : 5000 + (j - 2) * 3000
    in
      depend on j
        j ≤ 2                          : 5000
        (j > 2) and (normal ≤ 50000)   : normal
        (j > 2) and (normal > 50000)   : 50000

APLIKASI
  ⇒ parkingFee (1)    5000      { di dalam 2 jam pertama }
  ⇒ parkingFee (2)    5000      { batas 2 jam }
  ⇒ parkingFee (3)    8000      { jam tambahan pertama }
  ⇒ parkingFee (17)   50000     { tepat mencapai maksimum: 5000 + 15*3000 }
  ⇒ parkingFee (18)   50000     { melewati maksimum → dipotong }
```
Interpretasi soal (misalnya "dua jam pertama = Rp5.000 total") **harus ditulis di spesifikasi**.

#### Kasus 7: Maksimum tiga bilangan (dengan `depend on`, tanpa `max2`)

```
DEFINISI DAN SPESIFIKASI
  max3 : Integer, Integer, Integer → Integer
  { max3 (a, b, c) adalah nilai terbesar di antara a, b, dan c }

REALISASI
  max3 (a, b, c) :
    depend on a, b, c
      (a ≥ b) and (a ≥ c)          : a
      (b > a) and (b ≥ c)          : b
      (c > a) and (c > b)          : c

APLIKASI
  ⇒ max3 (1, 2, 3)    3
  ⇒ max3 (3, 3, 1)    3     { a = b: hanya kasus 1 yang benar }
  ⇒ max3 (2, 5, 5)    5     { b = c: hanya kasus 2 yang benar }
  ⇒ max3 (4, 4, 4)    4
```
Cara menjaga disjoint pada nilai yang sama: pakai `≥` dan `>` secara hati-hati supaya setiap seri (a = b, b = c, dst.) punya **tepat satu** pemilik. Bandingkan dengan versi yang jauh lebih pendek: `max3 (a, b, c) : max2 (a, max2 (b, c))`.

#### Kasus 8: Nilai mutlak tanpa `abs`

```
absolute : Integer → Integer ≥ 0
{ absolute (x) adalah nilai mutlak x }
absolute (x) : if x ≥ 0 then x else -x

⇒ absolute (-5)  5     ⇒ absolute (0)  0     ⇒ absolute (7)  7
```

---

## 4. Type Bentukan

### 4.1 Masalah yang melahirkannya

Pada `distance : Real, Real, Real, Real → Real`, empat Real itu sebenarnya dua **titik**, tetapi hal itu hanya ada di kepala penulis. Kalau urutan argumen tertukar, `distance (x1, x2, y1, y2)`, **tidak ada yang mencegahnya**. Type bentukan membuat gagasan "titik" tertulis secara eksplisit.

### 4.2 Type produk (tuple)

Satu nilai memegang beberapa komponen sekaligus, ditulis dengan kurung sudut:
```
⟨0.0, 0.0⟩      { titik pusat }
⟨3, 4⟩          { pecahan 3/4 }
```
Disebut "produk" karena himpunan nilainya adalah **hasil kali kartesian** dari himpunan nilai komponennya.

Contoh: `Point ⟨x, y⟩`, `Complex ⟨re, im⟩`, `Fraction ⟨numerator, denominator > 0⟩`, `Time ⟨hour, minute, second⟩`, `Date ⟨day, month, year⟩`, `Line ⟨start : Point, end : Point⟩`. Komponen juga boleh berupa type bentukan.

### 4.3 Lima bagian teks type (hafalkan!)

| Bagian | Isi | Direalisasi? |
|---|---|---|
| **Definisi type** | Nama + komposisi | ❌ tidak |
| **Selektor** | Mengambil satu komponen | ❌ tidak |
| **Konstruktor** | Membentuk nilai dari komponen | ❌ tidak |
| **Predikat** | Menentukan karakteristik nilai | ✅ ya |
| **Operator lain** | Fungsi atas type tersebut | ✅ ya |

**Aturan akses:** komponen hanya diakses lewat **selektor** `x (p)`, dan nilai hanya dibentuk lewat **konstruktor** `makePoint (...)`. Teksnya jadi sedikit lebih panjang, tetapi susunan Point cukup diatur di satu tempat. Inilah bibit **abstraksi data**.

### 4.4 Invarian type

Invarian adalah sifat yang **harus berlaku bagi setiap nilai sepanjang hidupnya**, misalnya `denominator > 0` pada Fraction. Invarian **tidak dijamin notasi maupun bahasa**, jadi harus ditegakkan di dua tempat:
1. **Konstruktor**: prasyarat (misalnya `d > 0`).
2. **Setiap operator yang menghasilkan nilai baru** dari type itu. Hasilnya juga harus memenuhi invarian. Contoh: `d1 × d2 > 0` bila kedua penyebut positif.

Kalau satu operator lalai, nilai tidak sah menyebar dan gejalanya muncul jauh dari penyebabnya.

### 4.5 Kesamaan harus dirancang

`makeFraction (1, 2)` dan `makeFraction (2, 4)` **nilainya sama**, meskipun representasinya berbeda. Karena itu `isEqFraction` **harus direalisasi** dengan perkalian silang. Membandingkan komponen satu per satu akan memberi jawaban keliru. (Pada Point cukup membandingkan komponen, karena setiap nilai hanya punya satu representasi.)

### 4.6 Tuple sebagai hasil & pembongkaran dengan `let`

Sebuah fungsi boleh menghasilkan lebih dari satu nilai:
```
divMod : Integer, Integer → ⟨Integer, Integer⟩
{ divMod (n, d) = ⟨q, r⟩ dengan n = q*d + r dan 0 ≤ r < d. Prasyarat: d > 0 }
divMod (n, d) : ⟨n div d, n mod d⟩
```
Tuple bisa dibongkar di `let`: `⟨q, r⟩ : divMod (x, 60)`.

**Kapan type bentukan perlu diberi nama?** Kalau nilainya punya arti sendiri atau ada operasi yang berlaku atasnya. Kalau hanya lewat sebentar lalu dibongkar (seperti `⟨q, r⟩`), tuple tanpa nama sudah cukup.

### 4.7 Type alternatif

Nilainya adalah **salah satu** dari beberapa kemungkinan, ditulis dengan `|`:
```
type Shape : Circle ⟨r : Real > 0⟩
           | Rectangle ⟨w : Real > 0, h : Real > 0⟩
```
| Type alternatif | Type produk |
|---|---|
| Analisis kasus **otomatis lengkap & disjoint** | Jaminan itu harus dijaga penulis |
| Selektor melekat pada **satu** alternatif (`r (s)` tak bermakna untuk Rectangle) | Selektor berlaku untuk semua nilai |
| Setiap alternatif punya konstruktornya sendiri | Hanya satu konstruktor |

Catatan untuk bab berikutnya: **List adalah type alternatif**, yaitu list kosong **atau** elemen yang diikuti list.

### 4.8 Notasi vs Haskell

| Hanya notasi yang bisa **menuliskan** | Hanya bahasa yang bisa **memeriksa** |
|---|---|
| Batas domain `Integer [1..12]` | Pattern matching |
| Invarian `denominator > 0` | Kelengkapan analisis kasus |
| Prasyarat konstruktor | Peringatan kasus terlewat |

```haskell
data Point = Point { x :: Double, y :: Double }
data Shape = Circle Double | Rectangle Double Double
area (Circle r)      = pi * r * r
area (Rectangle w h) = w * h
```

### 4.9 Contoh kasus

#### Kasus 1: TYPE POINT lengkap

```
TYPE Point

DEFINISI TYPE
  type Point : ⟨x : Real, y : Real⟩
  { ⟨x, y⟩ adalah sebuah Point, x absis, y ordinat }

DEFINISI DAN SPESIFIKASI SELEKTOR
  x : Point → Real     { absis dari p }
  y : Point → Real     { ordinat dari p }

DEFINISI DAN SPESIFIKASI KONSTRUKTOR
  makePoint : Real, Real → Point
  { makePoint (a, b) membentuk Point ⟨a, b⟩ }

DEFINISI DAN SPESIFIKASI PREDIKAT
  isOrigin : Point → Boolean
  { true bila p adalah titik pusat ⟨0.0, 0.0⟩ }
  isOnAxis : Point → Boolean
  { true bila p terletak pada sumbu X atau sumbu Y }

DEFINISI DAN SPESIFIKASI OPERATOR LAIN
  distanceFromOrigin : Point → Real
  distance : Point, Point → Real
  quadrant : Point → Integer [1..4]
  { Prasyarat: not (isOnAxis (p)) }
  translate : Point, Real, Real → Point
  { translate (p, dx, dy) adalah p yang digeser sejauh dx searah sumbu X
    dan dy searah sumbu Y }

REALISASI
  isOrigin (p) : (x (p) = 0.0) and (y (p) = 0.0)
  isOnAxis (p) : (x (p) = 0.0) or (y (p) = 0.0)
  distanceFromOrigin (p) : sqrt (x (p) * x (p) + y (p) * y (p))
  distance (p1, p2) : sqrt (sqrDif (x (p1), x (p2)) + sqrDif (y (p1), y (p2)))
  quadrant (p) :
    depend on x (p), y (p)
      (x (p) > 0.0) and (y (p) > 0.0) : 1
      (x (p) < 0.0) and (y (p) > 0.0) : 2
      (x (p) < 0.0) and (y (p) < 0.0) : 3
      (x (p) > 0.0) and (y (p) < 0.0) : 4
  translate (p, dx, dy) : makePoint (x (p) + dx, y (p) + dy)

APLIKASI
  ⇒ isOrigin (makePoint (0.0, 0.0))                       true
  ⇒ isOnAxis (makePoint (0.0, 5.0))                       true
  ⇒ distanceFromOrigin (makePoint (3.0, 4.0))             5.0
  ⇒ distance (makePoint (1.0, 1.0), makePoint (4.0, 5.0)) 5.0
  ⇒ quadrant (makePoint (-3.0, 2.0))                      2
  ⇒ translate (makePoint (0.0, 1.0), 2.0, -2.0)           ⟨2.0, -1.0⟩
```
Keuntungan memberi nama: prasyarat `quadrant` sekarang cukup ditulis `not (isOnAxis (p))`, urutan argumen tidak bisa tertukar lagi, dan perbendaharaan fungsi bertambah.

#### Kasus 2: TYPE FRACTION (invarian + kesamaan)

```
TYPE Fraction

DEFINISI TYPE
  type Fraction : ⟨numerator : Integer, denominator : Integer > 0⟩
  { pecahan bernilai numerator/denominator. Penyebut selalu positif;
    tanda pecahan dibawa pembilang }

DEFINISI DAN SPESIFIKASI SELEKTOR
  numerator   : Fraction → Integer        { pembilang }
  denominator : Fraction → Integer > 0    { penyebut }

DEFINISI DAN SPESIFIKASI KONSTRUKTOR
  makeFraction : Integer, Integer → Fraction
  { membentuk pecahan n/d. Prasyarat: d > 0 }

DEFINISI DAN SPESIFIKASI PREDIKAT
  isEqFraction : Fraction, Fraction → Boolean   { true bila nilai f1 = nilai f2 }
  isLtFraction : Fraction, Fraction → Boolean   { true bila nilai f1 < nilai f2 }

DEFINISI DAN SPESIFIKASI OPERATOR LAIN
  addFraction : Fraction, Fraction → Fraction
  subFraction : Fraction, Fraction → Fraction
  mulFraction : Fraction, Fraction → Fraction
  toReal      : Fraction → Real

REALISASI
  isEqFraction (f1, f2) :
    numerator (f1) * denominator (f2) = numerator (f2) * denominator (f1)

  isLtFraction (f1, f2) :
    numerator (f1) * denominator (f2) < numerator (f2) * denominator (f1)
    { sah karena kedua penyebut positif (invarian) }

  addFraction (f1, f2) :
    makeFraction (numerator (f1) * denominator (f2) + numerator (f2) * denominator (f1),
                  denominator (f1) * denominator (f2))

  subFraction (f1, f2) :
    makeFraction (numerator (f1) * denominator (f2) - numerator (f2) * denominator (f1),
                  denominator (f1) * denominator (f2))

  mulFraction (f1, f2) :
    makeFraction (numerator (f1) * numerator (f2),
                  denominator (f1) * denominator (f2))

  toReal (f) : asReal (numerator (f)) / asReal (denominator (f))

APLIKASI
  ⇒ isEqFraction (makeFraction (1,2), makeFraction (2,4))   true
  ⇒ addFraction (makeFraction (1,2), makeFraction (1,3))    ⟨5, 6⟩
  ⇒ mulFraction (makeFraction (2,3), makeFraction (3,4))    ⟨6, 12⟩
  ⇒ subFraction (makeFraction (1,2), makeFraction (1,3))    ⟨1, 6⟩
  ⇒ toReal (makeFraction (1,4))                             0.25
```
**Latihan memperbaiki, `subFraction` yang keliru:** versi di slide memakai penyebut `denominator (f1) - denominator (f2)`.
- `subFraction (makeFraction (1,2), makeFraction (1,2))` menghasilkan penyebut 2 − 2 = **0**.
- `subFraction (makeFraction (1,2), makeFraction (1,3))` menghasilkan penyebut 2 − 3 = **−1**.
- Keduanya **melanggar invarian `denominator > 0`** (dan prasyarat `makeFraction`). Perbaikannya: penyebut hasil adalah `d1 * d2`.

Perhatikan juga isLtFraction. Perkalian silang untuk `<` **hanya sah karena invarian** menjamin penyebut positif. Kalau penyebut boleh negatif, arah pertidaksamaannya bisa terbalik.

#### Kasus 3: TYPE DATE (perbandingan berjenjang)

```
DEFINISI TYPE
  type Date : ⟨day : Integer [1..31], month : Integer [1..12], year : Integer > 0⟩

KONSTRUKTOR
  makeDate : Integer, Integer, Integer → Date   { Prasyarat: isValidDate (d, m, y) }

PREDIKAT / OPERATOR
  isValidDate  : Integer, Integer, Integer → Boolean
  isBefore     : Date, Date → Boolean
  nDaysInMonth : Integer [1..12], Integer > 0 → Integer [28..31]

REALISASI
  nDaysInMonth (m, y) :
    depend on m
      m = 2                                    : if isLeapYear (y) then 29 else 28
      (m = 4) or (m = 6) or (m = 9) or (m = 11) : 30
      else                                     : 31

  isValidDate (d, m, y) :
    ((y > 0) and (m ≥ 1) and (m ≤ 12) and (d ≥ 1))
      and then (d ≤ nDaysInMonth (m, y))
    { and then: nDaysInMonth hanya terdefinisi bila m ∈ [1..12], y > 0 }

  isBefore (d1, d2) :
    let
      sameYear  : year (d1) = year (d2)
      sameMonth : month (d1) = month (d2)
    in
      depend on d1, d2
        not (sameYear)                 : year (d1) < year (d2)
        sameYear and not (sameMonth)   : month (d1) < month (d2)
        sameYear and sameMonth         : day (d1) < day (d2)

APLIKASI
  ⇒ nDaysInMonth (2, 1900)                                  28
  ⇒ nDaysInMonth (2, 2000)                                  29
  ⇒ isValidDate (29, 2, 2023)                               false
  ⇒ isValidDate (29, 2, 2024)                               true
  ⇒ isBefore (makeDate (31,12,2023), makeDate (1,1,2024))   true
  ⇒ isBefore (makeDate (1,3,2024), makeDate (1,3,2024))     false
```
Pola **perbandingan berjenjang**: bandingkan komponen yang paling berpengaruh lebih dulu (tahun → bulan → hari).

#### Kasus 4: TYPE TIME (latihan "giliran Anda")

```
TYPE Time

DEFINISI TYPE
  type Time : ⟨hour : Integer [0..23], minute : Integer [0..59], second : Integer [0..59]⟩
  { waktu dalam sehari dengan format 24 jam }
  { Invarian: setiap komponen berada dalam selangnya }

SELEKTOR
  hour   : Time → Integer [0..23]
  minute : Time → Integer [0..59]
  second : Time → Integer [0..59]

KONSTRUKTOR
  makeTime : Integer, Integer, Integer → Time
  { Prasyarat: isValidTime (h, m, s) }

PREDIKAT
  isValidTime : Integer, Integer, Integer → Boolean
  { true bila h ∈ [0..23], m ∈ [0..59], s ∈ [0..59] }
  isBeforeTime : Time, Time → Boolean
  { true bila t1 terjadi lebih awal daripada t2 pada hari yang sama }

OPERATOR LAIN
  toSeconds : Time → Integer [0..86399]
  { banyaknya detik sejak tengah malam }

REALISASI
  isValidTime (h, m, s) :
    (h ≥ 0) and (h ≤ 23) and (m ≥ 0) and (m ≤ 59) and (s ≥ 0) and (s ≤ 59)
  toSeconds (t) : hour (t) * 3600 + minute (t) * 60 + second (t)
  isBeforeTime (t1, t2) : toSeconds (t1) < toSeconds (t2)

APLIKASI
  ⇒ isValidTime (24, 0, 0)                                  false
  ⇒ toSeconds (makeTime (14, 20, 30))                       51630
  ⇒ isBeforeTime (makeTime (9,0,0), makeTime (14,20,30))    true
  ⇒ isBeforeTime (makeTime (9,0,0), makeTime (9,0,0))       false
```
Bandingkan dengan `isBefore` pada Date. Time bisa dibandingkan lewat **konversi ke satu bilangan** (`toSeconds`). Pada Date cara ini sulit karena panjang bulan dan tahun tidak seragam, sehingga Date memakai perbandingan berjenjang.

#### Kasus 5: `toDHMS` (tuple + `let`) dan penelusurannya

```
DEFINISI DAN SPESIFIKASI
  toDHMS : Integer ≥ 0 → ⟨Integer, Integer, Integer, Integer⟩
  { toDHMS (x) memecah x detik menjadi ⟨hari, jam, menit, detik⟩ }

REALISASI
  toDHMS (x) :
    let
      ⟨d, remD⟩ : divMod (x, 86400)
      ⟨h, remH⟩ : divMod (remD, 3600)
      ⟨m, s⟩    : divMod (remH, 60)
    in
      ⟨d, h, m, s⟩

APLIKASI
  ⇒ toDHMS (90061)   ⟨1, 1, 1, 1⟩
  ⇒ toDHMS (59)      ⟨0, 0, 0, 59⟩
  ⇒ toDHMS (3661)    ⟨0, 1, 1, 1⟩
```
Penelusuran `toDHMS (3661)`:
```
⇒ toDHMS (3661)
→ let ⟨d, remD⟩ : divMod (3661, 86400) ... in ⟨d, h, m, s⟩   { ekspansi toDHMS }
→ ⟨d, remD⟩ : ⟨3661 div 86400, 3661 mod 86400⟩               { ekspansi divMod }
→ ⟨d, remD⟩ : ⟨0, 3661⟩                                       { reduksi div, mod → d=0, remD=3661 }
→ ⟨h, remH⟩ : divMod (3661, 3600) → ⟨1, 61⟩                   { ekspansi + reduksi → h=1, remH=61 }
→ ⟨m, s⟩    : divMod (61, 60)     → ⟨1, 1⟩                    { ekspansi + reduksi → m=1, s=1 }
→ ⟨0, 1, 1, 1⟩
```
Tanpa `let`, `x mod 86400` harus dihitung tiga kali, dan pembaca juga harus memeriksa bahwa keempat baris itu membicarakan pemecahan yang sama.

#### Kasus 6: Latihan membaca type Segment

```
type Segment : ⟨a : Point, b : Point⟩
g (s) : makePoint ((x (a (s)) + x (b (s))) / 2.0, (y (a (s)) + y (b (s))) / 2.0)
h (s) : (x (a (s)) = x (b (s))) or (y (a (s)) = y (b (s)))
```
Jawaban:
```
midPoint : Segment → Point
{ midPoint (s) adalah titik tengah segmen s }

isAxisParallel : Segment → Boolean
{ true bila segmen s sejajar sumbu Y (absis sama) atau sejajar sumbu X (ordinat sama) }
```
Kasus yang membuat nama `h` menyesatkan: jika **a = b** (segmen berupa satu titik), `h` bernilai true padahal "segmen" itu tidak sejajar dengan apa pun. Solusinya: tambahkan prasyarat `a ≠ b` di definisi type, atau nyatakan kasus tersebut di spesifikasi.

#### Kasus 7: Type alternatif Shape

```
TYPE Shape

DEFINISI TYPE
  type Shape : Circle ⟨r : Real > 0⟩ | Rectangle ⟨w : Real > 0, h : Real > 0⟩

SELEKTOR
  r : Circle → Real > 0       w : Rectangle → Real > 0       h : Rectangle → Real > 0

KONSTRUKTOR
  makeCircle    : Real → Circle             { Prasyarat: a > 0.0 }
  makeRectangle : Real, Real → Rectangle    { Prasyarat: a > 0.0 and b > 0.0 }

PREDIKAT
  isCircle : Shape → Boolean       isRectangle : Shape → Boolean

OPERATOR LAIN
  area      : Shape → Real > 0
  perimeter : Shape → Real > 0

REALISASI
  area (s) :
    depend on s
      isCircle (s)    : pi * r (s) * r (s)
      isRectangle (s) : w (s) * h (s)

  perimeter (s) :
    depend on s
      isCircle (s)    : 2.0 * pi * r (s)
      isRectangle (s) : 2.0 * (w (s) + h (s))

APLIKASI
  ⇒ area (makeCircle (1.0))              3.14159
  ⇒ area (makeRectangle (3.0, 4.0))      12.0
  ⇒ perimeter (makeRectangle (3.0, 4.0)) 14.0
```
Kasus kedua **tidak** ditulis `else`. Menulis `isRectangle (s)` sama panjangnya, dan jaminan type-nya jadi terlihat.

---

## 5. Ekspresi Rekursif

### 5.1 Yang baru hanyalah sebuah izin

Nama fungsi **boleh muncul di dalam realisasinya sendiri**, dan nama type boleh muncul di dalam definisinya sendiri. Dengan izin ini, teks yang panjangnya tetap bisa mengerjakan sebanyak apa pun yang dituntut masukannya.

### 5.2 Dua kewajiban (sama-sama wajib!)

1. **Ada kasus yang berhenti (basis).** Paling sedikit satu kasus tidak mengandung aplikasi fungsi itu sendiri.
2. **Rekurens menuju basis (konvergensi).** Setiap aplikasi rekursif memakai argumen yang lebih dekat ke basis. **Sebutkan besaran yang mengecil beserta batasnya.**

Melanggar salah satunya membuat fungsi tidak berhenti. Pelanggaran kewajiban kedua paling sulit dilihat, karena fungsinya bisa tampak sah padahal tidak menghasilkan apa-apa.

### 5.3 Bentuk umum

```
namaFungsi (x) :
  depend on x
    kondisi basis    : ekspresi TANPA namaFungsi
    kondisi rekurens : ekspresi DENGAN namaFungsi
```
Kewajiban ditulis sebagai komentar di spesifikasi:
```
{ Basis: kondisi → hasil
  Rekurens: kondisi → ekspresi rekursif }
```
Kondisi rekurens ditulis utuh, **tidak memakai `else`**.

### 5.4 Poin-poin konseptual

- **Basis bukan berarti "kecil"**, melainkan kasus yang jawabannya sudah diketahui langsung. Basis juga boleh berupa kelompok nilai (misalnya `n < 10`).
- **Memilih basis berarti memilih domain.** Jika basisnya `n = 1`, fungsi tidak terdefinisi untuk n = 0.
- **Rekursi tidak selalu `n − 1`.** Bisa juga `n div 10`, `n div 2`, `a mod b`, atau menukar argumen. Yang dituntut hanya: argumen mendekati basis.
- **Rumus yang benar ≠ program yang berhenti.** Contohnya `n! = (n+1)!/(n+1)`: rumusnya benar, tetapi argumennya menjauhi basis.
- **Banyaknya basis ditentukan oleh sejauh mana rekurens melangkah mundur.** Kalau mundur dua langkah (Fibonacci), butuh dua basis.
- **Sumbu ketiga: banyaknya kerja.** Contoh: `power` vs `fastPower`. Yang lebih cepat biasanya lebih sulit dipercaya.
- **Akumulator** membawa hasil sementara maju, sehingga penelusurannya menjadi satu garis lurus.
- **Rekursi tidak langsung (saling rekursif):** f memanggil g, dan g memanggil f.
- **Type rekursif** (seperti `Nat`, `List`): kedua kewajiban **datang gratis** dari definisi type-nya. Integer **bukan** type rekursif, jadi setiap fungsi rekursif atas Integer harus membuktikan konvergensinya sendiri.

### 5.5 Kesalahan yang sering terjadi

| # | Kesalahan | Cara memeriksa |
|---|---|---|
| 1 | Tidak ada basis | Adakah ≥ 1 kasus yang tidak memuat nama fungsi itu sendiri? |
| 2 | Rekurens tidak menuju basis | Sebutkan besaran yang mengecil + batas bawahnya |
| 3 | Basis kurang | Kalau mundur k langkah, adakah k buah basis? |
| 4 | Rekurens melompati basis | Telusuri masukan terkecil di daerah rekurens: apakah mendarat tepat di basis? |
| 5 | Rekurens keluar domain | Apakah argumen rekursif masih memenuhi domain & prasyarat? |
| 6 | Kasus tidak disjoint | Sama seperti Bab 4 |
| 7 | Pekerjaan berulang | Gambar pohon aplikasi: adakah aplikasi yang sama muncul berkali-kali? |

Kesalahan #2 adalah satu-satunya yang **tidak bisa ditemukan dengan mencoba nilai batas**. Hasilnya bukan jawaban keliru, melainkan tidak ada jawaban sama sekali.

### 5.6 Padanan Haskell

```haskell
factorial 0 = 1
factorial n = n * factorial (n - 1)     -- pola diperiksa berurutan; domain hidup di spesifikasi

gcd' a 0 = a
gcd' a b = gcd' b (a `mod` b)

data Nat = Zero | Succ Nat
```

### 5.7 Contoh kasus

#### Kasus 1: Faktorial + penelusuran

```
JUDUL Faktorial

DEFINISI DAN SPESIFIKASI
  factorial : Integer ≥ 0 → Integer > 0
  { factorial (n) adalah n!, hasil kali seluruh bilangan bulat 1..n.
    factorial (0) bernilai 1 }
  { Basis: n = 0 → 1
    Rekurens: n > 0 → n * factorial (n - 1) }

REALISASI
  factorial (n) :
    depend on n
      n = 0 : 1
      n > 0 : n * factorial (n - 1)

APLIKASI
  ⇒ factorial (0)   1       { tepat di basis }
  ⇒ factorial (1)   1       { tepat di atas basis }
  ⇒ factorial (5)   120     { jauh di dalam daerah rekurens }
```
**Konvergensi:** n berkurang 1 setiap langkah dan terbatas di bawah oleh 0.

```
⇒ factorial (4)
→ 4 * factorial (3)                          ─┐
→ 4 * (3 * factorial (2))                     │ MEMBESAR
→ 4 * (3 * (2 * factorial (1)))               │ (ekspansi)
→ 4 * (3 * (2 * (1 * factorial (0))))         │
→ 4 * (3 * (2 * (1 * 1)))       { basis }    ─┘
→ 4 * (3 * (2 * 1))                          ─┐
→ 4 * (3 * 2)                                 │ MENGECIL
→ 4 * 6                                       │ (reduksi)
→ 24                                         ─┘
```

#### Kasus 2: Jumlah digit (basis berupa kelompok, rekurens `div 10`)

```
DEFINISI DAN SPESIFIKASI
  sumDigits : Integer ≥ 0 → Integer ≥ 0
  { jumlah seluruh digit n dalam penulisan desimal }
  { Basis: n < 10 → n
    Rekurens: n ≥ 10 → (n mod 10) + sumDigits (n div 10) }

REALISASI
  sumDigits (n) :
    depend on n
      n < 10 : n
      n ≥ 10 : (n mod 10) + sumDigits (n div 10)

APLIKASI
  ⇒ sumDigits (7)      7
  ⇒ sumDigits (10)     1
  ⇒ sumDigits (2026)   10
```
**Konvergensi:** `n div 10 < n` untuk n ≥ 10, dan batas bawahnya 10. `sumDigits (1000000)` mencapai basis dalam 7 langkah.

#### Kasus 3: Latihan memperbaiki `nDigits`

```
nDigits : Integer ≥ 0 → Integer > 0
{ banyaknya digit n. nDigits (0) bernilai 1 }
nDigits (n) : depend on n
  n = 0 : 0
  n > 0 : 1 + nDigits (n div 10)
```
Jawaban:
1. **Masukan salah:** `nDigits (0)` menghasilkan **0**, padahal menurut spesifikasi seharusnya **1**.
2. Hasil itu melanggar **range pada baris definisi** (`Integer > 0`).
3. **Yang diperbaiki adalah realisasi**, karena spesifikasi sudah benar (angka 0 memang ditulis dengan 1 digit).
4. Perbaikan:
```
nDigits (n) :
  depend on n
    n < 10  : 1
    n ≥ 10  : 1 + nDigits (n div 10)
{ Basis: n < 10 → 1 ; Rekurens: n ≥ 10 → 1 + nDigits (n div 10) }
```
Bandingkan dengan `sumDigits`: rekurensnya serupa, tetapi basisnya berbeda. Untuk n < 10, *jumlah* digitnya adalah n, sedangkan *banyaknya* digit adalah 1.

5. Aplikasi lengkap:
```
⇒ nDigits (0)     1     { batas bawah, sebelumnya gagal }
⇒ nDigits (9)     1     { batas atas basis }
⇒ nDigits (10)    2     { rekurens terkecil }
⇒ nDigits (2026)  4
```

#### Kasus 4: GCD + penelusuran

```
DEFINISI DAN SPESIFIKASI
  gcd : Integer ≥ 0, Integer ≥ 0 → Integer > 0
  { bilangan positif terbesar yang membagi habis a maupun b.
    Prasyarat: a dan b tidak keduanya nol }
  { Basis: b = 0 → a
    Rekurens: b > 0 → gcd (b, a mod b) }

REALISASI
  gcd (a, b) :
    depend on b
      b = 0 : a
      b > 0 : gcd (b, a mod b)
```
```
⇒ gcd (48, 18)           ⇒ gcd (18, 48)
→ gcd (18, 12)           → gcd (48, 18)   { 18 mod 48 = 18 → hanya menukar }
→ gcd (12, 6)            → gcd (18, 12)
→ gcd (6, 0)             → gcd (12, 6)
→ 6                      → gcd (6, 0)
                         → 6
```
**Konvergensi:** argumen kedua mengecil karena `a mod b < b`, dengan batas bawah 0. Argumen pertama tidak selalu mengecil. `gcd (18, 48)` butuh satu langkah lebih banyak karena langkah pertamanya hanya menukar argumen.

**Rekursi dalam fungsi antara:** `simplify` sendiri tidak rekursif, tetapi memakai `gcd`:
```
simplify : Fraction → Fraction
{ simplify (f) adalah pecahan bernilai sama dalam bentuk paling sederhana }
simplify (f) :
  let g : gcd (abs (numerator (f)), denominator (f))
  in makeFraction (numerator (f) div g, denominator (f) div g)

⇒ simplify (makeFraction (6, 12))    ⟨1, 2⟩
⇒ simplify (makeFraction (-4, 6))    ⟨-2, 3⟩
```
`abs` diperlukan karena domain gcd adalah Integer ≥ 0. Prasyarat gcd otomatis terpenuhi karena invarian menjamin `denominator > 0`.

#### Kasus 5: `power` vs `fastPower` (sumbu ketiga: banyaknya kerja)

```
power : Integer, Integer ≥ 0 → Integer
{ x pangkat n }
{ Basis: n = 0 → 1 ; Rekurens: n > 0 → x * power (x, n - 1) }
power (x, n) :
  depend on n
    n = 0 : 1
    n > 0 : x * power (x, n - 1)

fastPower : Integer, Integer ≥ 0 → Integer
{ x pangkat n, dengan ide xⁿ = (x^(n/2))² bila n genap }
fastPower (x, n) :
  depend on n
    n = 0                       : 1
    (n > 0) and (n mod 2 = 0)   : square (fastPower (x, n div 2))
    (n > 0) and (n mod 2 = 1)   : x * fastPower (x, n - 1)
```
| n | aplikasi `power` | aplikasi `fastPower` | jalur fastPower |
|---|---|---|---|
| 7 | 8 | 6 | 7→6→3→2→1→0 |
| 8 | 9 | 5 | 8→4→2→1→0 |
| 10 | 11 | 6 | 10→5→4→2→1→0 |
| 100 | 101 | 10 | |

Selisihnya berbeda untuk pangkat 7 dan 8 karena bilangan ganjil memakan satu langkah `n − 1` sebelum bisa dibagi dua.

#### Kasus 6: Fibonacci (dua basis) & akumulator

```
fibonacci : Integer ≥ 0 → Integer ≥ 0
{ suku ke-n deret Fibonacci }
{ Basis: n = 0 → 0 ; Basis: n = 1 → 1
  Rekurens: n > 1 → fibonacci (n - 1) + fibonacci (n - 2) }
fibonacci (n) :
  depend on n
    n = 0 : 0
    n = 1 : 1
    n > 1 : fibonacci (n - 1) + fibonacci (n - 2)
```
Kenapa perlu dua basis? Dengan basis `n = 0` saja, `fibonacci (1)` akan menuntut `fibonacci (−1)`, yang berada di luar domain.

**Pohon aplikasi `fibonacci (6)`:** total **25 aplikasi**, dan `fibonacci (2)` muncul **5 kali**. (Rumus: A(n) = 1 + A(n−1) + A(n−2), dengan A(0) = A(1) = 1, sehingga A(5) = 15, A(6) = 25, A(7) = 41.)

**Versi berakumulator:**
```
fibFrom : Integer ≥ 0, Integer, Integer → Integer
{ fibFrom (k, a, b): bila a dan b dua suku berurutan, hasilnya suku ke-k sesudah a }
{ Basis: k = 0 → a ; Rekurens: k > 0 → fibFrom (k - 1, b, a + b) }
fibFrom (k, a, b) :
  depend on k
    k = 0 : a
    k > 0 : fibFrom (k - 1, b, a + b)

fibonacci2 (n) : fibFrom (n, 0, 1)
```
```
⇒ fibonacci2 (6)
→ fibFrom (6, 0, 1)
→ fibFrom (5, 1, 1)
→ fibFrom (4, 1, 2)
→ fibFrom (3, 2, 3)
→ fibFrom (2, 3, 5)
→ fibFrom (1, 5, 8)
→ fibFrom (0, 8, 13)
→ 8                      { 7 aplikasi fibFrom vs 25 aplikasi fibonacci }
```
Bentuknya mirip loop, tetapi **tidak ada yang berubah nilai**. Setiap aplikasi adalah aplikasi baru atas argumen yang berbeda.

#### Kasus 7: Latihan membaca `p`, `q`

```
p (n) : depend on n                q (a, b) : depend on b
  n = 0 : 0                          b = 0 : 0
  n > 0 : n + p (n - 1)              b > 0 : a + q (a, b - 1)
```
Jawaban:
```
sumTo : Integer ≥ 0 → Integer ≥ 0
{ sumTo (n) adalah 1 + 2 + … + n; bernilai 0 bila n = 0 }
{ Basis: n = 0 → 0 ; Rekurens: n > 0 → n + sumTo (n - 1) }
{ Konvergensi: n berkurang 1, batas bawah 0 }

multiply : Integer, Integer ≥ 0 → Integer
{ multiply (a, b) adalah a × b, dihitung sebagai penjumlahan berulang }
{ Basis: b = 0 → 0 ; Rekurens: b > 0 → a + multiply (a, b - 1) }
{ Konvergensi: b berkurang 1, batas bawah 0 }
```
Kalau b boleh negatif, tidak ada kasus yang cocok (tidak lengkap). Kalau kondisinya diubah menjadi `else`, b akan terus menjauhi 0 dan fungsi tidak berhenti. Pencegahannya: **batasi domain** `b : Integer ≥ 0`.

#### Kasus 8: `reverseDigits` & `isPalindromeNumber` (akumulator)

```
JUDUL Membalik Digit

DEFINISI DAN SPESIFIKASI
  revOnto : Integer ≥ 0, Integer ≥ 0 → Integer ≥ 0
  { revOnto (n, acc) adalah digit-digit n dalam urutan terbalik yang
    ditempelkan di belakang digit acc }
  { Basis: n = 0 → acc
    Rekurens: n > 0 → revOnto (n div 10, acc * 10 + n mod 10) }

  reverseDigits : Integer ≥ 0 → Integer ≥ 0
  { membalik urutan digit n; nol di ujung n hilang (1230 → 321) }

  isPalindromeNumber : Integer ≥ 0 → Boolean
  { true bila n dibaca sama dari kiri dan kanan }

REALISASI
  revOnto (n, acc) :
    depend on n
      n = 0 : acc
      n > 0 : revOnto (n div 10, acc * 10 + n mod 10)

  reverseDigits (n) : revOnto (n, 0)
  isPalindromeNumber (n) : reverseDigits (n) = n

APLIKASI
  ⇒ reverseDigits (0)        0
  ⇒ reverseDigits (1230)     321
  ⇒ isPalindromeNumber (121)   true
  ⇒ isPalindromeNumber (1230)  false
```
Penelusuran `reverseDigits (1230)`: `revOnto (1230, 0) → (123, 0) → (12, 3) → (1, 32) → (0, 321) → 321`. **Konvergensi:** n mengecil lewat `div 10`, batas bawah 0.

#### Kasus 9: `isPrime` (fungsi antara dengan parameter tambahan)

```
DEFINISI DAN SPESIFIKASI
  hasNoDivisorFrom : Integer > 1, Integer → Boolean
  { hasNoDivisorFrom (n, k) true bila tidak ada bilangan d dengan k ≤ d < n
    yang membagi habis n. Prasyarat: 2 ≤ k ≤ n }
  { Basis: k = n → true
    Rekurens: k < n → (n mod k ≠ 0) and then hasNoDivisorFrom (n, k + 1) }

  isPrime : Integer > 1 → Boolean
  { true bila n bilangan prima }

REALISASI
  hasNoDivisorFrom (n, k) :
    depend on k
      k = n : true
      k < n : (n mod k ≠ 0) and then hasNoDivisorFrom (n, k + 1)

  isPrime (n) : hasNoDivisorFrom (n, 2)

APLIKASI
  ⇒ isPrime (2)    true     { langsung basis }
  ⇒ isPrime (9)    false    { berhenti di k = 3 }
  ⇒ isPrime (7)    true
```
**Konvergensi:** besaran `n − k` mengecil 1 setiap langkah, dengan batas bawah 0 (k = n). Bagian tersulitnya adalah menulis **spesifikasi fungsi antara**, yaitu arti parameter `k`.

#### Kasus 10: `dayOfYear` (rekursif atas nomor bulan)

```
daysBeforeMonth : Integer [1..12], Integer > 0 → Integer ≥ 0
{ banyaknya hari dari 1 Januari sampai sebelum bulan m, tahun y }
{ Basis: m = 1 → 0
  Rekurens: m > 1 → nDaysInMonth (m - 1, y) + daysBeforeMonth (m - 1, y) }
daysBeforeMonth (m, y) :
  depend on m
    m = 1 : 0
    m > 1 : nDaysInMonth (m - 1, y) + daysBeforeMonth (m - 1, y)

dayOfYear : Date → Integer [1..366]
{ nomor urut tanggal d dalam tahunnya; 1 Januari → 1 }
dayOfYear (d) : daysBeforeMonth (month (d), year (d)) + day (d)

⇒ dayOfYear (makeDate (1, 1, 2024))     1
⇒ dayOfYear (makeDate (1, 3, 2024))     61      { 31 + 29 + 1 }
⇒ dayOfYear (makeDate (31, 12, 2023))   365
```

#### Kasus 11: Rekursi tidak langsung & type rekursif

```
isEven (n) : depend on n              isOdd (n) : depend on n
  n = 0 : true                          n = 0 : false
  n > 0 : isOdd (n - 1)                 n > 0 : isEven (n - 1)

type Nat : Zero | Succ ⟨pred : Nat⟩
{ Nat adalah nol, atau penerus dari sebuah Nat }

toInteger (n) :
  depend on n
    isZero (n)       : 0
    not (isZero (n)) : 1 + toInteger (pred (n))

⇒ toInteger (Succ ⟨Succ ⟨Zero⟩⟩)    2
```

---

## 6. List

### 6.1 Motivasi

Satu `Date` menyimpan satu tanggal. Kalau banyaknya data **tidak diketahui saat program ditulis**, kita butuh **koleksi**: satu nama yang mewakili sekumpulan nilai (bisa kosong, satu, atau sejuta anggota). Pada **list**, anggotanya berurutan dan dicapai dari satu ujung. List adalah struktur rekursif yang paling sederhana.

### 6.2 Definisi type

```
TYPE List

DEFINISI TYPE
  type List : Nil | Cons ⟨head : Element, tail : List⟩
  { List adalah list kosong, atau sebuah elemen yang diikuti sebuah List }

SELEKTOR
  head : Cons → Element    { elemen pertama L }
  tail : Cons → List       { L tanpa elemen pertamanya }

KONSTRUKTOR
  makeNil  : → List                 { list kosong }
  makeCons : Element, List → List   { e sebagai elemen pertama, L sebagai sisanya }

PREDIKAT
  isEmpty      : List → Boolean     { true bila L kosong }
  isOneElement : List → Boolean     { true bila L tepat satu elemen }
```
- `Element` adalah **tempat kosong**, bukan type dasar. Semua elemen dalam satu list bertype sama (`List of Integer`, `List of Character`, `List of Date`).
- `head` dan `tail` hanya berlaku untuk **Cons** (list tidak kosong).
- Penulisan singkat:
```
[]          ≡ makeNil
[7]         ≡ makeCons (7, makeNil)
[1, 2, 3]   ≡ makeCons (1, makeCons (2, makeCons (3, makeNil)))
```
> ⚠️ Diktat lama memakai `head` untuk "list tanpa elemen terakhir". Di materi ini, **`head` = elemen PERTAMA**.

**List tidak simetris.** Elemen pertama murah dicapai (satu `head`), sedangkan elemen terakhir mahal karena terkubur di lapis paling dalam.

### 6.3 Basis-0 vs Basis-1

Satu pertanyaan penentu: **apakah fungsi ini punya jawaban untuk list kosong?**

| | Basis-0 | Basis-1 |
|---|---|---|
| Basis pada | list kosong | list satu elemen |
| Predikat | `isEmpty (L)` | `isOneElement (L)` |
| Dipakai bila | fungsi terdefinisi atas list kosong | fungsi **tidak** terdefinisi atas list kosong |
| Contoh | `length`, `sumList`, `isMember` | `maxList`, `earliest`, `lastElement` |
| Tambahan | — | List kosong dikeluarkan lewat **definisi** (`List tidak kosong`) **dan prasyarat** |

**Kewajiban rekursi datang gratis:** Nil adalah basis, Cons adalah rekurens, dan `tail (L)` selalu satu elemen lebih pendek, sehingga konvergensinya terjamin. Kondisi rekurens selalu ditulis utuh sebagai `not (isEmpty (L))`, **bukan** `else`.

### 6.4 Tiga bentuk rekurens menurut hasilnya

| Hasil | Pola rekurens | Contoh |
|---|---|---|
| **Nilai tunggal** | gabungkan `head (L)` dengan `f (tail (L))` | length, sumList, maxList, countChar |
| **List sama panjang** | `makeCons (ubah head (L), f (tail (L)))` | squareAll |
| **List mungkin lebih pendek** | sertakan **atau** lewati `head (L)` | keepPositive |

### 6.5 Aturan-aturan penting

- **Nilai basis harus netral** terhadap operator rekurensnya: `+` → 0, `*` → 1. Kalau basis productList diisi 0, semua hasilnya jadi 0.
- **`and then` wajib** kalau kondisi memakai `head (L)` atau `tail (L)`, karena keduanya tidak terdefinisi untuk list kosong.
- **`or else`** dipakai di `isMember` supaya penelusuran berhenti begitu elemennya ketemu.
- **Menambah di depan murah** (`makeCons`, 1 aplikasi). **Menambah di belakang mahal** (`addLast`, n+1 aplikasi, membangun ulang semua lapis).
- **Akumulator** hanya jelas menguntungkan untuk `reverse`. Untuk `length`, `sumList`, atau `squareAll`, akumulator hanya menambah panjang teks tanpa penghematan.
- **Teks adalah List of Character.** Tidak ada type teks tersendiri; `length`, `isMember`, dan `reverse` langsung berlaku.
- **Set** adalah List ditambah invarian "semua elemen berbeda, urutan tidak berarti". `makeCons` **bukan** konstruktor sah untuk Set; semua pembentukan harus melalui `insertSet`.

### 6.6 Kesalahan yang sering terjadi

| Kesalahan | Gejala | Periksa |
|---|---|---|
| Basis-0 padahal seharusnya basis-1 | Jawaban tampak wajar tetapi bukan anggota list | Adakah jawaban benar untuk list kosong? |
| Nilai basis salah | Benar untuk list panjang, salah untuk list pendek/kosong | Apakah basis netral terhadap operator? |
| `tail (L)` melanggar prasyarat | Gagal pada list satu elemen | Fungsi basis-1: apakah rekurens menjamin ≥ 2 elemen? |
| `and` padahal perlu `and then` | Tidak terdefinisi pada list kosong | Adakah kondisi yang memakai `head`/`tail`? |
| `makeCons` pada Set | Himpunan berisi elemen kembar | Semua pembentukan melalui `insertSet`? |
| Hasil terbalik | Elemen benar, urutan terbalik | Akibat akumulator; perlu `reverse` di ujung? |
| Rekursi pada list yang salah | Tidak berhenti | Apakah memakai `f (tail (L))` dan bukan `f (L)`? |

### 6.7 Padanan Haskell

| Notasi | Haskell |
|---|---|
| `type List : Nil \| Cons …` | `data [a] = [] \| a : [a]` |
| `makeNil` | `[]` |
| `makeCons (e, L)` | `e : L` |
| `head (L)`, `tail (L)` | `head l`, `tail l`, atau pola `(x : xs)` |
| `isEmpty (L)` | pola `[]`, atau `null l` |
| `isOneElement (L)` | pola `[x]` |
| `List of Character` | `String` = `[Char]` |
| `type Set : List + invarian` | **tidak ada padanan** (dijaga disiplin) |

```haskell
length' []       = 0
length' (_ : xs) = 1 + length' xs

keepPositive [] = []
keepPositive (x : xs)
  | x > 0     = x : keepPositive xs
  | otherwise = keepPositive xs
```
Pola `(x : xs)` mengerjakan tiga hal sekaligus: memeriksa list tidak kosong, mengambil head, dan mengambil tail. Karena itu `and then` tidak diperlukan lagi.

### 6.8 Contoh kasus

#### Kasus 1: `length` + penelusuran dengan konstruktor

```
DEFINISI DAN SPESIFIKASI
  length : List → Integer ≥ 0
  { banyaknya elemen L, nol bila L kosong }
  { Basis-0: isEmpty (L) → 0
    Rekurens: not (isEmpty (L)) → 1 + length (tail (L)) }

REALISASI
  length (L) :
    depend on L
      isEmpty (L)       : 0
      not (isEmpty (L)) : 1 + length (tail (L))

APLIKASI
  ⇒ length ([])          0
  ⇒ length ([7])         1
  ⇒ length ([1, 2, 3])   3
```
```
⇒ length (makeCons (1, makeCons (2, makeCons (3, makeNil))))
→ 1 + length (makeCons (2, makeCons (3, makeNil)))    { ekspansi, rekurens }
→ 1 + (1 + length (makeCons (3, makeNil)))            { ekspansi, rekurens }
→ 1 + (1 + (1 + length (makeNil)))                    { ekspansi, rekurens }
→ 1 + (1 + (1 + 0))                                   { ekspansi, basis-0 }
→ 1 + (1 + 1)                                         { reduksi + }
→ 1 + 2                                               { reduksi + }
→ 3                                                   { reduksi + }
```
Bentuknya sama persis dengan `factorial (4)`: membesar dulu, lalu mengecil. Penelusuran **ditulis dengan konstruktor**, bukan `[1,2,3]`, karena yang ingin diperlihatkan adalah pembongkaran konstruktornya.

#### Kasus 2: `isMember` (`or else`)

```
isMember : Element, List → Boolean
{ true bila x salah satu elemen L; false bila L kosong }
{ Basis-0: isEmpty (L) → false
  Rekurens: not (isEmpty (L)) → head (L) = x, atau x anggota tail (L) }
isMember (x, L) :
  depend on L
    isEmpty (L)       : false
    not (isEmpty (L)) : (head (L) = x) or else isMember (x, tail (L))

⇒ isMember (5, [])          false
⇒ isMember (2, [1, 2, 3])   true
⇒ isMember (9, [1, 2, 3])   false
```
Penelusuran pendek vs panjang:
```
⇒ isMember (1, [1,2,3])                ⇒ isMember (9, [1,2,3])
→ (1 = 1) or else isMember (1,[2,3])   → (1 = 9) or else isMember (9,[2,3])
→ true or else ...                     → false or else isMember (9,[2,3])
→ true     { berhenti }                → isMember (9, [2,3])
                                       → ... → isMember (9, []) → false
```
Ini fungsi list pertama yang **bisa selesai sebelum basis**, dan penyebabnya adalah `or else`.

#### Kasus 3: `sumList` & `productList` (nilai basis netral)

```
sumList : List of Integer → Integer
{ jumlah seluruh elemen L, nol bila L kosong }
{ Basis-0: isEmpty (L) → 0 ; Rekurens: head (L) + sumList (tail (L)) }
sumList (L) :
  depend on L
    isEmpty (L)       : 0
    not (isEmpty (L)) : head (L) + sumList (tail (L))

productList : List of Integer → Integer
{ hasil kali seluruh elemen L, satu bila L kosong }
{ Basis-0: isEmpty (L) → 1 ; Rekurens: head (L) * productList (tail (L)) }
productList (L) :
  depend on L
    isEmpty (L)       : 1
    not (isEmpty (L)) : head (L) * productList (tail (L))

⇒ sumList ([1, 2, 3])       6       ⇒ sumList ([])       0
⇒ productList ([1, 2, 3])   6       ⇒ productList ([])   1
```

#### Kasus 4: `maxList` (basis-1)

```
maxList : List of Integer tidak kosong → Integer
{ nilai elemen terbesar L. Prasyarat: L tidak kosong }
{ Basis-1: isOneElement (L) → head (L)
  Rekurens: not (isOneElement (L)) → max2 (head (L), maxList (tail (L))) }
maxList (L) :
  depend on L
    isOneElement (L)       : head (L)
    not (isOneElement (L)) : max2 (head (L), maxList (tail (L)))

⇒ maxList ([7])            7
⇒ maxList ([3, 9, 2])      9
⇒ maxList ([-5, -1, -8])   -1
```
Kenapa bukan basis-0 dengan hasil 0? Karena `maxList ([-5, -1, -8])` akan menghasilkan **0**, padahal 0 bukan anggota list-nya.

Konvergensi basis-1: kondisi rekurens berarti L punya ≥ 2 elemen, sehingga `tail (L)` punya ≥ 1 elemen dan prasyarat tetap terpenuhi.

#### Kasus 5: `squareAll` vs `keepPositive` (map vs filter)

```
squareAll : List of Integer → List of Integer
{ list berisi kuadrat setiap elemen L, urutan sama, panjang sama }
squareAll (L) :
  depend on L
    isEmpty (L)       : makeNil
    not (isEmpty (L)) : makeCons (square (head (L)), squareAll (tail (L)))

keepPositive : List of Integer → List of Integer
{ seluruh elemen positif L, urutan sama; banyaknya elemen dapat berkurang }
keepPositive (L) :
  depend on L
    isEmpty (L) : makeNil
    not (isEmpty (L)) and then (head (L) > 0) :
        makeCons (head (L), keepPositive (tail (L)))
    not (isEmpty (L)) and then (head (L) ≤ 0) :
        keepPositive (tail (L))

⇒ squareAll ([1, 2, 3])       [1, 4, 9]
⇒ keepPositive ([1, -2, 3])   [1, 3]
⇒ keepPositive ([-1, -2])     []
```

#### Kasus 6: `addLast`, `reverse`, dan `reverseOnto` (akumulator)

```
addLast : List, Element → List
{ seluruh elemen L, urutan sama, diikuti e sebagai elemen terakhir }
addLast (L, e) :
  depend on L
    isEmpty (L)       : makeCons (e, makeNil)
    not (isEmpty (L)) : makeCons (head (L), addLast (tail (L), e))

reverse (L) :                              { versi 1: benar tapi mahal }
  depend on L
    isEmpty (L)       : makeNil
    not (isEmpty (L)) : addLast (reverse (tail (L)), head (L))

reverseOnto : List, List → List
{ reverseOnto (L, acc): elemen L terbalik, diikuti acc }
reverseOnto (L, acc) :
  depend on L
    isEmpty (L)       : acc
    not (isEmpty (L)) : reverseOnto (tail (L), makeCons (head (L), acc))

reverse2 (L) : reverseOnto (L, makeNil)    { versi 2: murah }
```
Penelusuran `reverse2 ([1, 2, 3])`, dengan memperhatikan isi `acc`:
```
⇒ reverse2 ([1, 2, 3])
→ reverseOnto ([1, 2, 3], [])
→ reverseOnto ([2, 3], [1])
→ reverseOnto ([3], [2, 1])
→ reverseOnto ([], [3, 2, 1])
→ [3, 2, 1]
```
| | aplikasi reverse(Onto) | aplikasi addLast |
|---|---|---|
| `reverse ([1,2,3])` | 4 | 6 |
| `reverse2 ([1,2,3])` | 4 | 0 |

#### Kasus 7: `insertSorted` & `sort` (invarian keterurutan)

```
insertSorted : Integer, List of Integer → List of Integer
{ seluruh elemen L beserta x, terurut menaik. Prasyarat: L terurut menaik }
insertSorted (x, L) :
  depend on L
    isEmpty (L)                                : makeCons (x, makeNil)
    not (isEmpty (L)) and then (x ≤ head (L))  : makeCons (x, L)
    not (isEmpty (L)) and then (x > head (L))  : makeCons (head (L), insertSorted (x, tail (L)))

sort : List of Integer → List of Integer
{ sort (L) berisi seluruh elemen L, terurut menaik }
{ Basis-0: isEmpty (L) → [] ; Rekurens: sisipkan head (L) ke sort (tail (L)) }
sort (L) :
  depend on L
    isEmpty (L)       : makeNil
    not (isEmpty (L)) : insertSorted (head (L), sort (tail (L)))

⇒ insertSorted (5, [1, 7])   [1, 5, 7]
⇒ sort ([3, 1, 2])           [1, 2, 3]
⇒ sort ([])                  []
```
`sort` bisa sesingkat itu karena `sort (tail (L))` **dijamin terurut**, sehingga prasyarat `insertSorted` terpenuhi, dan `insertSorted` menjaga invarian keterurutan itu.

#### Kasus 8: `countChar` (list of character)

```
countChar : Character, List of Character → Integer ≥ 0
{ banyaknya kemunculan c di L }
countChar (c, L) :
  depend on L
    isEmpty (L) : 0
    not (isEmpty (L)) and then (head (L) = c) : 1 + countChar (c, tail (L))
    not (isEmpty (L)) and then (head (L) ≠ c) : countChar (c, tail (L))

⇒ countChar ('a', ['s','a','t','r','i','o'])   1
⇒ countChar ('a', [])                           0
```
Bentuknya gabungan `keepPositive` (percabangan) dan `length` (hasilnya bilangan). `countEven (L)` punya bentuk yang sama persis, cukup mengganti kondisinya dengan `head (L) mod 2 = 0`.

#### Kasus 9: `earliest` (list of type bentukan + `let`)

```
earliest : List of Date tidak kosong → Date
{ tanggal paling awal di L. Prasyarat: L tidak kosong }
{ Basis-1: isOneElement (L) → head (L)
  Rekurens: yang lebih awal antara head (L) dan earliest (tail (L)) }
earliest (L) :
  let
    rest : earliest (tail (L))
  in
    depend on L
      isOneElement (L)                                      : head (L)
      not (isOneElement (L)) and isBefore (head (L), rest)   : head (L)
      not (isOneElement (L)) and not (isBefore (head (L), rest)) : rest
```
`earliest` dan `maxList` punya bentuk yang sama. Bedanya hanya cara membandingkan (`isBefore` vs `max2`). Pola yang berulang ini menjadi alasan Bab 8 membahas *fungsi sebagai nilai*.

#### Kasus 10: Set (`insertSet`, `union`, `intersection`)

```
type Set : List
{ List yang seluruh elemennya berbeda; urutan tidak berarti }

insertSet : Element, Set → Set
{ anggota S beserta x; bila x sudah anggota, hasilnya S }
insertSet (x, S) : if isMember (x, S) then S else makeCons (x, S)

union : Set, Set → Set
{ himpunan beranggotakan seluruh anggota S1 maupun S2 }
union (S1, S2) :
  depend on S1
    isEmpty (S1)       : S2
    not (isEmpty (S1)) : insertSet (head (S1), union (tail (S1), S2))

intersection : Set, Set → Set
{ himpunan beranggotakan elemen yang ada di S1 dan juga di S2 }
{ Basis-0: isEmpty (S1) → []
  Rekurens: sertakan head (S1) bila anggota S2, lewati bila bukan }
intersection (S1, S2) :
  depend on S1
    isEmpty (S1) : makeNil
    not (isEmpty (S1)) and then isMember (head (S1), S2) :
        makeCons (head (S1), intersection (tail (S1), S2))
    not (isEmpty (S1)) and then not (isMember (head (S1), S2)) :
        intersection (tail (S1), S2)

⇒ insertSet (5, [5, 3])          [5, 3]
⇒ union ([1, 2], [2, 3])         [1, 2, 3]
⇒ intersection ([1, 2, 3], [2, 3, 4])   [2, 3]
```
Kenapa `makeCons` **boleh** dipakai di `intersection`? Karena S1 adalah Set (tidak ada elemen kembar), sehingga `head (S1)` pasti tidak muncul lagi di `tail (S1)`, dan hasilnya juga tidak akan kembar. **Invarian dijamin oleh invarian masukan.** Alternatif yang lebih aman adalah memakai `insertSet`.

Penelusuran `union ([1,2], [2,3])`:
```
→ insertSet (1, union ([2], [2, 3]))
→ insertSet (1, insertSet (2, union ([], [2, 3])))
→ insertSet (1, insertSet (2, [2, 3]))     { isMember (2, [2,3]) → true, 1 aplikasi }
→ insertSet (1, [2, 3])                    { isMember (1, [2,3]) → [3] → [] → false, 3 aplikasi }
→ makeCons (1, [2, 3])
→ [1, 2, 3]
{ total isMember diaplikasikan 4 kali }
```

#### Kasus 11: Latihan membaca `p`, `q`, `r`

```
p (L) : depend on L                        q (L) : depend on L
  isEmpty (L) : 0                            isOneElement (L) : head (L)
  not (isEmpty (L)) : head (L) + p (tail L)  not (isOneElement (L)) : q (tail (L))

r (x, L) : depend on L
  isEmpty (L) : makeNil
  not (isEmpty (L)) and then (head (L) = x) : r (x, tail (L))
  not (isEmpty (L)) and then (head (L) ≠ x) : makeCons (head (L), r (x, tail (L)))
```
| | Nama | Arti | Basis |
|---|---|---|---|
| p | `sumList` | jumlah elemen | basis-0 |
| q | `lastElement` | elemen terakhir; prasyarat L tidak kosong | **basis-1** (list kosong tidak punya elemen terakhir; dengan basis-0 tidak ada jawaban yang bisa dikembalikan) |
| r | `deleteAll` | L tanpa semua kemunculan x | basis-0 |

Untuk `r`: `r (1, [1, 2, 1])` → `[2]` (lebih pendek), dan `r (9, [1, 2])` → `[1, 2]` (sama panjang).

#### Kasus 12: `append` & `nth`

```
append : List, List → List
{ seluruh elemen L1 diikuti seluruh elemen L2 }
{ Basis-0: isEmpty (L1) → L2
  Rekurens: makeCons (head (L1), append (tail (L1), L2)) }
append (L1, L2) :
  depend on L1
    isEmpty (L1)       : L2
    not (isEmpty (L1)) : makeCons (head (L1), append (tail (L1), L2))

⇒ append ([], [3])        [3]
⇒ append ([1, 2], [3])    [1, 2, 3]
⇒ append ([1], [])        [1]
```
Rekursi dilakukan terhadap **L1** karena list hanya bisa dibongkar dan dibentuk dari depan. L2 bisa dipakai utuh sebagai ekor, sedangkan elemen L1 harus "dipasang ulang" satu per satu.

```
nth : Integer > 0, List → Element
{ elemen ke-n L, elemen pertama bernomor 1. Prasyarat: 1 ≤ n ≤ length (L) }
{ Basis: n = 1 → head (L)
  Rekurens: n > 1 → nth (n - 1, tail (L)) }
nth (n, L) :
  depend on n
    n = 1 : head (L)
    n > 1 : nth (n - 1, tail (L))

⇒ nth (1, [7, 8, 9])   7
⇒ nth (3, [7, 8, 9])   9
```
Untuk mencapai elemen terakhir dari list sepanjang 1000 dibutuhkan **1000 aplikasi**. Inilah akibat ketidaksimetrisan list.

#### Kasus 13: Memperbaiki `removeAdjacent`

Spesifikasi: `removeAdjacent (L)` adalah L tanpa elemen yang sama dengan elemen tepat sebelumnya, misalnya `[1, 1, 2] → [1, 2]`.

> **Catatan jujur:** realisasi yang tercetak di slide (kasus ketiga `removeAdjacent (tail (L))`) sebenarnya **sudah benar**. Sudah kucoba dengan `[1,1,1]`, `[1,2,2,1]`, dan `[1,1,1,2]`, dan semuanya menghasilkan jawaban yang benar. Pertanyaan no. 2 ("apa yang terjadi pada elemen yang baru menjadi elemen pertama sesudah rekurens") mengisyaratkan versi keliru yang **berbeda**, kemungkinan besar seperti di bawah ini. Tanyakan ke asisten/dosen versi mana yang dimaksud.

Versi keliru yang umum:
```
{ kasus ketiga keliru }
not (isEmpty (L)) and not (isOneElement (L)) and then (head (L) = head (tail (L))) :
    makeCons (head (L), removeAdjacent (tail (tail (L))))
```
1. Masukan salah: `removeAdjacent ([1, 1, 1])` menghasilkan `[1, 1]`, padahal seharusnya `[1]`.
2. Setelah dua elemen dilompati, elemen ketiga menjadi elemen pertama dari rekurens, tetapi **tidak pernah dibandingkan** dengan `head (L)` yang sudah dipasang.
3. Perbaikannya: rekursi atas `tail (L)` saja tanpa `makeCons`, supaya elemen kembar berikutnya tetap dibandingkan (inilah versi di slide).
4. Contoh aplikasi yang menampakkan bug: **tiga atau lebih elemen kembar berturut-turut**, misalnya `[1, 1, 1]` atau `[2, 2, 2, 3]`.

---

## 7. Cheat Sheet Akhir

### Template teks fungsi
```
JUDUL ...

DEFINISI DAN SPESIFIKASI
  f : Domain → Range
  { makna f (x) ... Prasyarat: ... }
  { Basis: ... → ...  Rekurens: ... → ... }     ← bila rekursif

REALISASI
  f (x) : ...

APLIKASI
  ⇒ f (nilai di dalam kasus)
  ⇒ f (nilai tepat di batas)
  ⇒ f (nilai ekstrem / basis)
```

### Template teks type
```
TYPE Nama
DEFINISI TYPE                     type Nama : ⟨komp1 : T1, komp2 : T2⟩   { + invarian }
DEFINISI DAN SPESIFIKASI SELEKTOR     komp1 : Nama → T1
DEFINISI DAN SPESIFIKASI KONSTRUKTOR  makeNama : T1, T2 → Nama   { Prasyarat }
DEFINISI DAN SPESIFIKASI PREDIKAT     isXxx : Nama → Boolean
DEFINISI DAN SPESIFIKASI OPERATOR LAIN
REALISASI                          { hanya predikat & operator lain }
APLIKASI
```

### Checklist sebelum mengumpulkan jawaban
- [ ] Nama sesuai konvensi (`is`, `make`, `to`, camelCase)?
- [ ] Domain & range tepat, termasuk pembatasan (`Integer ≥ 0`, `[1..12]`, `List tidak kosong`)?
- [ ] Spesifikasi menyatakan **arti**, bukan mengulang rumus?
- [ ] Prasyarat ditulis bila ada `div`, `mod`, `/`, `sqrt`, `head`, `tail`?
- [ ] Kasus **lengkap** dan **disjoint**? Nilai batas punya **tepat satu** pemilik?
- [ ] Tidak ada `if` tanpa `else`? `else` tidak menutupi lubang?
- [ ] `and then` / `or else` dipakai bila operan kedua mungkin tidak terdefinisi?
- [ ] `not (...)` selalu dengan kurung? Tidak mencampur Integer & Real tanpa `asReal`?
- [ ] Rekursif: ada **basis**, dan besaran yang **mengecil + batasnya** disebutkan?
- [ ] Basis cukup (mundur k langkah → k basis)? Nilai basis **netral**?
- [ ] List: basis-0 atau basis-1 sudah benar? Rekursi atas `tail (L)`, bukan `L`?
- [ ] Invarian type dijaga di konstruktor **dan** di setiap operator yang menghasilkan nilai baru?
- [ ] Aplikasi mencakup kasus kosong/batas, satu di atas batas, dan kasus umum?

### Istilah kunci (definisi satu kalimat)
| Istilah | Arti |
|---|---|
| Transparansi referensial | Ekspresi dapat diganti oleh nilainya di mana pun dan kapan pun |
| Ekspansi | Aplikasi fungsi diganti realisasinya |
| Reduksi | Operator/fungsi dasar dihitung |
| Prasyarat | Syarat masukan agar fungsi terdefinisi (bagian dari kontrak) |
| Nama antara | Nama lokal lewat `let`, hanya hidup di satu realisasi |
| Lengkap | Setiap nilai domain memenuhi ≥ 1 kondisi |
| Disjoint | Tidak ada nilai yang memenuhi > 1 kondisi |
| Invarian | Sifat yang berlaku bagi setiap nilai type sepanjang hidupnya |
| Type produk | Satu nilai berisi beberapa komponen sekaligus (tuple) |
| Type alternatif | Nilai adalah salah satu dari beberapa bentuk (`\|`) |
| Basis | Kasus yang jawabannya langsung diketahui, tanpa rekursi |
| Konvergensi | Argumen rekursif makin dekat ke basis |
| Akumulator | Parameter yang membawa hasil sementara maju |
| Basis-0 / Basis-1 | Basis pada list kosong / list satu elemen |

Semangat kuisnya! 🚀
