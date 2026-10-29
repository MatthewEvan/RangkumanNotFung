Saya sudah baca slide Mesin Abstrak. Sekarang saya baca bagian mesin karakter di dokumen persiapan praktikum supaya penjelasannya sesuai dengan yang akan diujikan.

Saya sudah baca slide **09x02 Mesin Abstrak dan Kontrak** dan bagian mesin karakter/mesin kata di dokumen **Persiapan Praktikum 8**. Ini penjelasannya, disusun mengikuti urutan slide.

## 1. Ide besar: mesin abstrak

**Mesin abstrak** adalah mesin yang *kita anggap ada* dan sanggup menjalankan beberapa perintah dasar. Kita tidak peduli bagaimana mesin itu dibuat di dalamnya. Mesin abstrak terdiri dari dua hal:

| Komponen | Arti | Analogi: pemutar kaset |
|---|---|---|
| **State** | kondisi mesin saat ini, yang boleh kamu lihat | lagu yang sedang diputar, lampu "kaset habis" |
| **Aksi primitif** | perintah dasar yang dimengerti mesin | tombol *play*, tombol *next* |

Kamu bisa memakai pemutar kaset tanpa tahu cara kerja motornya. Cukup tahu tombol apa saja yang ada dan apa yang terjadi kalau tombol itu ditekan.

Slide pertama memakai contoh `qsort` dari C. Untuk memakainya, kamu cukup membaca nama dan parameternya, serta tahu bahwa fungsi itu mengurutkan `n` elemen. Kamu tidak perlu tahu algoritma pengurutan apa yang dipakai di dalamnya.

## 2. Mesin karakter

Ini mesin abstrak utama di materi ini, dan akan muncul di praktikum.

```
Pita:     I T B _ A D A _ D I .
                  ▲
               jendela (cc)          Lampu EOP: ○ padam
```

| Bagian | Arti |
|---|---|
| **Pita** | deret karakter yang selalu diakhiri **MARK** `'.'` |
| **Pita kosong** | pita yang hanya berisi MARK: `.` |
| **Jendela / `cc`** (*current character*) | satu-satunya karakter yang bisa kamu lihat saat ini |
| **Lampu `eop`** (*end of pita*) | menyala (`true`) jika `cc` adalah MARK |
| **Tombol `start`** | mulai membaca, jendela menunjuk karakter pertama |
| **Tombol `adv`** (*advance*) | geser jendela satu karakter ke kanan |

**State mesin karakter = `cc` + `eop`.** Hanya itu yang bisa kamu lihat. Kamu **tidak bisa** mundur, mengintip karakter berikutnya, atau tahu panjang pita sebelum selesai membaca. Batasan ini yang membuat soal-soalnya menarik.

### Jejak mesin pada pita `ITB.`

| Aksi | `cc` | `eop` | Keterangan |
|---|---|---|---|
| `start()` | `'I'` | false | jendela di karakter pertama |
| `adv()` | `'T'` | false | |
| `adv()` | `'B'` | false | |
| `adv()` | `'.'` | **true** | sampai MARK, lampu menyala |
| `adv()` | ❌ | | **dilarang**: melanggar kontrak (lihat bagian 3) |

Pada pita kosong `.`, `start()` langsung menghasilkan `cc = '.'` dan `eop = true`.

## 3. Kontrak: I.S. dan F.S.

Setiap aksi primitif punya **kontrak** yang terdiri dari dua bagian:

```
procedure start
{ I.S.: sembarang
  F.S.: cc adalah karakter pertama pada pita.
        Jika cc ≠ MARK maka eop padam; jika cc = MARK maka eop menyala. }

procedure adv
{ I.S.: cc ≠ MARK
  F.S.: cc adalah karakter berikutnya dari cc yang lama (mungkin MARK).
        Jika cc = MARK maka eop menyala. }
```

- **I.S.** (*Initial State*) adalah syarat yang harus benar **sebelum** aksi dipanggil.
- **F.S.** (*Final State*) adalah keadaan yang dijamin benar **setelah** aksi dipanggil.

Ini mirip perjanjian antara dua pihak:

| Pihak | Tanggung jawab | Contoh |
|---|---|---|
| **Pemanggil** (kamu, program pemakai) | menjamin I.S. terpenuhi | tidak memanggil `adv()` saat `cc == MARK` |
| **Pemasok** (yang menulis `mesinkar.c`) | menjamin F.S. **jika** I.S. terpenuhi | setelah `adv()`, `cc` benar-benar karakter berikutnya |

Kalau kamu memanggil `adv()` saat `eop` sudah menyala, I.S. dilanggar. Hasilnya **tidak terdefinisi** dan **kesalahannya ada di kamu**, bukan di pemasok. Di implementasi slide, pita bahkan sudah ditutup dengan `fclose`, jadi program bisa error.

Bandingkan dengan prasyarat di Haskell, misalnya "Prasyarat: pohon tidak kosong" di `maxTree`. Konsepnya sama: pemanggil wajib menjamin prasyarat, dan kalau dilanggar, fungsi boleh error.

## 4. Kontrak di C: berkas `.h` vs `.c`

Di C, kontrak ditulis di **berkas header (`.h`)**, sedangkan isinya ditulis di **berkas `.c`**.

### `mesinkar.h`: kontrak (yang dibaca pemakai)

```c
#ifndef __MESIN_KAR__
#define __MESIN_KAR__
#include "boolean.h"
#define MARK '.'

/* State mesin */
extern char cc;
extern boolean eop;

void start(); /* I.S.: sembarang; F.S.: cc = karakter pertama */
void adv();   /* I.S.: cc != MARK; F.S.: cc = berikutnya */
#endif
```

Cara membacanya:

| Pertanyaan | Dijawab oleh | Di `mesinkar.h` |
|---|---|---|
| Apa yang boleh **dibaca** pemakai? | `extern` | `cc` dan `eop` |
| Apa yang boleh **dipanggil**? | prototipe fungsi | `start` dan `adv` |
| Apa yang harus benar **sebelum** memanggil? | I.S. (di komentar) | `adv`: `cc != MARK` |
| Apa yang dijamin **sesudahnya**? | F.S. (di komentar) | `cc` = karakter berikutnya, `eop` true jika `cc == MARK` |

Beberapa istilah C yang perlu kamu tahu:

- **`extern char cc;`** artinya "variabel `cc` *ada*, tapi didefinisikan di file lain". Ini membuat program pemakai bisa membaca `cc` walaupun `cc` dibuat di `mesinkar.c`.
- **`#ifndef ... #define ... #endif`** adalah *include guard*: mencegah isi header dimasukkan dua kali kalau `#include` dipanggil berulang.
- **`#define MARK '.'`**: konstanta. Pakai `MARK` di kode, jangan tulis `'.'` langsung, supaya kodenya tetap benar kalau MARK diganti.

### `mesinkar.c`: realisasi (yang tidak perlu dibaca pemakai)

```c
#include "mesinkar.h"
#include <stdio.h>

char cc;               /* definisi asli state */
boolean eop;
static FILE *pita;     /* static: tersembunyi, hanya terlihat di file ini */

void start() {
    pita = fopen("pitakar.txt", "r");
    adv();
}

void adv() {
    fscanf(pita, "%c", &cc);
    eop = (cc == MARK);
    if (eop) { fclose(pita); }
}
```

**`static FILE *pita`** penting: kata `static` membuat `pita` **tidak bisa diakses dari luar** `mesinkar.c`. Pemakai memang tidak boleh tahu dari mana karakter dibaca. Inilah yang dimaksud "menyembunyikan implementasi".

Catatan: versi di dokumen persiapan memakai `pita = stdin` (membaca dari input), bukan `fopen("pitakar.txt")`. Cek SOP praktikum untuk memastikan mana yang dipakai.

## 5. Kenapa dipisah? Satu program pemakai, dua implementasi

Ini inti dari materi kontrak. Slide 14 menunjukkan implementasi **lain**, yaitu pita yang dibaca dari string, bukan dari file:

```c
/* mesinkar_str.c */
static char *pita = "ITB.";
static int pos;

void start() { pos = 0; cc = pita[pos]; eop = (cc == MARK); }
void adv()   { pos++;   cc = pita[pos]; eop = (cc == MARK); }
```

Program pemakai (`hitungA.c`) **tidak perlu diubah sama sekali**. Ia tetap benar dengan `mesinkar.c` maupun `mesinkar_str.c`, karena ia hanya bergantung pada kontrak (`cc`, `eop`, `start`, `adv`), bukan pada cara kerja di dalamnya.

> Program pemakai bergantung pada `.h`, bukan pada isi `.c`.

Ini sama dengan template Haskell kemarin: kamu memakai `akar`, `left`, dan `isTreeEmpty` tanpa peduli bagaimana fungsi-fungsi itu ditulis.

## 6. Pola pemrosesan dengan MARK

Hampir semua soal mesin karakter memakai kerangka yang sama:

```
Inisialisasi       { siapkan variabel, misalnya ctr ← 0 }
start              { First Elmt: jendela di karakter pertama }
while (cc ≠ MARK)  { selama belum EOP }
    Proses cc      { kerjakan sesuatu dengan karakter sekarang }
    adv            { Next Elmt: geser ke karakter berikutnya }
{ cc = MARK }
Terminasi          { cetak hasil }
```

Dalam C:

```c
int main() {
    int ctr = 0;                 /* Inisialisasi */
    start();                     /* First Elmt   */
    while (cc != MARK) {         /* not EOP      */
        if (cc == 'A') {         /* Proses       */
            ctr++;
        }
        adv();                   /* Next Elmt    */
    }
    /* cc == MARK */
    printf("%d\n", ctr);         /* Terminasi    */
    return 0;
}
```

Perhatikan bahwa pola ini **otomatis aman**:
- `adv()` hanya dipanggil di dalam loop, dan loop hanya berjalan saat `cc != MARK`. Jadi I.S. `adv` selalu terpenuhi.
- Pita kosong ditangani dengan benar: `start()` langsung menghasilkan `cc == MARK`, loop tidak dijalankan, dan hasilnya `0`.

Jejak `hitungA` pada pita `ADA.`:

| Langkah | `cc` | Proses | `ctr` |
|---|---|---|---|
| `start()` | `'A'` | `cc == 'A'` ✔ | 1 |
| `adv()` | `'D'` | ✘ | 1 |
| `adv()` | `'A'` | ✔ | 2 |
| `adv()` | `'.'` | loop berhenti | **2** |

## 7. Kalau butuh "ingat" karakter sebelumnya

Mesin hanya menampilkan **satu** karakter. Jadi soal seperti **Hitung-LE** (hitung pasangan `L` yang langsung diikuti `E`) tidak bisa diselesaikan hanya dengan melihat `cc`. Kamu tidak bisa mundur atau mengintip ke depan.

Solusinya: **simpan sendiri** informasi yang dibutuhkan di variabel lokal. Untuk Hitung-LE, yang perlu diingat adalah karakter sebelumnya (`prev`). Di setiap langkah, cek apakah pasangan `(prev, cc)` adalah `('L', 'E')`, lalu perbarui `prev = cc` sebelum `adv()`.

Jejak pada pita `ELLE.` (hasil seharusnya `1`):

| `cc` | `prev` (sebelum dicek) | Pasangan `LE`? | `ctr` |
|---|---|---|---|
| `'E'` | (belum ada) | ✘ | 0 |
| `'L'` | `'E'` | ✘ | 0 |
| `'L'` | `'L'` | ✘ | 0 |
| `'E'` | `'L'` | ✔ | **1** |
| `'.'` | | loop berhenti | 1 |

Pertanyaan untukmu: nilai awal `prev` sebaiknya apa, supaya karakter pertama tidak salah dihitung? Petunjuk: pilih karakter yang pasti bukan `'L'`.

Teknik ini mirip **akumulator** di `foldl`: informasi dibawa dari satu langkah ke langkah berikutnya.

## 8. Mesin kata: mesin abstrak di atas mesin abstrak

Dokumen persiapan juga membahas **mesin kata**. Mesin ini dibangun **memakai** mesin karakter, tapi jendelanya satu **kata**, bukan satu karakter:

| | Mesin karakter | Mesin kata |
|---|---|---|
| State | `cc`, `eop` | `currentWord`, `EndWord` |
| Aksi | `start`, `adv` | `STARTWORD`, `ADVWORD` |
| Pola loop | `while (cc != MARK)` | `while (!EndWord)` |

Ini menunjukkan kekuatan kontrak: `mesinkata.c` hanya memakai `start`, `adv`, `cc`, dan `MARK`. Ia sendiri menjadi pemakai mesin karakter, lalu menyediakan kontrak baru untuk program di atasnya.

Hal penting dari dokumen persiapan: `TabWord` **tidak** diakhiri `'\0'`, jadi untuk mencetak dan membandingkan kata, jangan pakai `%s` atau `strcmp`. Pakai loop sampai `Length`.

## 9. Kaitan dengan materi kemurnian

`start()` dan `adv()` adalah kebalikan dari fungsi murni di Haskell. Keduanya **tidak mengembalikan nilai** (`void`). Yang mereka lakukan hanya **mengubah state global** (`cc`, `eop`) dan membaca dunia luar (file atau keyboard). Memanggil `adv()` dua kali memberi `cc` yang berbeda. Karena itu kontrak I.S./F.S. sangat penting di sini: tanpa kontrak, kamu tidak bisa tahu state mesin setelah sebuah aksi.

## 10. Jebakan yang perlu diingat (dari dokumen persiapan)

- **Jangan panggil `adv()` saat `cc == MARK`.** Ini melanggar I.S.
- **Input tanpa titik** membuat loop tidak pernah berhenti (TLE). Saat tes manual, jangan lupa mengetik `.` di akhir.
- **Pita kosong** (`.` saja) harus menghasilkan 0. Hati-hati pembagian dengan nol, misalnya saat menghitung rata-rata.
- **Newline bukan BLANK** di ADT standar. Input `a`↵`b.` dibaca sebagai satu kata.
- Baca **`.h` dari asisten** sebelum mulai mengerjakan, karena nama macro dan format output bisa berbeda dari dokumen persiapan.

## Cara compile dan tes

```bash
gcc -std=c99 -Wall -Wextra -o hitungLE hitungLE.c mesinkar.c
```

```bash
echo "LLE LE EL." | .\hitungLE.exe
```

Perhatikan bahwa yang dikompilasi adalah **program pemakai + realisasi mesin** (`mesinkar.c`). Berkas `.h` tidak ikut ditulis di perintah, karena sudah dimasukkan lewat `#include`.

## Ringkasan

- **Mesin abstrak** = state + aksi primitif. Untuk mesin karakter, state-nya `cc` dan `eop`, aksinya `start` dan `adv`.
- **Kontrak** ditulis di `.h`. `extern` menyatakan state yang boleh dibaca, prototipe menyatakan aksi yang boleh dipanggil, dan komentar I.S./F.S. menyatakan janjinya.
- **I.S. adalah janji pemanggil, F.S. adalah janji pemasok.** Kalau I.S. dilanggar, kesalahannya ada di pemanggil.
- Program pemakai **hanya bergantung pada `.h`**, jadi implementasi `.c` bisa diganti tanpa mengubah program pemakai.
- Pola standar: `start` → `while (cc != MARK) { proses; adv }` → terminasi. Kalau butuh informasi dari langkah sebelumnya, simpan di variabel lokal.

Coba kerjakan Hitung-LE sendiri dulu, tanpa melihat solusi di dokumen persiapan. Kalau sudah, tempel kodenya ke sini dan saya cek. Setelah itu, kasih tahu saya kalau mau lanjut ke linked list.


# Pembahasan Latihan Soal
# K0: Hitung-LE

### Soal

Diberikan sebuah pita karakter yang diakhiri MARK `'.'`. Hitunglah banyaknya **pasangan huruf `L` yang langsung diikuti huruf `E`** pada pita. Pita kosong menghasilkan 0.

Program **hanya boleh** memakai `start`, `adv`, `cc`, dan `MARK` dari `mesinkar.h`.

**Input:** satu pita karakter yang diakhiri `.`
**Output:** satu bilangan bulat, yaitu banyaknya pasangan `LE`, diikuti newline.

| Input | Output | Penjelasan |
|---|---|---|
| `.` | `0` | pita kosong |
| `LE.` | `1` | |
| `ELLE.` | `1` | hanya `L` kedua yang diikuti `E` |
| `LLE LE EL.` | `2` | `LLE` (satu) dan `LE` (satu). `EL` bukan `LE` |
| `L E.` | `0` | ada spasi di antara `L` dan `E`, jadi tidak langsung |
| `LEE LLLE.` | `2` | |

### Pembahasan

**Masalah utamanya:** jendela mesin hanya menampilkan satu karakter. Saat `cc == 'E'`, kamu tidak bisa melihat karakter sebelumnya. Mesin tidak bisa mundur.

**Ide:** simpan sendiri karakter sebelumnya di variabel lokal `prev`. Di setiap langkah, cek pasangan `(prev, cc)`.

```c
#include <stdio.h>
#include "mesinkar.h"

int main() {
    int ctr = 0;
    char prev = MARK;          /* nilai awal: pasti bukan 'L' */
    start();
    while (cc != MARK) {
        if (prev == 'L' && cc == 'E') {
            ctr++;
        }
        prev = cc;             /* cc sekarang jadi "sebelumnya" di langkah berikut */
        adv();
    }
    printf("%d\n", ctr);
    return 0;
}
```

**Kenapa `prev` diawali `MARK`?** Saat memproses karakter pertama, belum ada karakter sebelumnya. `prev` harus diisi nilai yang **pasti bukan `'L'`**, supaya pita `E...` tidak salah terhitung. `MARK` adalah pilihan aman, karena MARK tidak pernah muncul di tengah pita.

**Kenapa `prev = cc` ditulis sebelum `adv()`?** Setelah `adv()`, nilai `cc` lama sudah hilang. Jadi kalau mau disimpan, harus disimpan sebelum `adv()`.

**Jejak `LLE LE EL.`:**

| `cc` | `prev` saat dicek | `LE`? | `ctr` |
|---|---|---|---|
| `L` | `.` | ✘ | 0 |
| `L` | `L` | ✘ | 0 |
| `E` | `L` | ✔ | 1 |
| ` ` | `E` | ✘ | 1 |
| `L` | ` ` | ✘ | 1 |
| `E` | `L` | ✔ | 2 |
| ` ` | `E` | ✘ | 2 |
| `E` | ` ` | ✘ | 2 |
| `L` | `E` | ✘ | 2 |
| `.` | | loop berhenti | **2** |

**Pola yang bisa dipakai ulang:** kalau soal mesin karakter butuh informasi dari langkah sebelumnya, simpan di variabel. Teknik ini berlaku untuk mendeteksi pasangan, menghitung kemunculan berturut-turut, dan sejenisnya.

---

# K1: Hitung karakter dan vokal

### Soal

Diberikan sebuah pita karakter yang diakhiri MARK. Hitunglah:
1. **banyaknya karakter** sebelum MARK (spasi ikut dihitung), dan
2. **banyaknya huruf vokal** (`a`, `i`, `u`, `e`, `o`), **tanpa membedakan huruf besar dan kecil**.

**Input:** satu pita karakter yang diakhiri `.`
**Output:** dua bilangan bulat dipisah spasi: `<jumlah karakter> <jumlah vokal>`, diikuti newline.

| Input | Output | Penjelasan |
|---|---|---|
| `Halo Dunia.` | `10 5` | 10 karakter termasuk spasi. Vokal: a, o, u, i, a |
| `.` | `0 0` | pita kosong |
| `AIUEO xyz.` | `9 5` | huruf besar juga dihitung sebagai vokal |

### Pembahasan

Ini soal paling langsung: satu kali traversal dengan **dua penghitung sekaligus**.

```c
#include <stdio.h>
#include "mesinkar.h"

static boolean isVokal(char c) {
    if (c >= 'A' && c <= 'Z') {
        c = c - 'A' + 'a';            /* ubah huruf besar jadi huruf kecil */
    }
    return c == 'a' || c == 'i' || c == 'u' || c == 'e' || c == 'o';
}

int main() {
    int total = 0, vokal = 0;
    start();
    while (cc != MARK) {
        total++;                      /* setiap karakter dihitung */
        if (isVokal(cc)) {
            vokal++;                  /* vokal dihitung terpisah */
        }
        adv();
    }
    printf("%d %d\n", total, vokal);
    return 0;
}
```

**Trik huruf besar ke kecil:** `c - 'A' + 'a'`. Di tabel ASCII, huruf besar dan kecil sama-sama berurutan. `c - 'A'` memberi posisi huruf di alfabet (A=0, B=1, ...), lalu `+ 'a'` mengubahnya ke huruf kecil di posisi yang sama. Contoh: `'E' - 'A' + 'a'` = `4 + 'a'` = `'e'`. Fungsi `tolower` dari `<ctype.h>` juga bisa dipakai, kalau soal mengizinkan.

**Kenapa dibuat fungsi `isVokal` terpisah?** Supaya `main` tetap bersih dan polanya terlihat jelas. `static` membuat fungsi ini hanya terlihat di file ini, sama seperti `static FILE *pita` di `mesinkar.c`.

**Hati-hati:** "karakter sebelum MARK" berarti **spasi ikut dihitung**. `MARK` sendiri tidak dihitung, karena loop berhenti saat `cc == MARK`.

---

# K2: Statistik kata (mesin kata)

### Soal

Diberikan sebuah pita berisi kata-kata yang dipisahkan satu atau lebih spasi, diakhiri MARK. Gunakan **mesin kata** untuk mencetak:
1. **jumlah kata**,
2. **kata terpanjang**. Jika ada beberapa kata dengan panjang sama, ambil yang **muncul pertama**,
3. **rata-rata panjang kata**, dengan 2 angka di belakang koma.

Jika tidak ada kata sama sekali, **cetak baris pertama saja**.

**Input:** satu pita yang diakhiri `.`
**Output:**
```
Jumlah kata: <n>
Kata terpanjang: <kata>
Rata-rata panjang: <x.xx>
```

| Input | Output |
|---|---|
| `aku suka makan nasi goreng.` | `Jumlah kata: 5`<br>`Kata terpanjang: goreng`<br>`Rata-rata panjang: 4.40` |
| `  spasi di awal .` | `Jumlah kata: 3`<br>`Kata terpanjang: spasi`<br>`Rata-rata panjang: 3.67` |
| `.` | `Jumlah kata: 0` |
| `abc de fgh.` | `Jumlah kata: 3`<br>`Kata terpanjang: abc`<br>`Rata-rata panjang: 2.67` |

### Pembahasan

Ini soal pertama yang memakai **mesin kata**. Pola loop-nya sama dengan mesin karakter, tapi satuannya kata:

```c
STARTWORD();
while (!EndWord) {
    /* proses currentWord */
    ADVWORD();
}
```

Ada tiga hal yang dikumpulkan dalam **satu** traversal: jumlah kata, total panjang, dan kata terpanjang.

```c
int main() {
    int count = 0, totalLen = 0;
    Word longest;
    longest.Length = 0;                      /* belum ada kata, panjang 0 */
    STARTWORD();
    while (!EndWord) {
        count++;
        totalLen += currentWord.Length;
        if (currentWord.Length > longest.Length) {   /* > bukan >= */
            longest = currentWord;           /* salin seluruh struct */
        }
        ADVWORD();
    }
    printf("Jumlah kata: %d\n", count);
    if (count > 0) {
        printf("Kata terpanjang: ");
        printWord(longest);
        printf("\n");
        printf("Rata-rata panjang: %.2f\n", (float) totalLen / count);
    }
    return 0;
}
```

Empat detail penting:

1. **`>` bukan `>=`.** Soal minta kata terpanjang yang muncul **pertama** jika seri. Dengan `>`, kata berikutnya yang panjangnya sama tidak menggantikan kata pertama. Contoh `abc de fgh.`: `abc` (3) disimpan, lalu `fgh` (3) tidak menggantikannya karena `3 > 3` bernilai false.

2. **`longest = currentWord` menyalin seluruh isi struct**, termasuk array `TabWord`. Di C, assignment struct menyalin semua field. Penyalinan ini **wajib**, karena `currentWord` akan ditimpa oleh `ADVWORD()` berikutnya.

3. **`(float) totalLen / count`.** Tanpa cast, `totalLen / count` adalah pembagian bilangan bulat: `22 / 5 = 4`, bukan `4.40`. Cast ke `float` harus dilakukan **sebelum** pembagian.

4. **`if (count > 0)`.** Ini mencegah pembagian dengan nol pada pita kosong, dan sekaligus memenuhi aturan "cetak baris pertama saja".

**Kenapa memakai `printWord`, bukan `printf("%s", ...)`?** `TabWord` **tidak diakhiri `'\0'`**, jadi `%s` akan terus mencetak sampah memori sampai kebetulan menemukan byte nol. `printWord` mencetak tepat `Length` karakter:

```c
void printWord(Word w) {
    for (int i = 0; i < w.Length; i++) {
        printf("%c", w.TabWord[i]);
    }
}
```

**Contoh kedua (spasi di awal):** `STARTWORD` memanggil `IgnoreBlanks` lebih dulu, jadi spasi di awal pita dan spasi sebelum `.` tidak menghasilkan kata kosong. Kata-katanya `spasi` (5), `di` (2), `awal` (4), sehingga rata-ratanya 11/3 = 3.67.

---

# K3: Balik tiap kata

### Soal

Diberikan sebuah pita berisi kata-kata, diakhiri MARK. Cetak setiap kata dengan **urutan hurufnya dibalik**, tapi **urutan katanya tetap**. Pisahkan kata dengan **tepat satu spasi**, walaupun di input jarak antarkatanya lebih dari satu spasi. Akhiri output dengan newline.

| Input | Output |
|---|---|
| `halo dunia kita.` | `olah ainud atik` |
| `a bc .` | `a cb` |
| `  ITB   keren.` | `BTI nerek` |
| `.` | *(baris kosong)* |

### Pembahasan

**Membalik satu kata itu mudah**, karena `currentWord` sudah tersimpan lengkap di array. Cukup cetak dari indeks terakhir ke indeks pertama:

```c
for (int i = currentWord.Length - 1; i >= 0; i--) {
    printf("%c", currentWord.TabWord[i]);
}
```

Ini kelebihan mesin kata dibanding mesin karakter. Dengan mesin karakter, kamu tidak bisa membalik apa pun karena karakter yang sudah lewat tidak bisa dilihat lagi. Mesin kata menyimpan satu kata utuh di `TabWord`, jadi di dalam satu kata kamu bebas mengakses indeks mana saja.

**Bagian yang lebih sulit adalah spasinya.** Output harus `olah ainud atik`, tanpa spasi di awal maupun di akhir. Ada dua pilihan:
- cetak spasi **setelah** setiap kata: akan ada spasi berlebih di akhir ✘
- cetak spasi **sebelum** setiap kata **kecuali kata pertama** ✔

```c
int main() {
    boolean first = true;
    STARTWORD();
    while (!EndWord) {
        if (!first) {
            printf(" ");                 /* spasi pemisah, kecuali sebelum kata pertama */
        }
        for (int i = currentWord.Length - 1; i >= 0; i--) {
            printf("%c", currentWord.TabWord[i]);
        }
        first = false;
        ADVWORD();
    }
    printf("\n");
    return 0;
}
```

**Kenapa ini penting?** Output diperiksa secara otomatis dan harus sama persis. Spasi di akhir baris bisa membuat jawaban dinilai salah (WA).

**Spasi ganda di input** (`  ITB   keren.`) tidak perlu ditangani secara khusus. `IgnoreBlanks` di mesin kata sudah melewati semua spasi, dan program hanya mencetak satu spasi di antara kata.

---

# K4: Frekuensi kata target

### Soal

Diberikan sebuah pita berisi kata-kata, diakhiri MARK. **Kata pertama adalah kata target.** Hitung berapa kali kata target muncul **di sisa kalimat**, yaitu kata kedua sampai terakhir. Kata dianggap sama hanya jika **sama persis** (panjang dan semua hurufnya sama). Jika pita kosong, cetak `0`.

**Output:** satu bilangan bulat, diikuti newline.

| Input | Output | Penjelasan |
|---|---|---|
| `ab ab cd ab abc.` | `2` | target `ab`. Sisanya: `ab` ✔, `cd`, `ab` ✔, `abc` ✘ (lebih panjang) |
| `ab.` | `0` | hanya ada target, sisanya kosong |
| `.` | `0` | pita kosong |
| `x y x x z.` | `2` | |

### Pembahasan

Soal ini menggabungkan tiga hal: **menangani kasus khusus kata pertama**, **menyimpan salinan**, dan **membandingkan kata**.

```c
int main() {
    STARTWORD();
    if (EndWord) {                    /* pita kosong: tidak ada target */
        printf("0\n");
        return 0;
    }
    Word target = currentWord;        /* SALIN kata pertama */
    int count = 0;
    ADVWORD();                        /* lewati target, mulai dari kata kedua */
    while (!EndWord) {
        if (isWordEqual(currentWord, target)) {
            count++;
        }
        ADVWORD();
    }
    printf("%d\n", count);
    return 0;
}
```

**1. Kenapa ada `if (EndWord)` sebelum loop?** Pada pita kosong, tidak ada kata pertama yang bisa dijadikan target. Tanpa pengecekan ini, `target` berisi data yang tidak bermakna.

**2. Kenapa `target` harus disalin?** `currentWord` adalah **satu variabel global** milik mesin kata. Setiap `ADVWORD()` **menimpa** isinya. Kalau kamu hanya "mengingat" `currentWord`, target ikut berubah. `Word target = currentWord;` menyalin isinya ke variabel lokal yang tidak akan diubah mesin.

**3. Kenapa `ADVWORD()` dipanggil sekali sebelum loop?** Soal minta menghitung di **sisa** kalimat. Kalau tidak dilewati, target akan menghitung dirinya sendiri, dan `ab ab cd ab abc.` menghasilkan 3, bukan 2.

**4. Kenapa tidak memakai `strcmp`?** Karena `TabWord` tidak diakhiri `'\0'`. Bandingkan secara manual:

```c
boolean isWordEqual(Word a, Word b) {
    if (a.Length != b.Length) {
        return false;                 /* beda panjang pasti beda: ab ≠ abc */
    }
    for (int i = 0; i < a.Length; i++) {
        if (a.TabWord[i] != b.TabWord[i]) {
            return false;
        }
    }
    return true;
}
```

Pengecekan panjang di awal adalah yang membuat `abc` tidak dianggap sama dengan `ab`. Kalau hanya membandingkan sebanyak `a.Length` huruf, `ab` dan `abc` akan dianggap sama, dan itu salah.

---

# K5: Angka terurut (mesin kata + list)

### Soal

Diberikan sebuah pita berisi bilangan bulat (boleh negatif) yang dipisahkan spasi, diakhiri MARK. Baca setiap bilangan dengan mesin kata, lalu **sisipkan ke dalam list berkait sehingga list selalu terurut menaik**. Setelah semua dibaca:
1. cetak list dengan format `[a,b,c]` (tanpa spasi, list kosong ditulis `[]`),
2. cetak jumlah semua elemen di baris berikutnya.

Bebaskan semua memori list di akhir program.

| Input | Output |
|---|---|
| `5 -3 10 0 2.` | `[-3,0,2,5,10]`<br>`14` |
| `7 7 -1.` | `[-1,7,7]`<br>`13` |
| `.` | `[]`<br>`0` |

### Pembahasan

Ini soal penghubung antara dua materi. **Mesin kata membaca** input yang banyaknya tidak diketahui, dan **list menyimpan** hasilnya.

**Langkah 1: mengubah `Word` menjadi `int`.** Mesin kata hanya memberi deret karakter, misalnya `'-'`, `'3'`. Kamu harus mengubahnya sendiri menjadi angka `-3`:

```c
int wordToInt(Word w) {
    int i = 0, sign = 1, val = 0;
    if (w.Length > 0 && w.TabWord[0] == '-') {
        sign = -1;                            /* catat tanda negatif */
        i = 1;                                /* mulai dari digit setelah '-' */
    }
    for (; i < w.Length; i++) {
        val = val * 10 + (w.TabWord[i] - '0');
    }
    return sign * val;
}
```

Dua trik di sini:
- **`w.TabWord[i] - '0'`** mengubah karakter digit menjadi angka. Digit `'0'` sampai `'9'` berurutan di ASCII, jadi `'7' - '0'` = `7`.
- **`val = val * 10 + digit`** membangun angka dari kiri ke kanan. Contoh untuk `"105"`:

| digit | `val` |
|---|---|
| `1` | 0 × 10 + 1 = 1 |
| `0` | 1 × 10 + 0 = 10 |
| `5` | 10 × 10 + 5 = 105 |

**Langkah 2: program utama.**

```c
int main() {
    List l;
    CreateList(&l);
    STARTWORD();
    while (!EndWord) {
        insertSorted(&l, wordToInt(currentWord));   /* sisip di posisi yang benar */
        ADVWORD();
    }
    int sum = 0;
    for (Address p = FIRST(l); p != NULL; p = NEXT(p)) {   /* traversal list */
        sum += INFO(p);
    }
    displayList(l);
    printf("\n%d\n", sum);
    freeList(&l);                                    /* kembalikan semua memori */
    return 0;
}
```

Perhatikan bahwa ada **dua loop sekuensial** dengan pola yang sama: loop mesin kata (`STARTWORD` / `!EndWord` / `ADVWORD`) dan loop traversal list (`FIRST` / `!= NULL` / `NEXT`).

**Kenapa disimpan di list, bukan langsung diproses?** Mengurutkan butuh **semua** angka. Kamu tidak bisa tahu angka terkecil sebelum seluruh pita dibaca, dan pita tidak bisa dibaca dua kali. Banyaknya angka juga tidak diketahui sejak awal, jadi list yang bisa terus bertambah lebih cocok daripada array berukuran tetap.

**Perhatikan `displayList` dan newline.** Di dokumen persiapan, `displayList` mencetak `[...]` **tanpa** newline. Karena itu `printf("\n%d\n", sum)` diawali `\n`, untuk pindah baris setelah list dicetak.

**`insertSorted` dan `freeList`** dibahas di latihan list (L6 dan L1). Isinya akan kita bahas saat masuk materi linked list. Untuk sekarang, cukup pahami kontraknya:
- `insertSorted(&l, x)` menyisipkan `x` sehingga list tetap terurut menaik,
- `freeList(&l)` membebaskan semua node dan membuat `l` menjadi kosong.

---

## Pola umum dari keenam soal

| Soal | Mesin | Teknik kunci |
|---|---|---|
| K0 | karakter | simpan karakter sebelumnya (`prev`) |
| K1 | karakter | beberapa penghitung dalam satu traversal |
| K2 | kata | simpan salinan kata terbaik (`longest`), cast `float`, cegah bagi nol |
| K3 | kata | akses `TabWord` secara terbalik, spasi pemisah dengan flag `first` |
| K4 | kata | tangani kata pertama di luar loop, salin `target`, bandingkan kata secara manual |
| K5 | kata + list | ubah kata jadi angka, simpan ke list karena butuh semua data |

Kasus tepi yang **selalu** perlu dites: pita kosong (`.`), satu elemen saja, dan spasi berlebih (di awal, di akhir, atau di antara kata).

Saran urutan latihan: K0 dan K2 tercantum di checklist hari-H dokumen persiapan, jadi kerjakan keduanya dulu tanpa melihat pembahasan ini. Kalau mau dicek, tempel kodenya ke sini.
