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
