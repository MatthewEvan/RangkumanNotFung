# Mesin Karakter dan Mesin Kata dalam C

IF2110 Algoritma dan Pemrograman 2 · persiapan praktikum

Semua kode di dokumen ini ditulis dalam **C** dengan nama dari header praktikum (`currentChar`, `START`, `STARTWORD`, `currentWord.TabWord`, dan seterusnya), bukan notasi algoritmik. Semua kode sudah dikompilasi dengan `gcc -Wall` dan dijalankan dengan contoh soal dari arsip praktikum PTPB25 (Praktikum 8).

Isi:

1. [Cheatsheet nama](#1-cheatsheet-nama)
2. [Mesin karakter](#2-mesin-karakter)
3. [Mesin kata](#3-mesin-kata)
4. [Perbandingan mesin karakter dan mesin kata](#4-perbandingan-mesin-karakter-dan-mesin-kata)
5. [Versi akuisisi: slide vs header praktikum](#5-versi-akuisisi-slide-vs-header-praktikum)
6. [Kompilasi dan testing](#6-kompilasi-dan-testing)
7. [Latihan soal](#7-latihan-soal)
8. [Pembahasan](#8-pembahasan)
9. [Checklist sebelum submit](#9-checklist-sebelum-submit)

---

## 1. Cheatsheet nama

| Konsep | Header praktikum (dipakai di sini) | Slide 09x02 / 12x01 |
|---|---|---|
| karakter di jendela | `currentChar` | `cc` |
| lampu akhir pita | `EOP` | `eop` |
| mulai baca pita | `START()` | `start()` |
| maju satu karakter | `ADV()` | `adv()` |
| penanda akhir pita | `MARK` (`'.'`) | `MARK` |
| akhir akuisisi kata | `EndWord` | `endWord` |
| kata yang sedang dibaca | `currentWord` | `currentWord` |
| isi kata | `currentWord.TabWord[i]` | `currentWord.buffer[i]` |
| panjang kata | `currentWord.Length` | `currentWord.length` |
| kapasitas kata | `NMax` (50) | `NMAX` |
| mulai baca kata | `STARTWORD()` | `startWord()` |
| maju satu kata | `ADVWORD()` | `advWord()` |
| lewati blank | `IgnoreBlanks()` | `ignoreBlanks()` |
| salin satu kata | `CopyWord()` | `copyWord()` |

**Dua pola yang harus hafal:**

```c
/* Pola mesin karakter */
START();
while (!EOP) {              /* sama dengan currentChar != MARK */
    /* proses currentChar */
    ADV();
}

/* Pola mesin kata */
STARTWORD();
while (!EndWord) {
    /* proses currentWord */
    ADVWORD();
}
```

> **Penting:** header yang diberikan asisten bisa berbeda dari arsip tahun lalu. Baca `.h` yang dibagikan dulu, lalu sesuaikan nama.

---

## 2. Mesin karakter

### 2.1 Konsep

Mesin karakter memodelkan sebuah **pita** berisi deretan karakter yang diakhiri `MARK` (titik). Mesin hanya bisa melihat **satu karakter** dalam satu waktu lewat **jendela**, yaitu `currentChar`. Lampu `EOP` (End Of Pita) menyala saat `currentChar == MARK`.

```
Pita:     I T B _ A D A .        (_ = spasi)
          ^
          currentChar = 'I', EOP = false
```

**State** mesin = `currentChar` dan `EOP`. Pemakai boleh **membaca** state ini tapi tidak boleh mengubahnya langsung. Pemakai hanya boleh memanggil dua aksi primitif:

| Primitif | I.S. (janji pemanggil) | F.S. (janji realisasi) |
|---|---|---|
| `START()` | sembarang | `currentChar` = karakter pertama pita; `EOP` true jika itu `MARK` |
| `ADV()` | `currentChar != MARK` | `currentChar` = karakter berikutnya; `EOP` true jika itu `MARK` |

Mesin karakter **tidak bisa mundur** dan **tidak menyimpan** karakter sebelumnya. Kalau kamu butuh karakter sebelumnya, simpan sendiri di variabel (lihat latihan K3).

### 2.2 Header (`mesinkarakter.h`)

```c
#ifndef __MESIN_KAR_H_
#define __MESIN_KAR_H_

#include "boolean.h"

#define MARK '.'

/* State Mesin */
extern char currentChar;
extern boolean EOP;

void START();
void ADV();

#endif
```

Hal yang perlu dipahami dari header ini:

- **`extern`** artinya: variabel ini **dideklarasikan** di sini, tapi **didefinisikan** (diberi memori) di file `.c`. Tanpa `extern`, setiap file yang meng-include header akan membuat variabel sendiri dan linker protes *multiple definition*.
- **Include guard** (`#ifndef ... #define ... #endif`) mencegah header terbaca dua kali. Ini penting karena `mesinkata.h` juga meng-include `mesinkarakter.h`.
- `boolean.h` di praktikum biasanya mendefinisikan `boolean`, `true`, `false` sebagai makro.

### 2.3 Realisasi (`mesinkarakter.c`)

Di praktikum, pita dibaca dari **stdin** dengan `getchar()`.

```c
#include <stdio.h>
#include "mesinkarakter.h"

char currentChar;      /* definisi state (pasangan dari extern di .h) */
boolean EOP;

static int retval;     /* static: hanya terlihat di file ini */

void START() {
    retval = getchar();
    currentChar = (char) retval;
    EOP = (currentChar == MARK);
}

void ADV() {
    retval = getchar();
    currentChar = (char) retval;
    EOP = (currentChar == MARK);
}
```

Penjelasan:

- `START()` dan `ADV()` di sini isinya sama, karena stdin sudah "terbuka". Bedanya ada di kontrak: `START()` boleh dipanggil kapan saja, `ADV()` tidak boleh dipanggil saat `EOP` sudah true.
- `EOP = (currentChar == MARK);` lebih aman daripada `if (currentChar == MARK) EOP = true;` saja, karena versi `if` tidak pernah mengembalikan `EOP` ke false.
- Kata kunci `static` pada variabel global membuat variabel itu privat milik file. Pemakai mesin tidak bisa menyentuhnya. Ini cara C menyembunyikan detail realisasi.

### 2.4 Contoh program pemakai

```c
#include <stdio.h>
#include "mesinkarakter.h"

int main(void) {
    int nA = 0;
    START();
    while (!EOP) {
        if (currentChar == 'A') nA++;
        ADV();
    }
    printf("%d\n", nA);
    return 0;
}
```

Input `AKU ANAK ITB.` menghasilkan `3`.

### 2.5 Jebakan mesin karakter di C

| Jebakan | Akibat | Cara aman |
|---|---|---|
| Memanggil `ADV()` saat `EOP` true | membaca melewati titik (input berikutnya) | selalu cek `!EOP` sebelum `ADV()` |
| Input tidak punya titik | `getchar()` mengembalikan `EOF` (-1), tidak pernah sama dengan `MARK`, loop tidak berhenti | pastikan input uji diakhiri `.`; soal praktikum menjamin ini |
| Memproses `currentChar` sesudah `ADV()` di dalam loop | karakter pertama terlewat, `MARK` ikut diproses | urutan: **proses dulu, baru `ADV()`** |
| Lupa `START()` | `currentChar` berisi nilai sampah | `START()` sekali di awal |
| Mendefinisikan `char currentChar;` di `.h` | *multiple definition* saat linking | di `.h` pakai `extern`, definisi di `.c` |
| Menganggap `'\n'` sebagai blank | `BLANK` hanya `' '`, jadi newline ikut menjadi bagian kata | perhatikan format input soal |

---

## 3. Mesin kata

### 3.1 Konsep

**Kata** adalah deretan karakter yang **bukan blank**, dipisahkan satu atau lebih blank. Mesin kata membaca pita **satu kata per langkah**, bukan satu karakter. Mesin kata dibangun **di atas** mesin karakter: di dalamnya ia hanya memanggil `START()`, `ADV()`, dan membaca `currentChar`.

```
Pita:  _ _ I T B _ _ _ A D A _ D I _ _ .
Kata:      [ITB]       [ADA] [DI]
```

Blank di awal, blank ganda di tengah, dan blank sebelum titik semuanya diabaikan. Pita di atas berisi tepat tiga kata.

**State** mesin kata:

- `currentWord`: kata yang baru saja diakuisisi (tipe `Word`).
- `EndWord`: true jika sudah tidak ada kata lagi.

### 3.2 Header (`mesinkata.h`)

```c
#ifndef __MESINKATA_H__
#define __MESINKATA_H__

#include "boolean.h"
#include "mesinkarakter.h"

#define NMax 50
#define BLANK ' '

typedef struct {
    char TabWord[NMax];   /* indeks 0..NMax-1 */
    int Length;
} Word;

/* State Mesin Kata */
extern boolean EndWord;
extern Word currentWord;

void IgnoreBlanks();
/* I.S. currentChar sembarang
   F.S. currentChar != BLANK atau currentChar = MARK */

void STARTWORD();
/* I.S. currentChar sembarang
   F.S. EndWord = true dan currentChar = MARK;
        atau EndWord = false, currentWord = kata pertama */

void ADVWORD();
/* I.S. currentChar adalah karakter pertama kata yang akan diakuisisi (atau MARK)
   F.S. currentWord = kata berikutnya, currentChar = karakter pertama
        kata sesudahnya (mungkin MARK). Jika tidak ada kata lagi, EndWord = true */

void CopyWord();
/* I.S. currentChar adalah karakter pertama kata
   F.S. currentWord berisi kata itu; currentChar = BLANK atau MARK.
        Jika panjang kata > NMax, sisa kata dipotong */

#endif
```

**`Word` bukan string C.** `TabWord` tidak diakhiri `'\0'`. Jangan pakai `printf("%s", currentWord.TabWord)` atau `strcmp`. Selalu pakai `Length`.

### 3.3 Realisasi (`mesinkata.c`)

```c
#include <stdio.h>
#include "mesinkata.h"

boolean EndWord;
Word currentWord;

void IgnoreBlanks() {
    while (currentChar == BLANK) {
        ADV();
    }
}

void STARTWORD() {
    START();
    IgnoreBlanks();
    if (currentChar == MARK) {
        EndWord = true;
    } else {
        EndWord = false;
        CopyWord();
        IgnoreBlanks();          /* currentChar = huruf pertama kata berikutnya, atau MARK */
    }
}

void ADVWORD() {
    if (currentChar == MARK) {
        EndWord = true;          /* currentWord tidak diubah */
    } else {
        CopyWord();
        IgnoreBlanks();
    }
}

void CopyWord() {
    int i = 0;
    while (currentChar != MARK && currentChar != BLANK) {
        if (i < NMax) {
            currentWord.TabWord[i] = currentChar;
            i++;
        }
        ADV();                   /* tetap maju walau buffer penuh */
    }
    currentWord.Length = i;
}
```

Penjelasan tiap primitif:

- **`IgnoreBlanks`**: majukan pita selama `currentChar` blank. Aman dipanggil saat `currentChar == MARK` karena `MARK` bukan blank, jadi loop tidak jalan.
- **`STARTWORD`**: buka pita, buang blank awal. Kalau langsung ketemu titik, pita tidak berisi kata. Kalau tidak, salin kata pertama, lalu buang blank sesudahnya supaya `currentChar` siap di huruf pertama kata kedua. Ini memenuhi I.S. `ADVWORD` di header praktikum.
- **`ADVWORD`**: kalau `currentChar` sudah `MARK`, semua kata sudah dibaca, nyalakan `EndWord`. Kalau belum, salin kata ini lalu buang blank sesudahnya.
- **`CopyWord`**: salin huruf sampai ketemu blank atau titik. Kalau kata lebih dari `NMax` huruf, huruf ke-51 dan seterusnya **dilewati** (tetap `ADV()`, tapi tidak disimpan), jadi kata berikutnya tetap terbaca benar.

### 3.4 Menelusuri pita `__ITB___ADA_DI__.`

(`_` = spasi)

| Setelah | `currentWord` | `currentChar` | `EndWord` |
|---|---|---|---|
| `STARTWORD()` | `ITB` (3) | `'A'` | false |
| `ADVWORD()` ke-1 | `ADA` (3) | `'D'` | false |
| `ADVWORD()` ke-2 | `DI` (2) | `'.'` | false |
| `ADVWORD()` ke-3 | `DI` (tidak berubah) | `'.'` | **true** |

Perhatikan baris ketiga: sesudah kata terakhir dibaca, `currentChar` sudah titik tapi `EndWord` **masih false**. Itu sebabnya pola `while (!EndWord) { proses; ADVWORD(); }` memproses `DI` dulu, baru `ADVWORD()` berikutnya menyalakan `EndWord`.

Hasil uji implementasi di atas:

| Input | Kata yang terbaca |
|---|---|
| `.` | tidak ada |
| `   .` | tidak ada |
| `A.` | `A` |
| `  ITB   ADA DI  .` | `ITB`, `ADA`, `DI` |
| `hello world abc def .` | `hello`, `world`, `abc`, `def` |
| `example test case.` | `example`, `test`, `case` |
| 55 huruf A, spasi, `B.` | `AAA...A` (Length 50), `B` |

### 3.5 Helper yang sering dibutuhkan

Simpan fungsi-fungsi ini, hampir selalu terpakai di soal mesin kata.

```c
/* Cetak Word (karena Word bukan string C) */
void printWord(Word w) {
    int i;
    for (i = 0; i < w.Length; i++) putchar(w.TabWord[i]);
}

/* Bandingkan dua Word */
boolean isWordEqual(Word a, Word b) {
    int i;
    if (a.Length != b.Length) return false;
    for (i = 0; i < a.Length; i++)
        if (a.TabWord[i] != b.TabWord[i]) return false;
    return true;
}

/* Bandingkan Word dengan string C, misal "KOMA" */
boolean IsKataSama(Word K1, char *str) {
    int i = 0;
    while (i < K1.Length) {
        if (str[i] == '\0' || K1.TabWord[i] != str[i]) return false;
        i++;
    }
    return str[i] == '\0';   /* str juga harus habis tepat di sini */
}

/* Ubah Word berisi digit menjadi int, "123" -> 123 */
int wordToInt(Word w) {
    int i, val = 0;
    for (i = 0; i < w.Length; i++) val = val * 10 + (w.TabWord[i] - '0');
    return val;
}
```

Kenapa `IsKataSama` mengecek `str[i] == '\0'` di dalam loop dan di akhir? Supaya `"KOM"` vs `"KOMA"` dan `"KOMAA"` vs `"KOMA"` sama-sama mengembalikan false. Tanpa pengecekan di akhir, kata yang lebih pendek dari string akan dianggap sama.

`Word` adalah struct, jadi **bisa di-assign** langsung: `Word simpan = currentWord;` menyalin seluruh isi `TabWord` dan `Length`. Ini cara menyimpan kata sebelum `ADVWORD()` menimpanya.

---

## 4. Perbandingan mesin karakter dan mesin kata

| Aspek | Mesin karakter | Mesin kata |
|---|---|---|
| Satuan yang dibaca | 1 karakter | 1 kata (deret non-blank) |
| State | `currentChar`, `EOP` | `currentWord`, `EndWord` |
| Primitif pemakai | `START`, `ADV` | `STARTWORD`, `ADVWORD` |
| Tipe state | `char` | `Word` (struct: `TabWord` + `Length`) |
| Kapan selesai | `EOP` true = `currentChar` adalah `MARK` | `EndWord` true = tidak ada kata lagi |
| Blank | karakter biasa, ikut diproses | pemisah, otomatis dibuang |
| Hubungan | berdiri sendiri, membaca stdin | dibangun di atas mesin karakter |
| Memori | 1 karakter | 1 kata (maks `NMax` huruf) |
| State bawah masih terlihat? | tidak ada lapisan bawah | ya, `currentChar` tetap bisa dibaca |
| Cocok untuk | hitung huruf, pola antarhuruf (`"LE"`) | hitung kata, bandingkan kata, parsing token |

**Persamaannya:** keduanya mesin abstrak (state + primitif + kontrak), keduanya hanya maju, keduanya memakai pola loop yang sama (mulai, selama belum selesai: proses lalu maju).

### Satu masalah, dua mesin: menghitung kata

Dengan **mesin karakter saja**, kamu harus mengurus blank sendiri:

```c
int hitungKataPakaiKarakter() {
    int nWord = 0;
    START();
    while (!EOP) {
        while (!EOP && currentChar == ' ') ADV();     /* lewati blank */
        if (!EOP) {
            nWord++;
            while (!EOP && currentChar != ' ') ADV(); /* lewati huruf kata */
        }
    }
    return nWord;
}
```

Dengan **mesin kata**, urusan blank sudah disembunyikan:

```c
int hitungKataPakaiKata() {
    int nWord = 0;
    STARTWORD();
    while (!EndWord) {
        nWord++;
        ADVWORD();
    }
    return nWord;
}
```

Keduanya menghasilkan 3 untuk `  ITB   ADA DI  .` dan 0 untuk `   .`. Versi mesin karakter penuh dengan `!EOP &&` karena setiap loop dalam bisa mencapai titik. Inilah alasan mesin kata dibuat: logika "lewati blank, salin huruf" ditulis **sekali** di `mesinkata.c`, lalu semua program pemakai cukup memakai `STARTWORD`/`ADVWORD`.

---

## 5. Versi akuisisi: slide vs header praktikum

Ada dua cara meletakkan `currentChar` sesudah sebuah kata dibaca. Keduanya benar asal **konsisten** antara `STARTWORD`, `ADVWORD`, dan I.S. di header.

| | Slide 12x01 | Header praktikum ("Versi I") |
|---|---|---|
| Sesudah kata dibaca, `currentChar` ada di | blank sesudah kata, atau `MARK` | huruf pertama kata berikutnya, atau `MARK` |
| I.S. `ADVWORD` | `currentChar` = BLANK atau MARK | `currentChar` = huruf pertama kata (atau MARK) |
| Isi `ADVWORD` | `IgnoreBlanks; if MARK EndWord=true else CopyWord` | `if MARK EndWord=true else { CopyWord; IgnoreBlanks; }` |
| Pola pemakai | sama | sama |

Implementasi di Bagian 3.3 mengikuti header praktikum.

### Peringatan: jangan salin `mesinkata.c` dari arsip mentah-mentah

`mesinkata.c` di arsip PTPB25 (folder Pra Praktikum 8) memakai kombinasi berikut:

```c
/* VERSI ARSIP, BERMASALAH */
void STARTWORD() { START(); IgnoreBlanks();
                   if (currentChar == MARK) EndWord = true;
                   else { EndWord = false; CopyWord(); } }        /* tanpa IgnoreBlanks sesudahnya */
void ADVWORD()   { CopyWord(); IgnoreBlanks();
                   if (currentChar == MARK) EndWord = true; }     /* CopyWord sebelum cek MARK */
```

Saya tulis ulang logika itu dan menjalankannya dengan driver yang sama:

| Input | Yang terbaca |
|---|---|
| `mom dad level.` | `mom`, **kata kosong**, `dad` (kata `level` **hilang**) |
| `hello world abc def .` | `hello`, **kata kosong**, `world`, `abc` (kata `def` **hilang**) |

Dua masalahnya:

1. Sesudah `STARTWORD`, `currentChar` masih di blank. `ADVWORD` pertama langsung `CopyWord()`, yang berhenti seketika karena ketemu blank, jadi menghasilkan kata dengan `Length` 0.
2. `ADVWORD` menyalakan `EndWord` begitu kata terakhir selesai disalin. Akibatnya loop berhenti **sebelum** kata terakhir diproses.

Arsip tersebut tetap mendapat nilai penuh di Olympia, kemungkinan karena grader memakai implementasi mesin kata miliknya sendiri. Jangan diandalkan. Kalau hari ini diminta mengumpulkan `mesinkata.c`, pakai versi di Bagian 3.3 dan uji dengan tabel di 3.4.

---

## 6. Kompilasi dan testing

Struktur folder:

```
boolean.h
mesinkarakter.h   mesinkarakter.c
mesinkata.h       mesinkata.c
reversewords.c    (atau file soal lain, berisi main)
```

Kompilasi, **semua `.c` ikut**:

```bash
gcc -Wall -o reversewords reversewords.c mesinkata.c mesinkarakter.c
```

Program yang hanya memakai mesin karakter tidak perlu `mesinkata.c`:

```bash
gcc -Wall -o hitung hitung.c mesinkarakter.c
```

Menjalankan dengan input:

```bash
echo "hello world abc def ." | ./reversewords     # lewat pipe
./reversewords < input.txt                        # dari file
./reversewords                                    # ketik manual, akhiri dengan titik lalu Enter
```

Newline sesudah titik tidak masalah karena pembacaan berhenti di `MARK`.

Driver sederhana untuk menguji mesin kata (sangat berguna untuk Pra Praktikum):

```c
#include <stdio.h>
#include "mesinkata.h"

int main(void) {
    int i;
    STARTWORD();
    while (!EndWord) {
        printf("[");
        for (i = 0; i < currentWord.Length; i++) putchar(currentWord.TabWord[i]);
        printf("] len=%d cc='%c'\n", currentWord.Length, currentChar);
        ADVWORD();
    }
    printf("END cc='%c'\n", currentChar);
    return 0;
}
```

Untuk `  ITB   ADA DI  .` keluarannya:

```
[ITB] len=3 cc='A'
[ADA] len=3 cc='D'
[DI] len=2 cc='.'
END cc='.'
```

Kalau kamu melihat `[]` (kata kosong) atau kata terakhir hilang, mesin katamu salah.

Error yang sering muncul:

| Pesan | Penyebab |
|---|---|
| `undefined reference to 'ADV'` | lupa menyertakan `mesinkarakter.c` saat kompilasi |
| `multiple definition of 'currentChar'` | variabel didefinisikan di `.h` tanpa `extern`, atau didefinisikan di dua `.c` |
| `multiple definition of 'main'` | file solusi dan driver sama-sama punya `main` |
| program diam terus | input tidak diakhiri titik, atau `ADV()` dipanggil di loop yang tidak pernah maju |

---

## 7. Latihan soal

Soal 1 sampai 5 berasal dari arsip PTPB25, folder `Semester 3/IF2110/Praktikum/Praktikum 8` (Pra Praktikum dan Post Praktikum 8, IF2110 2025/2026). Soal K dan W adalah latihan tambahan dari slide untuk melatih pola dasar. Coba kerjakan dulu, pembahasan ada di Bagian 8.

### Soal 1 · Implementasi mesin (Pra Praktikum 8)

Diberikan `mesinkarakter.h` dan `mesinkata.h` (Bagian 2.2 dan 3.2). Buat `mesinkarakter.c` dan `mesinkata.c`. Pita dibaca dari stdin. Buat driver sendiri untuk menguji setiap fungsi.

### Soal 2 · `reversewords.c` (Post Praktikum 8, Q1)

Diberikan sekuens n kata (1 ≤ n ≤ 1000) dipisahkan satu atau lebih whitespace, diakhiri `.` yang bukan bagian kata. Balik huruf setiap kata, urutan kata tetap. Cetak satu kata per baris, setiap baris diakhiri `\n`.

| Masukan | Keluaran |
|---|---|
| `hello world abc def .` | `olleh` / `dlrow` / `cba` / `fed` |
| `example test case.` | `elpmaxe` / `tset` / `esac` |

### Soal 3 · `palindrome.c` (Post Praktikum 8, Q2)

Lengkapi:

```c
boolean isPalindrome(Word w);   /* case-sensitive; kata kosong dan 1 huruf = palindrom */
int countPalindromes();         /* jumlah kata palindrom pada pita */
```

| Masukan | Keluaran |
|---|---|
| `mom dad hello level world.` | 3 |
| `hello world test.` | 0 |
| `a b c d.` | 4 |

### Soal 4 · `telegram.c` (Post Praktikum 8, Q4)

Baca telegram dengan mesin kata. Tanda baca ditulis sebagai kata `KOMA` (`,`) dan `TITIK` (`.`).

- Kata dipisahkan satu atau lebih spasi; pita diakhiri `.`.
- Tanda baca tidak berbayar.
- Kata reguler (panjang < 10) Rp1000, kata panjang (≥ 10) Rp1500.
- Jika total kata **termasuk tanda baca** > 10, diskon 10%.
- Tanda baca menempel pada kata sebelumnya dan dipisah satu spasi dengan kata berikutnya.

Keluaran: telegram hasil konversi, jumlah kata reguler, jumlah kata panjang, total biaya, masing-masing satu baris.

| Masukan | Keluaran |
|---|---|
| `SEGERA PULANG KOMA ADIK SAKIT TITIK.` | `SEGERA PULANG, ADIK SAKIT.` / `4` / `0` / `4000` |
| `APA KABAR TITIK SAYA SENANG MENERIMA TAWARANNYA KOMA SEGERA KIRIM PROPOSAL KE KANTOR BANDUNG TITIK.` | `APA KABAR. SAYA SENANG MENERIMA TAWARANNYA, SEGERA KIRIM PROPOSAL KE KANTOR BANDUNG.` / `11` / `1` / `11250` |
| `TITIK KOMA A KOMA TITIK TITIK KOMA KOMA TITIK TITIK TITIK.` | `., A,..,,...` / `1` / `0` / `900` |

### Soal 5 · `parser.c` (Post Praktikum 8, Q3)

Periksa apakah kalimat valid terhadap grammar:

```
<Kalimat>  ::= <Subjek> <Predikat> <Objek> { , <Objek> }*
<Subjek>   ∈ { aku, kamu, dia }
<Predikat> ∈ { beli, ambil }
<Objek>    = satu kata alfabet (A-Z, a-z)
```

Lengkapi `int cfg_parser()` yang memanggil `STARTWORD()` sendiri dan mengembalikan 1 (valid) atau 0. Koma bisa menempel ke objek (`buku,`) atau berdiri sendiri (`,`). Skeleton menyediakan `isSubjek`/`isPredikat` (case-insensitive) dan enum state `S_EXPECT_SUBJECT`, `S_EXPECT_PREDICATE`, `S_EXPECT_FIRST_OBJECT`, `S_EXPECT_SEPARATOR_OR_END`, `S_EXPECT_OBJECT_AFTER_COMMA`, `S_INVALID`.

| Masukan | Keluaran |
|---|---|
| `aku beli buku.` | 1 |
| `aku beli buku, pensil, penggaris.` | 1 |
| `dia ambil tas , buku.` | 1 |
| `Dia ambil tas        , buku, penggariss              .` | 1 |
| `aku beli .` | 0 (tidak ada objek) |
| `beli buku.` | 0 (tidak ada subjek) |
| `aku beli , pensil.` | 0 (koma langsung sesudah predikat) |
| `aku beli buku, .` | 0 (koma jadi token terakhir) |

Pertanyaan tambahan: solusi di arsip mendapat 66/100 dengan verdict *Time limit exceeded* dan *Wrong answer*. Temukan penyebabnya (kodenya ada di Pembahasan 5).

### Latihan tambahan mesin karakter (hanya `START`, `ADV`, `currentChar`, `EOP`)

**K1.** Hitung jumlah karakter (tanpa `MARK`) dan jumlah huruf `'A'`. `AKU ANAK ITB.` → `12 3`.

**K2.** Hitung jumlah kata **tanpa** mesin kata. `  ITB   ADA DI  .` → 3, `   .` → 0.

**K3.** Hitung banyak pasangan `'L'` yang langsung diikuti `'E'`. `.` → 0, `LE.` → 1, `ELLE.` → 1, `LLE LE EL.` → 2, `L E.` → 0.

### Latihan tambahan mesin kata

**W1.** Rata-rata panjang kata, dua angka di belakang koma. `ITB ADA DI.` → `2.67`. Pita tanpa kata → `Pita tidak mengandung kata`.

**W2.** Cetak kata terpanjang (yang pertama jika seri) dan panjangnya. `ITB ADA DI BANDUNG.` → `BANDUNG 7`.

**W3.** Pita berisi bilangan bulat non-negatif. Cetak jumlahnya. `12 30   8 100.` → `150`.

**W4.** Berapa kali kata pertama muncul di pita? `DI ITB ADA DI DI.` → 3.

---

## 8. Pembahasan

### Pembahasan 1 · Implementasi mesin

Jawabannya adalah kode di **Bagian 2.3** (`mesinkarakter.c`) dan **Bagian 3.3** (`mesinkata.c`). Poin yang biasanya dinilai:

- Variabel state **didefinisikan** di `.c` (`char currentChar; boolean EOP;`, `boolean EndWord; Word currentWord;`), pasangan dari `extern` di `.h`.
- `EOP` di-set ulang di setiap `START()`/`ADV()`, bukan hanya dinyalakan.
- `CopyWord` tetap `ADV()` walau buffer penuh, dan `Length` maksimal `NMax`.
- `STARTWORD` dan `ADVWORD` konsisten soal posisi `currentChar` (lihat Bagian 5). Uji dengan driver di Bagian 6 dan semua baris tabel 3.4: pita kosong, hanya blank, satu huruf, blank berlebih, kata lebih dari `NMax`.

### Pembahasan 2 · `reversewords.c`

```c
#include <stdio.h>
#include "mesinkata.h"

int main(void) {
    int i;
    STARTWORD();
    while (!EndWord) {
        for (i = currentWord.Length - 1; i >= 0; i--) {
            putchar(currentWord.TabWord[i]);
        }
        putchar('\n');
        ADVWORD();
    }
    return 0;
}
```

Membalik kata cukup dengan mencetak `TabWord` dari indeks `Length - 1` turun ke 0. Tidak perlu array tambahan. Mesin kata sudah membuang blank ganda dan blank sebelum titik (contoh 1 punya spasi sebelum `.`), jadi tidak ada baris kosong. `putchar('\n')` di setiap kata memenuhi syarat "setiap output diakhiri endline".

### Pembahasan 3 · `palindrome.c`

```c
boolean isPalindrome(Word w) {
    int l = 0, r = w.Length - 1;
    while (l < r) {
        if (w.TabWord[l] != w.TabWord[r]) return false;
        l++;
        r--;
    }
    return true;
}

int countPalindromes() {
    int cnt = 0;
    STARTWORD();
    while (!EndWord) {
        if (isPalindrome(currentWord)) cnt++;
        ADVWORD();
    }
    return cnt;
}
```

Teknik **dua pointer**: `l` dari kiri, `r` dari kanan, bandingkan lalu bergerak ke tengah. Kondisi `l < r` cukup karena huruf tengah (pada kata ganjil) tidak perlu dibandingkan dengan dirinya sendiri. Kasus khusus:

- kata kosong: `r = -1`, loop tidak jalan, `true`;
- kata 1 huruf: `l = r = 0`, loop tidak jalan, `true`;
- `Mom` → `false` karena case-sensitive (`'M' != 'm'`).

Hasil uji: contoh soal 3, 0, 4; `abba abcba ab.` → 2.

Perhatikan: di soal ini `countPalindromes()` yang memanggil `STARTWORD()`. Kalau `main` di grader juga memanggil `STARTWORD()`, pita terbaca dua kali. Ikuti komentar I.S. di skeleton.

### Pembahasan 4 · `telegram.c`

```c
boolean IsKataSama(Word K1, char *str) {
    int i = 0;
    while (i < K1.Length) {
        if (str[i] == '\0' || K1.TabWord[i] != str[i]) return false;
        i++;
    }
    return str[i] == '\0';
}

void processTelegram() {
    int countRegular = 0, countLong = 0, countTotal = 0, totalCost, i;
    boolean firstWord = true;

    STARTWORD();
    while (!EndWord) {
        countTotal++;                              /* tanda baca ikut dihitung untuk diskon */
        if (IsKataSama(currentWord, "KOMA")) {
            putchar(',');                          /* menempel, tanpa spasi */
        } else if (IsKataSama(currentWord, "TITIK")) {
            putchar('.');
        } else {
            if (currentWord.Length < 10) countRegular++;
            else countLong++;
            if (!firstWord) putchar(' ');          /* spasi SEBELUM kata, kecuali kata pertama */
            for (i = 0; i < currentWord.Length; i++) putchar(currentWord.TabWord[i]);
        }
        firstWord = false;
        ADVWORD();
    }
    totalCost = countRegular * 1000 + countLong * 1500;
    if (countTotal > 10) totalCost = totalCost * 9 / 10;
    printf("\n%d\n%d\n%d\n", countRegular, countLong, totalCost);
}
```

Kunci soal ini adalah **kapan mencetak spasi**. Trik: cetak spasi **sebelum** kata biasa (bukan sesudah), dan tidak pernah sebelum tanda baca. Dengan begitu tanda baca otomatis menempel ke kata sebelumnya, dan tidak ada spasi berlebih di akhir baris.

`firstWord` di-set false sesudah token apa pun, termasuk tanda baca. Itu sebabnya contoh 3 menghasilkan `., A` (ada spasi sebelum `A` karena sudah ada token sebelumnya).

Hitungan contoh 2: 11 kata reguler × 1000 + 1 kata panjang (`TAWARANNYA`, 10 huruf) × 1500 = 12500. Total token 14 > 10, jadi 12500 × 9 / 10 = 11250. Biaya selalu kelipatan 500, jadi `* 9 / 10` dan `/ 10 * 9` memberi hasil yang sama.

Hasil uji: ketiga contoh cocok. Pita `.` menghasilkan baris kosong lalu `0`, `0`, `0`.

### Pembahasan 5 · `parser.c`

Ide: **FSM** (finite state machine). Setiap kata yang dibaca memindahkan state. Kalimat valid jika di akhir pita state berada di `S_EXPECT_SEPARATOR_OR_END` (baru saja membaca objek tanpa koma).

```
SUBJECT --subjek--> PREDICATE --predikat--> FIRST_OBJECT
FIRST_OBJECT / OBJECT_AFTER_COMMA --objek--> SEPARATOR_OR_END
FIRST_OBJECT / OBJECT_AFTER_COMMA --objek,--> OBJECT_AFTER_COMMA
SEPARATOR_OR_END --","--> OBJECT_AFTER_COMMA
token lain di state mana pun --> INVALID (tetap INVALID sampai akhir)
```

```c
#include <stdio.h>
#include <string.h>
#include "mesinkata.h"

/* enum State, eqWordCStrInsensitive, isSubjek, isPredikat: dari skeleton */

boolean isAlpha(char c) {
    return ('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z');
}

boolean isCommaToken(Word w) {
    return w.Length == 1 && w.TabWord[0] == ',';
}

/* true jika w objek sah: huruf semua, boleh diakhiri satu koma menempel */
boolean isObjToken(Word w, int *hasTrailingComma) {
    int n = w.Length, i;
    *hasTrailingComma = 0;
    if (n > 0 && w.TabWord[n - 1] == ',') {
        *hasTrailingComma = 1;
        n--;                                  /* koma tidak ikut dicek */
    }
    if (n == 0) return false;                 /* token "," bukan objek */
    for (i = 0; i < n; i++) {
        if (!isAlpha(w.TabWord[i])) return false;
    }
    return true;
}

int cfg_parser() {
    State state = S_EXPECT_SUBJECT;
    int nObj = 0, tc;

    STARTWORD();
    while (!EndWord) {
        switch (state) {
        case S_EXPECT_SUBJECT:
            state = isSubjek(currentWord) ? S_EXPECT_PREDICATE : S_INVALID;
            break;
        case S_EXPECT_PREDICATE:
            state = isPredikat(currentWord) ? S_EXPECT_FIRST_OBJECT : S_INVALID;
            break;
        case S_EXPECT_FIRST_OBJECT:
        case S_EXPECT_OBJECT_AFTER_COMMA:
            if (isObjToken(currentWord, &tc)) {
                nObj++;
                state = tc ? S_EXPECT_OBJECT_AFTER_COMMA : S_EXPECT_SEPARATOR_OR_END;
            } else {
                state = S_INVALID;
            }
            break;
        case S_EXPECT_SEPARATOR_OR_END:
            state = isCommaToken(currentWord) ? S_EXPECT_OBJECT_AFTER_COMMA : S_INVALID;
            break;
        case S_INVALID:
            break;                            /* tetap invalid, tapi pita tetap dibaca */
        }
        ADVWORD();                            /* WAJIB dipanggil di setiap iterasi */
    }
    return (state == S_EXPECT_SEPARATOR_OR_END && nObj >= 1);
}
```

Penelusuran `dia ambil tas , buku.`:

| Token | State sesudahnya |
|---|---|
| `dia` | PREDICATE |
| `ambil` | FIRST_OBJECT |
| `tas` | SEPARATOR_OR_END |
| `,` | OBJECT_AFTER_COMMA |
| `buku` | SEPARATOR_OR_END |
| (pita habis) | valid → 1 |

`aku beli buku, .` berakhir di OBJECT_AFTER_COMMA (sesudah `buku,`), jadi 0. `aku beli .` berakhir di FIRST_OBJECT, jadi 0.

**Kenapa solusi arsip hanya dapat 66:**

```c
/* potongan solusi arsip */
case S_EXPECT_SEPARATOR_OR_END:
    if (isCommaToken(currentWord)) currState = S_EXPECT_OBJECT_AFTER_COMMA;
    break;                                   /* (2) token lain diabaikan */
...
case S_INVALID:
    continue;                                /* (1) loncat ke awal loop, ADVWORD tidak dipanggil */
}
ADVWORD();
```

```c
boolean isObjToken(Word w, int *hasTrailingComma) {
    *hasTrailingComma = (int)(w.Length && w.TabWord[w.Length - 1] == ',');
    return !isCommaToken(w);                 /* (3) tidak cek alfabet */
}
```

1. **Time limit exceeded.** `continue` di dalam `switch` yang ada di dalam `while` akan melompat ke iterasi berikutnya **tanpa** memanggil `ADVWORD()`. Pita tidak maju, `EndWord` tidak pernah true, loop tak berhenti. Ini terjadi pada setiap input invalid yang masih punya kata sesudah token salah, misalnya `beli buku.`. Perbaikan: `break` (tetap invalid) lalu `ADVWORD()` tetap jalan, atau langsung `return 0`.
2. **Wrong answer.** Di `S_EXPECT_SEPARATOR_OR_END`, token selain koma diam-diam diabaikan. `aku beli buku pensil.` (dua objek tanpa koma) jadi dianggap valid. Seharusnya ke `S_INVALID`.
3. **Wrong answer.** `isObjToken` tidak memeriksa huruf, jadi `buku2` atau `buku,pensil` diterima sebagai objek.

Hasil uji versi perbaikan: 8 contoh soal cocok, plus `aku beli buku pensil.` → 0, `aku beli buku2.` → 0, `aku beli buku,pensil.` → 0, `kamu beli x, y , z.` → 1, `.` → 0.

> **Perlu dicek:** soal tidak eksplisit menyebut apakah koma di tengah token (`buku,pensil`) valid. Kalimat "Objek dianggap valid jika seluruh karakternya alfabet (kecuali koma opsional di posisi paling akhir token)" mengarah ke **tidak valid**, dan kunci ini mengikuti tafsiran itu.

### Pembahasan K1 sampai K3 · mesin karakter

```c
/* K1 */
void k1(int *nChar, int *nA) {
    *nChar = 0; *nA = 0;
    START();
    while (!EOP) {
        (*nChar)++;
        if (currentChar == 'A') (*nA)++;
        ADV();
    }
}
```

Parameter output di C memakai pointer. Perhatikan `(*nChar)++`, bukan `*nChar++`. Tanpa kurung, `++` menggeser pointer, bukan menambah nilainya.

**K2**: lihat `hitungKataPakaiKarakter()` di Bagian 4. Setiap loop dalam wajib punya `!EOP &&` supaya tidak `ADV()` melewati titik. Urutan operand juga penting: `!EOP` diperiksa dulu, berkat short-circuit `&&`.

```c
/* K3 */
int k3() {
    int cnt = 0;
    char prev = ' ';                 /* belum ada karakter sebelumnya */
    START();
    while (!EOP) {
        if (prev == 'L' && currentChar == 'E') cnt++;
        prev = currentChar;          /* simpan SESUDAH dibandingkan */
        ADV();
    }
    return cnt;
}
```

Mesin karakter tidak bisa melihat ke belakang, jadi karakter sebelumnya disimpan di `prev`. Inisialisasi `prev` dengan karakter yang pasti bukan `'L'`. Hasil uji: 0, 1, 1, 2, 0 sesuai contoh.

### Pembahasan W1 sampai W4 · mesin kata

```c
/* W1: rata-rata panjang */
int n = 0, tot = 0;
STARTWORD();
while (!EndWord) { tot += currentWord.Length; n++; ADVWORD(); }
if (n == 0) printf("Pita tidak mengandung kata\n");
else printf("%.2f\n", (float) tot / n);
```

Cast `(float)` wajib. Tanpa itu, `8 / 3` adalah pembagian integer = 2.

```c
/* W2: kata terpanjang */
Word best; best.Length = -1;          /* penanda belum ada kata */
STARTWORD();
while (!EndWord) {
    if (currentWord.Length > best.Length) best = currentWord;   /* salin struct */
    ADVWORD();
}
if (best.Length < 0) printf("Pita tidak mengandung kata\n");
else { printWord(best); printf(" %d\n", best.Length); }
```

`best = currentWord` menyalin seluruh struct, jadi aman walau `currentWord` ditimpa `ADVWORD()`. Pakai `>` (bukan `>=`) supaya yang pertama menang saat seri.

```c
/* W3: jumlah bilangan */
int sum = 0;
STARTWORD();
while (!EndWord) { sum += wordToInt(currentWord); ADVWORD(); }
printf("%d\n", sum);
```

`wordToInt` (Bagian 3.5) memakai `TabWord[i] - '0'` untuk mengubah karakter digit ke angka. Pola `val = val * 10 + digit` membaca dari kiri ke kanan. Pola ini sering dipakai di soal yang input-nya angka (misalnya membaca ukuran lalu isi array).

```c
/* W4: kemunculan kata pertama */
Word first; int cnt = 0;
STARTWORD();
if (EndWord) { printf("0\n"); return 0; }
first = currentWord;
while (!EndWord) {
    if (isWordEqual(first, currentWord)) cnt++;
    ADVWORD();
}
printf("%d\n", cnt);
```

Kata pertama ikut terhitung karena loop dimulai dari kata pertama itu sendiri. Hasil uji semua W cocok: `2.67`, `BANDUNG 7`, `150`, `3`.

---

## 9. Checklist sebelum submit

- [ ] Nama fungsi dan variabel **sama persis** dengan `.h` yang diberikan (besar kecil huruf berpengaruh: `STARTWORD` ≠ `StartWord`).
- [ ] Kalau soal meminta fungsi saja (seperti `palindrome.c`, `parser.c`), **jangan kumpulkan `main`**, atau beri komentar seperti di skeleton. Grader biasanya punya `main` sendiri.
- [ ] Tidak memakai `printf("%s", currentWord.TabWord)` atau `strcmp` pada `Word`.
- [ ] Setiap loop mesin kata memanggil `ADVWORD()` di **setiap** iterasi, termasuk cabang error. Kalau tidak: TLE.
- [ ] Setiap loop mesin karakter mengecek `!EOP` sebelum `ADV()`.
- [ ] Format output persis: newline di akhir, tidak ada spasi berlebih (evaluator Olympia: *Exact*).
- [ ] Diuji dengan: pita kosong `.`, hanya blank, satu kata, blank berlebih di awal, tengah, dan sebelum titik.
- [ ] Kompilasi dengan `-Wall` dan baca semua warning.

---

**Sumber:** slide IF2110 09x02 (Mesin Abstrak dan Kontrak) dan 12x01 (Mesin Kata); arsip PTPB25-Archival, `Semester 3/IF2110 - Algoritma dan Pemrograman 2/Praktikum/Praktikum 8` (soal.pdf, mesinkarakter.h, mesinkata.h, dan file solusi).
