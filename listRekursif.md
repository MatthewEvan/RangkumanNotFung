# List Rekursif dalam Konteks Prosedural (C)

Catatan persiapan Praktikum 9 IF2110, berdasarkan slide **IF2110_11_1** dan **IF2110_11_2 List Rekursif (Prosedural)**, ditambah pembahasan soal Post Praktikum 9 tahun lalu (bagian 17). Semua kode C di bawah sudah dikompilasi dengan `gcc -std=c99 -Wall -Wextra` (GCC 15.2) dan dijalankan. Output yang ditulis adalah output aslinya.

> Catatan terkait: list berkait iteratif ada di materi Praktikum 8, dan rekursi list di Haskell ada di [`../penjelasanLambda.md`](../penjelasanLambda.md) serta [`../binarytree.md`](../binarytree.md). Materi ini menggabungkan keduanya.

---

## Daftar Isi

1. [Mengingat kembali: list berkait](#1-mengingat-kembali-list-berkait)
2. [Cara pandang rekursif terhadap list](#2-cara-pandang-rekursif-terhadap-list)
3. [Jembatan dari Haskell](#3-jembatan-dari-haskell)
4. [Struktur data dan header `list.h`](#4-struktur-data-dan-header-listh)
5. [Primitif dasar: `isEmpty`, `head`, `tail`, `konso`](#5-primitif-dasar-isempty-head-tail-konso)
6. [Pola rekursi list](#6-pola-rekursi-list)
7. [Studi kasus slide 1: `displayList`, `length`, `search`](#7-studi-kasus-slide-1-displaylist-length-search)
8. [Fungsi vs prosedur di C](#8-fungsi-vs-prosedur-di-c)
9. [Mengubah list secara rekursif: trik `&NEXT(*l)`](#9-mengubah-list-secara-rekursif-trik-nextl)
10. [Studi kasus slide 2: `konsDot`, `copy`, `concat`](#10-studi-kasus-slide-2-konsdot-copy-concat)
11. [Memori: berbagi node vs menyalin](#11-memori-berbagi-node-vs-menyalin)
12. [Rekursif vs iteratif](#12-rekursif-vs-iteratif)
13. [Strategi mengerjakan soal](#13-strategi-mengerjakan-soal)
14. [Kesalahan umum](#14-kesalahan-umum)
15. [Cheat sheet](#15-cheat-sheet)
16. [Latihan soal](#16-latihan-soal)
17. [Pembahasan soal Post Praktikum 9 (2025)](#17-pembahasan-soal-post-praktikum-9-2025)
18. [Lampiran: kode lengkap untuk dites](#18-lampiran-kode-lengkap-untuk-dites)

---

## 1. Mengingat kembali: list berkait

List berkait adalah **rantai node**. Setiap node menyimpan:
- `info`: nilai elemen,
- `next`: alamat node berikutnya (`NIL` kalau sudah tidak ada).

List itu sendiri **hanya pointer ke node pertama**. List kosong berarti pointer itu `NIL`.

```
  l
  │
  ▼
┌───┬───┐   ┌───┬───┐   ┌───┬─────┐
│ 3 │ ●─┼──►│ 5 │ ●─┼──►│ 7 │ NIL │
└───┴───┘   └───┴───┘   └───┴─────┘
 info next
```

Di Praktikum 8, list diproses secara **iteratif**: sebuah pointer `p` digeser dengan loop.

```c
Address p = l;
while (p != NIL) {
    /* proses INFO(p) */
    p = NEXT(p);
}
```

Di Praktikum 9, list yang **sama persis** diproses secara **rekursif**. Struktur datanya tidak berubah. Yang berubah adalah **cara kita memandang list**.

---

## 2. Cara pandang rekursif terhadap list

Slide membuka dengan definisi ini:

> **Basis:** list kosong adalah list.
> **Rekurens:** list tidak kosong terdiri atas **sebuah elemen** dan **sisanya adalah list**.

Artinya, setiap list yang tidak kosong bisa dipecah menjadi dua bagian:

```
  l = [3, 5, 7]

  ┌───┐   ┌───────────┐
  │ 3 │ + │ [5, 7]    │
  └───┘   └───────────┘
  head      tail  ← ini juga sebuah list!

  tail lagi:   [5, 7]  = 5 + [7]
  tail lagi:   [7]     = 7 + []
  tail lagi:   []      ← basis
```

Yang penting: **`tail` dari sebuah list adalah list juga**. Jadi masalah untuk list `[3, 5, 7]` bisa diselesaikan dengan:
1. mengurus elemen pertama (`3`), lalu
2. menyelesaikan masalah **yang sama** untuk list yang lebih pendek (`[5, 7]`), dengan memanggil fungsi itu sendiri.

Karena setiap pemanggilan membuat list lebih pendek satu elemen, cepat atau lambat kita sampai ke list kosong (basis) dan rekursi berhenti.

### Bandingkan dua cara berpikir

Soal: hitung banyak elemen list.

| Iteratif (Praktikum 8) | Rekursif (Praktikum 9) |
|---|---|
| "Siapkan penghitung 0. Jalan dari node pertama sampai `NIL`, tambah 1 di setiap node." | "Panjang list kosong adalah 0. Panjang list tidak kosong adalah 1 + panjang sisanya." |
| Memikirkan **langkah-langkah** | Memikirkan **definisi** |
| Butuh variabel `p` dan `counter` | Tidak butuh variabel bantu |

```c
/* Iteratif */                      /* Rekursif */
int length(List l) {                int length(List l) {
    int n = 0;                          if (isEmpty(l)) {
    Address p = l;                          return 0;
    while (p != NIL) {                  } else {
        n++;                                return 1 + length(tail(l));
        p = NEXT(p);                    }
    }                               }
    return n;
}
```

Versi rekursif dibaca hampir persis seperti definisi matematikanya.

---

## 3. Jembatan dari Haskell

Kamu sudah menulis rekursi list di Haskell. Ternyata konsepnya **sama persis**, hanya sintaksnya yang berbeda.

| Konsep | Haskell | C (list rekursif) |
|---|---|---|
| List kosong | `[]` | `NIL` |
| Cek kosong | `null l` atau pola `[]` | `isEmpty(l)` |
| Elemen pertama | `head l` atau `x` di pola `(x:xs)` | `head(l)` = `INFO(l)` |
| Sisa list | `tail l` atau `xs` di pola `(x:xs)` | `tail(l)` = `NEXT(l)` |
| Tambah di depan | `x : l` | `konso(x, l)` |
| Tambah di belakang | `l ++ [x]` | `konsDot(l, x)` |

Contohnya `sumList`:

```haskell
-- Haskell
sumList :: [Int] -> Int
sumList []     = 0
sumList (x:xs) = x + sumList xs
```

```c
/* C */
int sumList(List l) {
    if (isEmpty(l)) {
        return 0;
    } else {
        return head(l) + sumList(tail(l));
    }
}
```

Slide 11_2 bahkan menyebut langsung: *"ingat kembali list dalam pemrograman fungsional"*, dengan notasi `[ ]` atau `[e ∘ List]`. Pola `(x:xs)` di Haskell **adalah** pemecahan `head` + `tail`.

Bedanya hanya satu, tapi penting: **di C kamu yang mengurus memori**. Haskell membuat list baru secara otomatis dan membersihkannya sendiri. Di C, `konso` memanggil `malloc`, dan node yang tidak dipakai lagi harus di-`free`. Bagian 11 membahas ini.

---

## 4. Struktur data dan header `list.h`

Representasinya sama dengan list berkait Praktikum 8. Header di bawah adalah ringkasan dari `list.h` yang dipakai di praktikum tahun lalu (ada di folder ini), dengan nama primitif yang sama persis:

```c
#ifndef LIST_H
#define LIST_H

#include "boolean.h"
#include <stdio.h>
#include <stdlib.h>

#define NIL NULL

typedef int ElType;
typedef struct node *Address;
typedef struct node {
    ElType info;
    Address next;
} Node;
typedef Address List;

/* Selektor */
#define INFO(p) (p)->info
#define NEXT(p) (p)->next

/* Manajemen memori */
Address newNode(ElType x);
/* Mengirimkan address hasil alokasi sebuah elemen.
   Jika berhasil: INFO = x, NEXT = NIL. Jika gagal: NIL */

/* Pemeriksaan kondisi list */
boolean isEmpty(List l);
/* true jika l kosong */
boolean isOneElmt(List l);
/* true jika l berisi tepat 1 elemen */

/* Primitif pemrosesan list */
ElType head(List l);
/* Elemen pertama list l yang tidak kosong */
List tail(List l);
/* l tanpa elemen pertamanya, mungkin list kosong */
List konso(ElType e, List l);
/* l dengan tambahan e sebagai elemen PERTAMA. Jika alokasi gagal, mengirimkan l */
List konsDot(List l, ElType e);
/* l dengan tambahan e sebagai elemen TERAKHIR. Jika alokasi gagal, mengirimkan l */

#endif
```

Beberapa hal yang perlu diperhatikan:

- **`NIL` (huruf besar semua)**, bukan `Nil` seperti di Praktikum 8. Ikuti header yang diberikan asisten.
- **Urutan parameter `konso` adalah `(ElType e, List l)`**, yaitu elemen dulu, baru list. Ini sama dengan notasi algoritmik slide (`kons(e, l)`) dan dengan Haskell (`e : l`), dan dipakai oleh `list.h` praktikum tahun lalu.
- **`konsDot`** adalah "kons belakang" (di slide ditulis `kons•`). Urutan parameternya **kebalikan** `konso`: `konsDot(List l, ElType e)`, yaitu list dulu, baru elemen. Cara mengingatnya: posisi elemen di parameter sama dengan posisinya di list hasil. `konso(e, l)` menaruh `e` di depan, `konsDot(l, e)` menaruh `e` di belakang.
- **`list.c` sudah mengimplementasikan** `displayList`, `length`, `search`, `deleteX`, `copy`, `mCopy`, `concat`, dan `mConcat`. Di soal praktikum, fungsi-fungsi ini tinggal dipakai. Jangan menulis ulang fungsi dengan nama yang sama di file jawaban, karena akan terjadi *multiple definition* saat linking.

> ⚠️ **Urutan parameter `konso` berbeda antarsumber.** Header C di slide 11_2 halaman 14 menulis `konso(List l, ElType e)` (list dulu), sedangkan `list.h` praktikum tahun lalu menulis `konso(ElType e, List l)` (elemen dulu). Catatan ini mengikuti `list.h` praktikum. Kalau urutannya tertukar, misalnya `konso(l, 3)`, GCC 15 di laptopmu langsung menolaknya dengan error `makes integer from pointer without a cast`. Tapi GCC versi 13 ke bawah hanya memberi *warning*, programnya tetap jadi, lalu hasilnya salah atau crash saat dijalankan. Versi GCC di server penilai belum tentu sama, jadi jangan mengandalkan compiler. **Selalu baca `.h` dari asisten sebelum mulai, dan jangan abaikan warning `-Wint-conversion`.**

---

## 5. Primitif dasar: `isEmpty`, `head`, `tail`, `konso`

```c
boolean isEmpty(List l) {
    return l == NIL;
}

boolean isOneElmt(List l) {
    return l != NIL && NEXT(l) == NIL;
}

ElType head(List l) {
    return INFO(l);
}

List tail(List l) {
    return NEXT(l);
}

List konso(ElType e, List l) {
    Address p = newNode(e);
    if (p == NIL) {
        return l;          /* alokasi gagal: list tidak berubah */
    }
    NEXT(p) = l;           /* node baru menunjuk ke list lama */
    return p;              /* node baru menjadi kepala list */
}
```

`head` dan `tail` hanya **membungkus** `INFO` dan `NEXT`. Gunanya supaya kode terbaca dengan bahasa "list", bukan bahasa "pointer".

```
head([3,5,7]) = 3
tail([3,5,7]) = [5,7]
isOneElmt([3,5,7]) = false
isOneElmt([9])     = true
isOneElmt([])      = false
```

### `tail` tidak menyalin

```c
List t = tail(a);    /* t menunjuk ke node [5] milik a */
```

Hasil tes: `tail(a) == NEXT(a)` bernilai **ya, node yang sama**. `tail` tidak membuat list baru. Ia hanya memberi alamat node kedua. Jadi kalau kamu mengubah `INFO(t)`, `a` ikut berubah, dan kalau kamu mem-`free` `t`, sebagian `a` ikut rusak.

### `konso` tidak mengubah list lama

```
Sebelum:     l ──► [5] ──► [7] ──► NIL

konso(3, l):
             p ──► [3] ──┐
                         ▼
             l ──► [5] ──► [7] ──► NIL

Hasil:       [3] ──► [5] ──► [7] ──► NIL     (yang dikembalikan adalah p)
```

```c
List b = konso(3, L(5, 7));   /* b = [3,5,7] */
```

`konso` hanya membuat satu node baru dan menaruhnya di depan. Node-node lama tidak disentuh.

> 💡 Hasil `konso` **harus ditampung**: `l = konso(3, l);`. Kalau hanya ditulis `konso(3, l);`, node baru dibuat tapi alamatnya dibuang, sehingga terjadi kebocoran memori dan `l` tidak berubah.

---

## 6. Pola rekursi list

Hampir semua fungsi rekursif list mengikuti template ini:

```c
TipeHasil f(List l) {
    if (isEmpty(l)) {           /* BASIS 0: list kosong */
        return ...;
    } else {                    /* REKURENS */
        ... head(l) ... f(tail(l)) ...
    }
}
```

Setiap kali menulis fungsi baru, jawab tiga pertanyaan:

1. **Apa jawaban untuk list kosong?** Itu basis.
2. **Kalau aku sudah tahu jawaban untuk `tail(l)`, bagaimana menggabungkannya dengan `head(l)`?** Itu rekurens. Percayalah bahwa `f(tail(l))` sudah benar, lalu pikirkan satu langkah saja.
3. **Apakah list kosong bermakna untuk soal ini?** Kalau soal punya prasyarat "list tidak kosong" (misalnya mencari maksimum), basisnya menjadi **list satu elemen** (`isOneElmt`).

### Basis 0 dan basis 1

Slide memakai istilah ini:

| Istilah | Kondisi | Contoh fungsi |
|---|---|---|
| **Basis 0** | list kosong | `length`, `sumList`, `search`, `copy` |
| **Basis 1** | list satu elemen, **atau** jawaban sudah ketemu sebelum list habis | `maxList`, `lastElmt`, `search` versi 2 |

Ini sama dengan pohon biner kemarin: pohon boleh kosong → basis `Empty`. Pohon tidak kosong → basis daun.

### Tabel cepat: soal → basis dan rekurens

| Soal | Basis (`isEmpty`) | Rekurens |
|---|---|---|
| banyak elemen | `0` | `1 + f(tail)` |
| jumlah elemen | `0` | `head + f(tail)` |
| hitung kemunculan `x` | `0` | `(head == x ? 1 : 0) + f(tail)` |
| cari `x` | `false` | `head == x \|\| f(tail)` |
| semua memenuhi syarat | `true` | `syarat(head) && f(tail)` |
| salin list | `NIL` | `konso(head, f(tail))` |
| maksimum (tidak kosong) | *basis 1:* `head` | `max(head, f(tail))` |

---

## 7. Studi kasus slide 1: `displayList`, `length`, `search`

### Studi kasus 1: `displayList`

```c
void displayList(List l) {
/* I.S. l terdefinisi. F.S. setiap elemen list dicetak */
    if (isEmpty(l)) {
        /* basis: tidak melakukan apa-apa */
    } else {
        printf("%d ", head(l));
        displayList(tail(l));
    }
}
```

```
displayList([3,5,7])  →  3 5 7
displayList([])       →  (tidak mencetak apa-apa)
```

**Jejak eksekusi:**

```
displayList([3,5,7])
├─ cetak 3
└─ displayList([5,7])
   ├─ cetak 5
   └─ displayList([7])
      ├─ cetak 7
      └─ displayList([])   ← basis, kembali
```

**Pertanyaan menarik:** apa yang terjadi kalau dua baris di rekurens ditukar?

```c
displayList(tail(l));     /* rekursi DULU */
printf("%d ", head(l));   /* baru cetak */
```

Output menjadi `7 5 3`, yaitu **terbalik**. Pemanggilan rekursif selesai lebih dulu (sampai ujung list), baru elemen dicetak saat "pulang". Ini trik penting: **posisi pemrosesan relatif terhadap pemanggilan rekursif menentukan urutan**, sama seperti pre-order dan post-order di pohon biner.

#### Format `[a,b,c]`

Soal praktikum biasanya minta format `[1,2,3]`. Koma hanya dicetak **di antara** elemen, jadi pisahkan pencetakan kurung (sekali) dari pencetakan isi (rekursif):

```c
static void displayIsi(List l) {
    if (!isEmpty(l)) {
        printf("%d", head(l));
        if (!isEmpty(tail(l))) {
            printf(",");            /* masih ada elemen sesudahnya */
        }
        displayIsi(tail(l));
    }
}

void displayBracket(List l) {
    printf("[");
    displayIsi(l);
    printf("]");
}
```

```
displayBracket([3,5,7])  →  [3,5,7]
displayBracket([])       →  []
```

Pola "fungsi luar + fungsi bantu rekursif" ini sering dipakai kalau ada sesuatu yang hanya boleh dilakukan **sekali** (di sini kurung `[` dan `]`).

### Studi kasus 2a: `length` versi fungsi

```c
int length(List l) {
    if (isEmpty(l)) {
        return 0;                       /* basis 0 */
    } else {
        return 1 + length(tail(l));     /* rekurens */
    }
}
```

**Jejak `length([3,5,7])`:**

```
length([3,5,7])
= 1 + length([5,7])
= 1 + (1 + length([7]))
= 1 + (1 + (1 + length([])))
= 1 + (1 + (1 + 0))
= 3
```

Perhatikan bahwa penjumlahan baru bisa dikerjakan **setelah** semua pemanggilan sampai ke basis. Semua `1 + ...` "menunggu" di *call stack*.

### Studi kasus 2b: `length` versi prosedur

Di notasi algoritmik:

```
procedure length(input l: List, output n: integer)
```

`output n` di C menjadi **pointer**:

```c
void lengthP(List l, int *n) {
    int n1;
    if (isEmpty(l)) {
        *n = 0;
    } else {
        lengthP(tail(l), &n1);    /* hasil untuk tail disimpan di n1 */
        *n = 1 + n1;
    }
}
```

Cara memakainya:

```c
int hasil;
lengthP(a, &hasil);     /* hasil = 3 */
```

Kenapa butuh variabel lokal `n1`? Karena prosedur tidak "mengembalikan" nilai. Untuk tahu panjang `tail`, kita harus menyediakan tempat (`n1`) dan memberikan alamatnya ke pemanggilan rekursif.

### Studi kasus 2c: `length` dengan akumulator

```c
void lengthAcc(List l, int acc, int *n) {
    if (isEmpty(l)) {
        *n = acc;                         /* hasil sudah terkumpul di acc */
    } else {
        lengthAcc(tail(l), acc + 1, n);   /* tambah 1, lanjut ke tail */
    }
}

void lengthPAcc(List l, int *n) {
    lengthAcc(l, 0, n);                   /* akumulator mulai dari 0 */
}
```

**Jejak `lengthAcc([3,5,7], 0, n)`:**

| Pemanggilan | `l` | `acc` |
|---|---|---|
| 1 | `[3,5,7]` | 0 |
| 2 | `[5,7]` | 1 |
| 3 | `[7]` | 2 |
| 4 | `[]` | 3 → `*n = 3` |

Bedanya dengan versi 2a: penghitungan dilakukan **sebelum** turun ke rekursi berikutnya (`acc + 1`), bukan setelah kembali. Saat sampai di basis, jawabannya sudah lengkap. Tidak ada operasi yang "menunggu".

Ini sama persis dengan `foldl` di Haskell, yaitu akumulator dibawa dari kiri. Sedangkan versi 2a mirip `foldr`, yang hasilnya dibangun saat pulang.

Fungsi pembungkus `lengthPAcc` diperlukan supaya pemakai tidak perlu tahu bahwa ada akumulator yang harus diisi `0`. Pola ini juga dipakai di `sumDeepestLeaves` waktu kita belajar pohon.

### Studi kasus 3: `search`

**Versi 1**, dengan operator logika:

```c
boolean search(List l, ElType x) {
    if (isEmpty(l)) {
        return false;
    } else {
        return head(l) == x || search(tail(l), x);
    }
}
```

**Versi 2**, dengan basis 1 eksplisit:

```c
boolean search2(List l, ElType x) {
    if (isEmpty(l)) {              /* basis 0: tidak ketemu */
        return false;
    } else if (head(l) == x) {     /* basis 1: ketemu, berhenti */
        return true;
    } else {                       /* rekurens */
        return search2(tail(l), x);
    }
}
```

```
search([3,5,7], 5) = true
search([3,5,7], 4) = false
search([], 1)      = false
```

Kedua versi **sama-sama berhenti begitu `x` ditemukan**. Di versi 1, ini terjadi karena `||` di C bersifat *short-circuit*: kalau `head(l) == x` sudah `true`, `search(tail(l), x)` tidak dijalankan. Versi 2 hanya menuliskannya secara eksplisit.

---

## 8. Fungsi vs prosedur di C

Slide sering menampilkan **dua versi** untuk operasi yang sama. Ini terjemahannya ke C:

| Notasi algoritmik | C | Kapan |
|---|---|---|
| `function f(l: List) → T` | `T f(List l)` | menghasilkan satu nilai |
| `procedure p(input l: List, output n: T)` | `void p(List l, T *n)` | hasil dikirim lewat parameter |
| `procedure p(input/output l: List)` | `void p(List *l)` | **list-nya sendiri berubah** |

Aturan penting dari Praktikum 8 tetap berlaku: **C selalu mengirim salinan**. Kalau sebuah prosedur perlu mengubah variabel milik pemanggil (termasuk variabel `List`), yang dikirim harus **alamatnya**.

---

## 9. Mengubah list secara rekursif: trik `&NEXT(*l)`

Ini bagian **paling penting dan paling membingungkan** dari materi ini.

### Masalahnya

Lihat `deleteX` dari slide 11_1, yang menghapus semua elemen bernilai `x`:

```
procedure deleteX(input/output l: List, input x: ElType)
    if (isEmpty(l)) then
        { do nothing }
    else
        deleteX(l.next, x)          ← rekursi pada l.next sebagai input/output
        if l.info = x then
            p ← l
            l ← l.next
            dealokasi(p)
```

Perhatikan `deleteX(l.next, x)`. Parameternya `input/output`, artinya pemanggilan rekursif **boleh mengubah `l.next`**. Misalnya kalau node sesudah `l` dihapus, `l.next` harus diarahkan ke node sesudahnya lagi.

Di C, `input/output` menjadi pointer: `void deleteX(List *l, ElType x)`. Jadi pemanggilan rekursifnya harus memberikan **alamat dari field `next`**:

```c
deleteX(&NEXT(*l), x);
```

### Membaca `&NEXT(*l)` pelan-pelan

| Ekspresi | Tipe | Arti |
|---|---|---|
| `l` | `List *` | alamat variabel list milik pemanggil |
| `*l` | `List` | variabel list itu sendiri, yaitu alamat node pertama |
| `NEXT(*l)` | `List` | field `next` di dalam node pertama. Field ini juga sebuah `List`: kepala dari sisa list |
| `&NEXT(*l)` | `List *` | **alamat** field `next` itu |

Kunci pemahamannya: **field `next` di dalam node adalah variabel bertipe `List`**. Ia berperan sebagai "kepala list" bagi sisa list. Jadi untuk pemanggilan rekursif, field `next` itulah yang menjadi "variabel list milik pemanggil".

```
Pemanggil:   l_main ──► [4] ──► [1] ──► [4] ──► NIL
               ▲          next     next
               │           ▲        ▲
               │           │        └─ &NEXT pada level 2: "kepala" untuk [4]
               │           └─ &NEXT pada level 1: "kepala" untuk [1,4]
               └─ &l_main pada level 0: kepala untuk seluruh list
```

Setiap level rekursi memegang **alamat dari pointer yang menunjuk ke node-nya**. Kalau sebuah level menjalankan `*l = NEXT(*l)`, yang berubah adalah pointer **milik level di atasnya**. Dengan begitu node bisa dilepas dari rantai tanpa perlu variabel `prec` seperti di versi iteratif.

### `deleteX` lengkap

```c
void deleteX(List *l, ElType x) {
/* I.S. l dan x terdefinisi. F.S. semua elemen l yang bernilai x dihapus */
    Address p;
    if (isEmpty(*l)) {
        /* basis: tidak melakukan apa-apa */
    } else {
        deleteX(&NEXT(*l), x);     /* bersihkan sisa list dulu */
        if (INFO(*l) == x) {       /* lalu urus node ini */
            p = *l;
            *l = NEXT(*l);         /* pointer milik pemanggil melompati node ini */
            free(p);
        }
    }
}
```

```
deleteX([4,1,4,4,2,4], 4)  →  [1,2]
deleteX([4,4], 4)          →  []
deleteX([1,2], 9)          →  [1,2]
```

**Jejak `deleteX([4,1,4], 4)`:** sisa list dibersihkan lebih dulu, lalu node diurus saat "pulang".

```
Level 0: *l = [4] → [1] → [4]
  Level 1: *l = [1] → [4]          (l = &NEXT dari node 4 pertama)
    Level 2: *l = [4]              (l = &NEXT dari node 1)
      Level 3: *l = NIL → basis, kembali
    Level 2: INFO = 4 == x → NEXT milik node [1] diubah jadi NIL, node 4 dibebaskan
             list sekarang: [4] → [1] → NIL
  Level 1: INFO = 1 ≠ x → tidak berubah
Level 0: INFO = 4 == x → l_main diubah menunjuk [1], node 4 dibebaskan
Hasil: [1]
```

Kasus "hapus elemen pertama" tidak perlu ditangani terpisah. Level 0 memegang `&l_main`, jadi mengubah `*l` di level 0 otomatis mengubah kepala list milik pemanggil. Inilah keindahan versi rekursif.

### `konsDot` versi prosedur: trik yang sama

Slide 11_2 halaman 7 menambah elemen di belakang:

```
procedure kons•(input/output l: List, input e: ElType)
    if isEmpty(l) then
        l ← alokasi(e)
    else
        kons•(l.next, e)
```

```c
void konsDotP(List *l, ElType e) {
    if (isEmpty(*l)) {
        *l = newNode(e);           /* ujung ditemukan: isi "kepala" kosong ini */
    } else {
        konsDotP(&NEXT(*l), e);      /* turun ke sisa list */
    }
}
```

```c
List d = NIL;
konsDotP(&d, 1);
konsDotP(&d, 2);
konsDotP(&d, 3);     /* d = [1,2,3] */
```

Rekursi turun sampai menemukan `next` yang bernilai `NIL` (atau list yang memang kosong), lalu mengisinya dengan node baru. Karena yang dipegang adalah **alamat** field `next` itu, perubahannya langsung tersambung ke rantai.

---

## 10. Studi kasus slide 2: `konsDot`, `copy`, `concat`

### `konsDot` versi fungsi

```c
List konsDot(List l, ElType e) {
    if (isEmpty(l)) {
        return newNode(e);              /* list kosong: hasilnya node baru */
    }
    NEXT(l) = konsDot(tail(l), e);        /* sambungkan hasil rekursi ke node ini */
    return l;
}
```

```
konsDot([3,5], 7)  →  [3,5,7]
```

Ini versi fungsi dari `konsDotP`. Alih-alih memberikan alamat field `next`, pemanggilan rekursif **mengembalikan kepala sisa list yang baru**, lalu hasilnya ditulis ulang ke `NEXT(l)`. Pada sebagian besar level, `NEXT(l)` ditulis dengan nilai yang sama seperti sebelumnya. Yang benar-benar berubah hanya level terakhir, ketika `NIL` diganti node baru.

> ⚠️ `konsDot` versi fungsi **mengubah list `l`** (node terakhirnya disambung ke node baru). Jadi walaupun bentuknya fungsi, ia tidak murni. Berbeda dengan `l ++ [x]` di Haskell yang membuat list baru.

### `copy`: menyalin list

```c
List copy(List l) {
    if (isEmpty(l)) {
        return NIL;
    } else {
        return konso(head(l), copy(tail(l)));
    }
}
```

Bacanya: **salinan dari `[h, ...sisa]` adalah `h` ditaruh di depan salinan `sisa`.** Persis seperti definisi Haskell `copy (x:xs) = x : copy xs`.

```
copy([3,5,7])
= konso(3, copy([5,7]))
= konso(3, konso(5, copy([7])))
= konso(3, konso(5, konso(7, copy([]))))
= konso(3, konso(5, konso(7, NIL)))
= konso(3, konso(5, [7]))
= konso(3, [5,7])
= [3,5,7]           ← semua node BARU
```

Hasil tes menunjukkan bahwa node hasil `copy` **berbeda** dari node asli. Mengubah salinan (`INFO(e) = 100` menghasilkan `[100,5,7]`) tidak mengubah list asli (tetap `[3,5,7]`).

### `mCopy`: versi prosedur

```c
void mCopy(List lin, List *lout) {
    List lTemp;
    if (isEmpty(lin)) {
        *lout = NIL;
    } else {
        mCopy(tail(lin), &lTemp);           /* salin sisa ke lTemp */
        *lout = konso(head(lin), lTemp);    /* tambahkan head di depannya */
    }
}
```

Polanya sama dengan `lengthP`: variabel lokal `lTemp` menampung hasil rekursi, lalu hasil akhirnya ditulis ke `*lout`.

### `concat`: menyambung dua list

```c
List concat(List l1, List l2) {
/* Mengirimkan SALINAN hasil konkatenasi l1 dan l2 */
    if (isEmpty(l1)) {
        return copy(l2);
    } else {
        return konso(head(l1), concat(tail(l1), l2));
    }
}
```

```
concat([1,2], [3,4])  →  [1,2,3,4]
concat([1], [])       →  [1]
```

Basisnya `copy(l2)`, bukan `l2`. Alasannya: spesifikasi meminta **salinan**. Kalau basis mengembalikan `l2` langsung, hasil `concat` akan **berbagi node** dengan `l2`. Dengan begitu, mem-`free` hasil concat juga merusak `l2`, dan sebaliknya.

### `mConcat`: versi prosedur

```c
void mConcat(List l1, List l2, List *result) {
    List lTemp;
    if (isEmpty(l1)) {
        *result = copy(l2);
    } else {
        mConcat(tail(l1), l2, &lTemp);
        *result = konso(head(l1), lTemp);
    }
}
```

```
mConcat([], [8,9])  →  [8,9]
```

---

## 11. Memori: berbagi node vs menyalin

Di Haskell, kamu tidak pernah memikirkan memori. Di C, setiap fungsi list jatuh ke salah satu dari tiga kategori. Kamu **harus** tahu fungsi yang kamu tulis termasuk yang mana:

| Kategori | Contoh | Membuat node baru? | Mengubah list asli? | Konsekuensi |
|---|---|---|---|---|
| **Hanya membaca** | `length`, `search`, `sumList`, `displayList` | tidak | tidak | aman |
| **Membuat list baru** | `copy`, `concat`, `inverse`, `takeN` | ya (`konso`/`newNode`) | tidak | hasilnya harus di-`free` nanti |
| **Mengubah di tempat** | `deleteX`, `konsDotP`, `insertSorted`, `gandakan` | kadang | **ya** | butuh `List *l` kalau kepala list bisa berubah |

### Membebaskan list secara rekursif

```c
void freeList(List *l) {
    if (!isEmpty(*l)) {
        freeList(&NEXT(*l));   /* bebaskan sisa list dulu */
        free(*l);              /* baru node ini */
        *l = NIL;
    }
}
```

Urutannya penting: sisa list dibebaskan **lebih dulu**. Kalau `free(*l)` dijalankan duluan, `NEXT(*l)` tidak boleh diakses lagi karena node-nya sudah dikembalikan ke sistem.

### Jebakan: `tail` berbagi node

```c
List a = ...;              /* [3,5,7] */
List t = tail(a);          /* t menunjuk ke node 5 MILIK a, bukan salinan */
freeList(&t);              /* ✘ membebaskan node 5 dan 7 milik a! */
```

Setelah ini, `a` masih menunjuk ke node 3, tapi `NEXT` dari node 3 menunjuk ke memori yang sudah dibebaskan. Kalau butuh sisa list yang **terpisah**, pakai `copy(tail(a))`.

---

## 12. Rekursif vs iteratif

| | Rekursif | Iteratif |
|---|---|---|
| Kode | pendek, mirip definisi | lebih panjang, butuh variabel bantu |
| Menghapus/menyisip | tidak perlu `prec` (pakai `&NEXT(*l)`) | perlu `prec` dan kasus khusus elemen pertama |
| Memori tambahan | satu *stack frame* per elemen | konstan |
| List sangat panjang | bisa **stack overflow** | aman |

Seberapa panjang yang masih aman? Hasil tes `length` rekursif di Windows (GCC 15.2, stack bawaan 1 MB):

| Panjang list | Hasil |
|---|---|
| 1.000 | ✔ |
| 10.000 | ✔ |
| 30.000 | ✔ |
| 100.000 | ✘ crash, *stack overflow* (kode keluar `0xC00000FD`) |

Batasan soal praktikum biasanya jauh di bawah angka ini (misalnya `n ≤ 1000`), jadi rekursi aman dipakai. Tapi kalau batasannya ratusan ribu, versi iteratif lebih aman. Batas pastinya juga bergantung pada sistem operasi dan compiler server penilai.

---

## 13. Strategi mengerjakan soal

### Langkah-langkah

1. **Baca header.** Cek nama fungsi, urutan parameter (`konso(e, l)` atau `konso(l, e)`?), dan apakah tiap parameter `List` atau `List *`.
2. **Tentukan kategori** (bagian 11): hanya membaca, membuat list baru, atau mengubah di tempat?
3. **Tentukan basis.** List boleh kosong → basis `isEmpty`. Prasyarat tidak kosong → basis `isOneElmt`.
4. **Tulis rekurens dengan "percaya pada rekursi".** Anggap `f(tail(l))` sudah benar, lalu pikirkan cara menggabungkannya dengan `head(l)`.
5. **Kalau mengubah list**, pakai `List *l` dan rekursi pada `&NEXT(*l)`.
6. **Jejaki dengan list kecil**: `[]`, `[a]`, `[a,b]`.
7. **Tes** kasus tepi: list kosong, satu elemen, elemen di awal, elemen di akhir.

### Kapan proses di depan, kapan di belakang?

| Proses **sebelum** pemanggilan rekursif | Proses **sesudah** pemanggilan rekursif |
|---|---|
| urutan alami (depan ke belakang) | urutan terbalik (belakang ke depan) |
| `displayList` biasa | `displayList` terbalik |
| akumulator (`lengthAcc`) | `deleteX` (sisa dibersihkan dulu) |
| `search` (berhenti begitu ketemu) | `freeList` (sisa dibebaskan dulu) |

---

## 14. Kesalahan umum

| Kesalahan | Penyebab | Perbaikan |
|---|---|---|
| crash di `head(l)` / `INFO(l)` | dipanggil pada list kosong | cek `isEmpty(l)` dulu |
| rekursi tidak berhenti / *stack overflow* | pemanggilan rekursif memakai `l`, bukan `tail(l)` | pastikan list mengecil di setiap pemanggilan |
| `konso(l, 3)` error tipe atau hasil aneh | urutan parameter salah | cek header: `konso(ElType e, List l)` |
| `konso(x, l);` tidak mengubah apa-apa | hasil tidak ditampung | `l = konso(x, l);` |
| `deleteX(NEXT(*l), x)` error tipe | lupa `&` | `deleteX(&NEXT(*l), x)` |
| prosedur yang mengubah list tidak berefek | parameter `List l`, bukan `List *l` | ubah jadi `List *l`, panggil dengan `&l` |
| `maxList` salah untuk bilangan negatif | basis `isEmpty` mengembalikan `0` | prasyarat tidak kosong → basis `isOneElmt` |
| `concat` lalu `free` merusak list lain | basis mengembalikan `l2`, bukan `copy(l2)` | ikuti spesifikasi: salin |
| akses memori setelah `free` | `free(*l)` sebelum `freeList(&NEXT(*l))` | bebaskan sisa list dulu |
| output `[1,2,3,]` (koma berlebih) | koma dicetak setelah setiap elemen | cetak koma hanya jika `tail` tidak kosong |
| variabel output tidak terisi di basis | lupa `*n = ...` di cabang basis | setiap cabang harus mengisi parameter output |

---

## 15. Cheat sheet

```c
/* STRUKTUR: sama dengan list berkait biasa */
typedef Address List;          /* list = alamat node pertama; kosong = NIL */

/* PRIMITIF */
isEmpty(l)        /* l == NIL */
isOneElmt(l)      /* l != NIL && NEXT(l) == NIL */
head(l)           /* INFO(l): hanya untuk list tidak kosong */
tail(l)           /* NEXT(l): BERBAGI node, bukan salinan */
konso(e, l)       /* e di depan; hasil HARUS ditampung */
konsDot(l, e)       /* e di belakang */

/* TEMPLATE FUNGSI (hanya membaca) */
T f(List l) {
    if (isEmpty(l)) return BASIS;
    else return GABUNG(head(l), f(tail(l)));
}

/* TEMPLATE FUNGSI (basis 1, list tidak kosong) */
T f(List l) {
    if (isOneElmt(l)) return ...head(l)...;
    else return GABUNG(head(l), f(tail(l)));
}

/* TEMPLATE FUNGSI YANG MEMBUAT LIST BARU */
List f(List l) {
    if (isEmpty(l)) return NIL;
    else return konso(...head(l)..., f(tail(l)));
}

/* TEMPLATE PROSEDUR YANG MENGUBAH LIST */
void p(List *l, ...) {
    if (isEmpty(*l)) { ... }
    else {
        p(&NEXT(*l), ...);      /* &NEXT(*l): alamat "kepala" sisa list */
        ... *l = ... ...        /* mengubah pointer milik level di atas */
    }
}

/* TEMPLATE AKUMULATOR */
void fAcc(List l, T acc, T *hasil) {
    if (isEmpty(l)) *hasil = acc;
    else fAcc(tail(l), OLAH(acc, head(l)), hasil);
}
```

**Cara cepat memilih bentuk:**

- Hasilnya **angka** → basis `0` (atau `1` untuk perkalian), gabung dengan `+`
- Hasilnya **boolean** → basis `false` (ada?) atau `true` (semua?), gabung dengan `||` / `&&`
- Hasilnya **list baru** → basis `NIL`, gabung dengan `konso`
- **Mengubah list** → `List *l`, rekursi ke `&NEXT(*l)`
- Prasyarat **tidak kosong** → basis `isOneElmt`
- Butuh urutan **terbalik** → proses sesudah pemanggilan rekursif

---

## 16. Latihan soal

Untuk mengetes, pakai kode di [lampiran](#18-lampiran-kode-lengkap-untuk-dites). Kerjakan sendiri dulu sebelum membuka jawaban. Semua jawaban sudah dites.

### Level 1: hanya membaca

**1. `int sumList(List l)`**
Jumlah semua elemen. List kosong menghasilkan 0.
```
sumList([3,5,7]) = 15
sumList([])      = 0
```

**2. `int countX(List l, ElType x)`**
Banyaknya elemen yang bernilai `x`.
```
countX([1,2,1,1], 1) = 3
countX([], 1)        = 0
```

**3. `ElType maxList(List l)`**
Elemen terbesar. **Prasyarat: list tidak kosong.**
```
maxList([3,9,2])    = 9
maxList([-5,-2,-8]) = -2
maxList([4])        = 4
```

**4. `ElType lastElmt(List l)`**
Elemen terakhir. **Prasyarat: list tidak kosong.**
```
lastElmt([3,5,7]) = 7
lastElmt([4])     = 4
```

**5. `ElType getElmt(List l, int idx)`**
Elemen ke-`idx` (mulai dari 0). **Prasyarat: `0 ≤ idx < length(l)`.**
```
getElmt([3,5,7], 0) = 3
getElmt([3,5,7], 2) = 7
```

<details>
<summary>Jawaban level 1</summary>

```c
int sumList(List l) {
    if (isEmpty(l)) {
        return 0;
    } else {
        return head(l) + sumList(tail(l));
    }
}

int countX(List l, ElType x) {
    if (isEmpty(l)) {
        return 0;
    } else {
        return (head(l) == x ? 1 : 0) + countX(tail(l), x);
    }
}

ElType maxList(List l) {
    ElType m;
    if (isOneElmt(l)) {
        return head(l);                   /* basis 1 */
    } else {
        m = maxList(tail(l));
        return head(l) > m ? head(l) : m;
    }
}

ElType lastElmt(List l) {
    if (isOneElmt(l)) {
        return head(l);
    } else {
        return lastElmt(tail(l));
    }
}

ElType getElmt(List l, int idx) {
    if (idx == 0) {
        return head(l);                   /* basis: indeks 0 adalah head */
    } else {
        return getElmt(tail(l), idx - 1); /* elemen ke-idx dari l = elemen ke-(idx-1) dari tail */
    }
}
```

Catatan:
- `maxList`: kalau basisnya `isEmpty → 0`, hasil untuk `[-5,-2,-8]` akan `0`, padahal seharusnya `-2`. Sama dengan jebakan `maxTree` di pohon biner.
- `maxList`: hasil rekursi disimpan di `m` supaya `maxList(tail(l))` tidak dipanggil dua kali. Kalau ditulis `head(l) > maxList(tail(l)) ? head(l) : maxList(tail(l))`, setiap level memanggil rekursi dua kali, dan waktunya meledak secara eksponensial.
- `getElmt`: basisnya bukan list kosong, melainkan `idx == 0`. Prasyarat menjamin list tidak pernah habis sebelum `idx` mencapai 0.

</details>

### Level 2: predikat dan perbandingan

**6. `boolean isSorted(List l)`**
`true` jika list terurut menaik (tidak menurun). List kosong dan satu elemen dianggap terurut.
```
isSorted([1,2,2,5]) = true
isSorted([1,3,2])   = false
isSorted([])        = true
isSorted([7])       = true
```

**7. `boolean isEqual(List l1, List l2)`**
`true` jika kedua list punya panjang sama dan elemen yang seposisi sama.
```
isEqual([1,2], [1,2])   = true
isEqual([1,2], [1,2,3]) = false
isEqual([], [])         = true
isEqual([1,2], [2,1])   = false
```

**8. `void gandakan(List l)`**
Kalikan dua setiap elemen list, **langsung di node-node yang ada**.
```
l = [1,-2,3];  gandakan(l);  →  l = [2,-4,6]
```
*Pertanyaan:* kenapa parameternya cukup `List l`, bukan `List *l`?

<details>
<summary>Jawaban level 2</summary>

```c
boolean isSorted(List l) {
    if (isEmpty(l) || isOneElmt(l)) {
        return true;
    } else {
        return head(l) <= head(tail(l)) && isSorted(tail(l));
    }
}

boolean isEqual(List l1, List l2) {
    if (isEmpty(l1) && isEmpty(l2)) {
        return true;                    /* sama-sama habis */
    } else if (isEmpty(l1) || isEmpty(l2)) {
        return false;                   /* satu habis duluan: panjang beda */
    } else {
        return head(l1) == head(l2) && isEqual(tail(l1), tail(l2));
    }
}

void gandakan(List l) {
    if (!isEmpty(l)) {
        INFO(l) = 2 * INFO(l);
        gandakan(tail(l));
    }
}
```

Catatan:
- `isSorted` membandingkan `head(l)` dengan `head(tail(l))`, jadi butuh **dua** elemen. Karena itu basisnya mencakup list kosong **dan** satu elemen.
- `isEqual` mirip `samaBentuk` di latihan pohon biner: pola matching pada dua struktur sekaligus.
- `gandakan` cukup `List l` karena **kepala list tidak berubah**. Yang berubah hanya isi `info` di dalam node, dan node-node itu bisa diakses lewat salinan pointer `l`. `List *l` hanya diperlukan kalau **pointer kepala list** bisa berubah (menghapus/menyisip di depan).

</details>

### Level 3: membuat list baru

**9. `List inverse(List l)`**
Hasilkan list baru yang urutannya terbalik. List asli tidak berubah.
```
inverse([1,2,3]) = [3,2,1]
inverse([])      = []
```
Buat dua versi: (a) dengan `konsDot`, (b) dengan akumulator.

**10. `List takeN(List l, int n)`**
Salinan `n` elemen pertama. Kalau list lebih pendek dari `n`, salin semuanya.
```
takeN([3,5,7], 2) = [3,5]
takeN([3,5,7], 5) = [3,5,7]
takeN([3,5,7], 0) = []
```

**11. `void splitPosNeg(List l, List *pos, List *neg)`**
Pisahkan elemen `≥ 0` ke `pos` dan elemen `< 0` ke `neg`, dengan urutan asli tetap. Keduanya list baru.
```
splitPosNeg([3,-1,0,-7,4])  →  pos = [3,0,4], neg = [-1,-7]
splitPosNeg([])             →  pos = [],      neg = []
```

<details>
<summary>Jawaban level 3</summary>

```c
/* (a) dengan konsDot: balik sisanya, lalu taruh head di BELAKANG */
List inverse(List l) {
    if (isEmpty(l)) {
        return NIL;
    } else {
        return konsDot(inverse(tail(l)), head(l));
    }
}

/* (b) dengan akumulator: setiap head dipindah ke DEPAN akumulator */
static List inverseAcc(List l, List acc) {
    if (isEmpty(l)) {
        return acc;
    } else {
        return inverseAcc(tail(l), konso(head(l), acc));
    }
}
List inverse2(List l) {
    return inverseAcc(l, NIL);
}

List takeN(List l, int n) {
    if (n == 0 || isEmpty(l)) {
        return NIL;
    } else {
        return konso(head(l), takeN(tail(l), n - 1));
    }
}

void splitPosNeg(List l, List *pos, List *neg) {
    if (isEmpty(l)) {
        *pos = NIL;
        *neg = NIL;
    } else {
        splitPosNeg(tail(l), pos, neg);   /* pisahkan sisa dulu */
        if (head(l) >= 0) {
            *pos = konso(head(l), *pos);  /* lalu taruh head di depan list yang sesuai */
        } else {
            *neg = konso(head(l), *neg);
        }
    }
}
```

Catatan:
- `inverse` (a) mudah dibaca, tapi `konsDot` berjalan sampai ujung list setiap kali, jadi totalnya O(n²). Versi (b) hanya O(n), karena `konso` selalu bekerja di depan. Ini sama dengan `balik` memakai `foldl` di `penjelasanLambda.md`.
- Jejak `inverseAcc([1,2,3], [])`: `([2,3], [1])` → `([3], [2,1])` → `([], [3,2,1])` → hasil `[3,2,1]`.
- `splitPosNeg`: rekursi dijalankan **dulu**, baru `head` ditaruh di depan. Akibatnya urutan asli tetap terjaga. Kalau dibalik (taruh dulu, rekursi kemudian), kamu harus memakai `konsDot`.
- `takeN` punya **dua** kondisi basis: `n` habis atau list habis.

</details>

### Level 4: mengubah list di tempat

**12. `void insertSorted(List *l, ElType x)`**
Sisipkan `x` ke list yang sudah terurut menaik, sehingga list tetap terurut. Duplikat tetap disimpan. (Ini versi rekursif dari `ArsipTerurut` Praktikum 8.)
```
sisipkan 8, 3, 8, -2, 5 ke list kosong  →  [-2,3,5,8,8]
```

**13. `void deleteFirstX(List *l, ElType x)`**
Hapus **kemunculan pertama** `x` saja. Kalau tidak ada, list tidak berubah.
```
deleteFirstX([1,2,3,2], 2)  →  [1,3,2]
deleteFirstX([5], 5)        →  []
deleteFirstX([1,3], 9)      →  [1,3]
```

<details>
<summary>Jawaban level 4</summary>

```c
void insertSorted(List *l, ElType x) {
    Address p;
    if (isEmpty(*l) || x < INFO(*l)) {
        /* posisi ditemukan: sisip di depan "kepala" ini */
        p = newNode(x);
        if (p != NIL) {
            NEXT(p) = *l;
            *l = p;
        }
    } else {
        insertSorted(&NEXT(*l), x);    /* belum saatnya: turun ke sisa list */
    }
}

void deleteFirstX(List *l, ElType x) {
    Address p;
    if (isEmpty(*l)) {
        /* x tidak ditemukan */
    } else if (INFO(*l) == x) {
        p = *l;
        *l = NEXT(*l);
        free(p);
    } else {
        deleteFirstX(&NEXT(*l), x);
    }
}
```

Catatan:
- Bandingkan `insertSorted` dengan `sisipUrut` iteratif di Praktikum 8. Versi iteratif butuh **dua kasus** (sisip di depan vs. sisip setelah `prec`) dan loop pencarian `prec`. Versi rekursif hanya punya **satu** cara menyisip, yaitu "sisip di depan list ini". Trik `&NEXT(*l)` membuat setiap posisi di tengah list terlihat seperti "depan list" bagi level rekursi yang sedang berjalan.
- `deleteFirstX` berbeda dari `deleteX`: begitu ketemu, langsung hapus dan **berhenti** (basis 1). Tidak ada pemanggilan rekursif lagi. Pemeriksaan dilakukan **sebelum** rekursi, karena kita ingin kemunculan **pertama**.

</details>

---

## 17. Pembahasan soal Post Praktikum 9 (2025)

Bagian ini membahas empat soal Post Praktikum 9 tahun lalu (file `soal (3).pdf` di folder ini), beserta jawaban yang dikumpulkan saat itu. Semua soal memakai ADT `list.h` dan `list.c` di folder ini.

| No | File | Inti soal | Nilai tahun lalu |
|---|---|---|---|
| 1 | `duplicates.c` | list terurut: hapus **semua** elemen yang punya duplikat | 100 |
| 2 | `merge-sort.c` | merge sort rekursif, cetak list setiap selesai merging | tidak dijawab |
| 3 | `rotate-right.c` | rotasi list ke kanan sebanyak `k`, list asli tidak boleh berubah | 90 (runtime error di test 1) |
| 4 | `duplicate.c` | hapus duplikat, sisakan kemunculan **terakhir** | 100 |

Semua jawaban di bawah sudah dites dengan `list.c` asli. Selain contoh soal, setiap soal juga diuji dengan **3000 list acak** yang hasilnya dibandingkan dengan jawaban yang dihitung secara brute force.

> ⚠️ **`list.h` di folder ini rusak.** Baris terakhirnya tertulis `#endifm`, bukan `#endif`. Akibatnya `list.c` gagal dikompilasi dengan error `invalid preprocessing directive #endifm` dan `unterminated #ifndef`. Perbaiki baris itu dulu sebelum mencoba soal-soal di bawah. Kemungkinan ini hanya salah ketik saat menyalin, karena `merge-sort.c` berisi salinan `list.h` yang benar.

---

### Soal 1: `duplicates.c` (hapus semua yang punya duplikat)

#### Soal

Diberikan sebuah **list terurut** berukuran N (N ≥ 0). Hapus **seluruh** elemen yang memiliki duplikat (muncul lebih dari sekali). Hasilnya adalah list yang hanya berisi elemen yang muncul **tepat sekali**. Gunakan ADT List Rekursif yang diberikan.

| Masukan | Keluaran |
|---|---|
| `[1, 2, 3, 3, 4, 4, 5]` | `[1, 2, 5]` |
| `[1, 1, 1]` | `[]` |

Perhatikan bahwa elemen yang punya duplikat dibuang **semuanya**, bukan disisakan satu. `3, 3` hilang dua-duanya.

#### Jawaban tahun lalu (nilai 100)

```c
List removeDuplicates(List l) {
    if (isEmpty(l)) {
        return NIL;
    } else {
        if (search(tail(l), head(l))) {
            deleteX(&l, head(l));
            return removeDuplicates(l);
        }
        else return konso(head(l), removeDuplicates(tail(l)));
    }
}
```

Cara kerjanya:
- Kalau `head(l)` masih muncul lagi di `tail(l)`, berarti elemen itu punya duplikat. Hapus **semua** kemunculannya dengan `deleteX`, lalu proses sisa list.
- Kalau tidak, elemen itu unik. Simpan dengan `konso`, lalu proses `tail(l)`.

Jawaban ini benar (lolos 3000 uji acak), tapi ada dua hal yang perlu kamu sadari:

1. **List asli ikut dirusak.** `deleteX(&l, ...)` membebaskan node milik list yang dikirim pemanggil. `l` di dalam fungsi hanyalah salinan pointer, jadi kalau node pertama ikut dihapus, variabel list milik `main` masih menunjuk ke node yang sudah di-`free`. Ini aman selama `main` tidak memakai list aslinya lagi. Kebetulan di soal ini `main` memang tidak memakainya.
2. **Tidak memanfaatkan sifat terurut.** `search` menelusuri seluruh sisa list di setiap langkah, jadi totalnya O(n²). Padahal list sudah terurut, sehingga duplikat **pasti bersebelahan**.

#### Versi yang disarankan

Karena list terurut, cukup bandingkan `head(l)` dengan elemen tepat sesudahnya:

```c
/* lewati semua elemen di awal l yang bernilai x */
static List lewatiSama(List l, ElType x) {
    if (!isEmpty(l) && head(l) == x) {
        return lewatiSama(tail(l), x);
    }
    return l;
}

List removeDuplicates(List l) {
    if (isEmpty(l)) {
        return NIL;
    } else if (!isEmpty(tail(l)) && head(tail(l)) == head(l)) {
        /* head punya kembaran tepat di sebelahnya: buang seluruh kelompoknya */
        return removeDuplicates(lewatiSama(l, head(l)));
    } else {
        /* head unik */
        return konso(head(l), removeDuplicates(tail(l)));
    }
}
```

Versi ini O(n) dan **tidak mengubah list asli**. `lewatiSama` hanya menggeser pointer tanpa membebaskan apa pun, dan semua node hasil dibuat baru dengan `konso`. Hasil tes: list asli `[1,2,3,3,4,4,5]` tetap utuh setelah fungsi dipanggil.

**Jejak `[1, 2, 3, 3, 4, 4, 5]`:**

| `head` | elemen sesudahnya | Keputusan | Lanjut ke |
|---|---|---|---|
| 1 | 2 | unik → simpan | `[2,3,3,4,4,5]` |
| 2 | 3 | unik → simpan | `[3,3,4,4,5]` |
| 3 | 3 | kembar → lewati semua `3` | `[4,4,5]` |
| 4 | 4 | kembar → lewati semua `4` | `[5]` |
| 5 | (kosong) | unik → simpan | `[]` |

Hasil: `[1,2,5]` ✔

**Hasil tes kasus tepi:**

| Input | Output |
|---|---|
| `[1,1,1]` | `[]` |
| `[]` | `[]` |
| `[7]` | `[7]` |
| `[1,1,2,3,3]` | `[2]` |
| `[2,2,5,5]` | `[]` |

Kondisi `!isEmpty(tail(l))` di cabang kedua **wajib** ada. Tanpa itu, `head(tail(l))` pada list satu elemen sama dengan `INFO(NIL)`, dan program crash. Ini jebakan yang sama dengan `NEXT(NEXT(p))` di Praktikum 8.

---

### Soal 2: `merge-sort.c` (tidak dijawab tahun lalu)

#### Soal

Diberikan bilangan bulat `n`, diikuti `n` bilangan bulat. Urutkan secara menaik dengan **merge sort rekursif**, memakai ADT List Rekursif.

**Format masukan:** satu bilangan `n` (2 ≤ n ≤ 100), lalu `n` bilangan bulat.
**Format keluaran:** tampilkan list **setiap kali satu tahap merging selesai**, dengan `displayList`, lalu newline.
**Keterangan:** indeks tengah pembagian list adalah `floor(L / 2)`, dengan `L` panjang list. Bagian kiri berisi `floor(L/2)` elemen pertama.

| Input | Output |
|---|---|
| `6`<br>`5 3 1 4 2 5` | `1 3 `<br>`1 3 5 `<br>`2 5 `<br>`2 4 5 `<br>`1 2 3 4 5 5 ` |
| `8`<br>`6 10 3 7 4 1 8 -1` | `6 10 `<br>`3 7 `<br>`3 6 7 10 `<br>`1 4 `<br>`-1 8 `<br>`-1 1 4 8 `<br>`-1 1 3 4 6 7 8 10 ` |

Ada spasi setelah setiap angka, karena `displayList` di `list.c` mencetak `"%d "`.

> File `merge-sort.c` di folder ini hanya berisi salinan `list.h`, bukan template soal. Jadi bentuk template aslinya (misalnya apakah `main` sudah disediakan) tidak diketahui. Jawaban di bawah adalah program lengkap.

#### Ide merge sort

Merge sort adalah algoritma yang **secara alami rekursif**:

1. **Basis:** list kosong atau satu elemen sudah pasti terurut.
2. **Bagi:** potong list menjadi bagian kiri (`floor(L/2)` elemen pertama) dan bagian kanan (sisanya).
3. **Urutkan** masing-masing bagian secara rekursif.
4. **Gabung (merge):** satukan dua list yang sudah terurut menjadi satu list terurut. **Cetak hasilnya di sini.**

Pohon pemanggilan untuk contoh 1. Angka di kanan adalah urutan pencetakan:

```
mergeSort [5,3,1,4,2,5]                      L=6, kiri 3 elemen
├── mergeSort [5,3,1]                        L=3, kiri 1 elemen
│   ├── mergeSort [5]        → basis
│   ├── mergeSort [3,1]                      L=2, kiri 1 elemen
│   │   ├── mergeSort [3]    → basis
│   │   ├── mergeSort [1]    → basis
│   │   └── merge [3] + [1]       = 1 3            (cetak 1)
│   └── merge [5] + [1,3]         = 1 3 5          (cetak 2)
├── mergeSort [4,2,5]
│   ├── mergeSort [4]        → basis
│   ├── mergeSort [2,5]
│   │   └── merge [2] + [5]       = 2 5            (cetak 3)
│   └── merge [4] + [2,5]         = 2 4 5          (cetak 4)
└── merge [1,3,5] + [2,4,5]       = 1 2 3 4 5 5    (cetak 5)
```

Urutan pencetakan mengikuti urutan **post-order**: kiri selesai dulu, lalu kanan, lalu gabungan keduanya. Basis tidak mencetak apa-apa, karena tidak ada proses merging di sana.

#### `merge`: inti algoritmanya

Merge adalah fungsi rekursif list yang indah. Bandingkan kepala kedua list, ambil yang lebih kecil, lalu merge sisanya:

```c
List merge(List l1, List l2) {
    if (isEmpty(l1)) {
        return copy(l2);                                  /* l1 habis: sisanya l2 */
    } else if (isEmpty(l2)) {
        return copy(l1);                                  /* l2 habis: sisanya l1 */
    } else if (head(l1) <= head(l2)) {
        return konso(head(l1), merge(tail(l1), l2));      /* ambil dari l1 */
    } else {
        return konso(head(l2), merge(l1, tail(l2)));      /* ambil dari l2 */
    }
}
```

Jejak `merge([1,3,5], [2,4,5])`:

| `head(l1)` | `head(l2)` | Ambil | Sisa |
|---|---|---|---|
| 1 | 2 | 1 dari l1 | `[3,5]`, `[2,4,5]` |
| 3 | 2 | 2 dari l2 | `[3,5]`, `[4,5]` |
| 3 | 4 | 3 dari l1 | `[5]`, `[4,5]` |
| 5 | 4 | 4 dari l2 | `[5]`, `[5]` |
| 5 | 5 | 5 dari l1 (`<=`) | `[]`, `[5]` |
| (habis) | | `copy([5])` | |

Hasil: `[1,2,3,4,5,5]` ✔

Ada dua basis karena ada dua list yang bisa habis duluan. Ini mirip `isEqual` di latihan 7.

#### Program lengkap

```c
#include "list.h"
#include <stdio.h>
#include <stdlib.h>

/* salinan m elemen pertama l */
List ambilDepan(List l, int m) {
    if (m == 0 || isEmpty(l)) {
        return NIL;
    } else {
        return konso(head(l), ambilDepan(tail(l), m - 1));
    }
}

/* salinan l tanpa m elemen pertama */
List buangDepan(List l, int m) {
    if (m == 0) {
        return copy(l);
    } else {
        return buangDepan(tail(l), m - 1);
    }
}

/* menggabungkan dua list terurut menjadi satu list terurut (list baru) */
List merge(List l1, List l2) {
    if (isEmpty(l1)) {
        return copy(l2);
    } else if (isEmpty(l2)) {
        return copy(l1);
    } else if (head(l1) <= head(l2)) {
        return konso(head(l1), merge(tail(l1), l2));
    } else {
        return konso(head(l2), merge(l1, tail(l2)));
    }
}

void freeList(List *l) {
    if (!isEmpty(*l)) {
        freeList(&NEXT(*l));
        free(*l);
        *l = NIL;
    }
}

/* mengirimkan salinan l yang terurut menaik; mencetak list setiap selesai merging */
List mergeSort(List l) {
    int mid;
    List kiri, kanan, kiriUrut, kananUrut, hasil;
    if (isEmpty(l) || isOneElmt(l)) {
        return copy(l);                  /* basis: sudah terurut, tidak dicetak */
    }
    mid = length(l) / 2;                 /* floor(L/2) karena pembagian int */
    kiri = ambilDepan(l, mid);
    kanan = buangDepan(l, mid);
    kiriUrut = mergeSort(kiri);
    kananUrut = mergeSort(kanan);
    hasil = merge(kiriUrut, kananUrut);
    displayList(hasil);                  /* cetak setiap tahap merging */
    printf("\n");
    freeList(&kiri);
    freeList(&kanan);
    freeList(&kiriUrut);
    freeList(&kananUrut);
    return hasil;
}

int main(void) {
    int n, x;
    List l = NIL, urut;
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%d", &x);
        l = konsDot(l, x);
    }
    urut = mergeSort(l);
    freeList(&l);
    freeList(&urut);
    return 0;
}
```

Hasil tes: kedua contoh soal cocok baris per baris, termasuk spasi di akhir. `n = 2` (`9 4` → `4 9 `) dan `n = 3` (`3 2 1` → `1 2 ` lalu `1 2 3 `) juga benar.

Beberapa catatan:

- **`mid = length(l) / 2`.** Pembagian bilangan bulat di C otomatis membulatkan ke bawah, jadi ini sudah `floor(L/2)`. Untuk `L = 3`, kiri berisi 1 elemen dan kanan 2 elemen, sesuai pohon di atas. Kalau pembagiannya terbalik (kiri yang lebih panjang), urutan dan isi baris output berbeda, dan jawaban dinilai salah.
- **Setiap fungsi menghasilkan list baru**, jadi semua hasil antara dibebaskan dengan `freeList`. `list.h` tidak menyediakan `freeList`, jadi fungsi ini ditulis sendiri (bentuknya sama dengan bagian 11).
- **Kenapa tidak berbagi node?** `buangDepan` bisa saja mengembalikan `tail` tanpa menyalin, karena itu lebih hemat. Tapi kalau begitu, membebaskan `kanan` akan ikut membebaskan node milik `l`. Dengan semua bagian berupa salinan, setiap list bisa dibebaskan tanpa saling mengganggu. Untuk `n ≤ 100`, biaya menyalin tidak berarti.
- **`displayList` dari `list.c`** dipakai sesuai perintah soal. Jangan membuat versi sendiri dengan format lain.

---

### Soal 3: `rotate-right.c` (rotasi ke kanan)

#### Soal

Rotasi list ke kanan sebanyak `k` posisi, artinya **`k` elemen terakhir dipindahkan ke depan**.

```
I.S.  : l terdefinisi, mungkin kosong; k >= 0
F.S.  : Mengembalikan list BARU hasil rotasi l sebanyak k posisi ke kanan.
        List asli l TIDAK dimodifikasi (pendekatan fungsional).
        Jika k lebih besar dari panjang list, gunakan k modulo panjang list.
```

| `l` | `k` | Hasil |
|---|---|---|
| `[1,2,3,4,5]` | 2 | `[4,5,1,2,3]` |
| `[1,2,3,4,5]` | 0 | `[1,2,3,4,5]` |
| `[1,2,3,4,5]` | 7 | `[4,5,1,2,3]` (karena 7 mod 5 = 2) |

#### Jawaban tahun lalu (nilai 90, runtime error di test 1)

```c
List rotateRight(List l, int k) {
    if (isEmpty(l)) return l;
    int kmod = k;
    if (k > 0) kmod = k % length(l) - length(l);
    if (kmod == 0) {
        return l;
    } else {
        return rotateRight(konsDot(tail(l), head(l)), kmod + 1);
    }
}
```

Idenya: rotasi ke kanan sebanyak `k` sama dengan rotasi ke **kiri** sebanyak `n - (k mod n)`. Setiap langkah rotasi kiri memindahkan `head` ke belakang dengan `konsDot(tail(l), head(l))`. Variabel `kmod` dibuat negatif sebagai penghitung mundur, lalu dinaikkan satu per langkah sampai 0.

Untuk input biasa, hasilnya benar (lolos 3000 uji acak). Tapi pengujian menunjukkan tiga masalah:

**1. List asli ikut berubah, padahal F.S. melarangnya.**

`konsDot(tail(l), head(l))` **menyambung node baru ke ujung list** milik `tail(l)`. Node-node itu masih menjadi bagian dari list asli. Hasil tes:

```
List asli sebelum dipanggil (k=2):  [1,2,3,4,5]
List asli sesudah dipanggil:        [1,2,3,4,5,1,2,3]
List asli sesudah dipanggil (k=5):  [1,2,3,4,5,1,2,3,4,5]
```

Setiap langkah rotasi menambah satu node di ujung rantai milik `l`. Ini contoh nyata peringatan di bagian 10: **`konsDot` versi fungsi tidak murni, karena mengubah list yang diberikan.**

**2. Hasil berbagi node dengan list asli.**

Hasil rotasi memakai node-node `2, 3, 4, 5` milik list asli, dan untuk `k = 0` yang dikembalikan adalah **pointer `l` itu sendiri**, bukan list baru. Kalau program penilai membebaskan list asli dan list hasil secara terpisah, node yang sama dibebaskan dua kali (*double free*), yang menyebabkan runtime error.

**3. Lambat: O(n²).**

Setiap langkah memanggil `konsDot`, yang berjalan sampai ujung list. Untuk `n` langkah, totalnya sekitar n² operasi. Hasil pengukuran (dikompilasi dengan `-O2`):

| `n` | `k` | Waktu |
|---|---|---|
| 1.000 | 1 | 0,00 detik |
| 5.000 | 1 | 0,05 detik |
| 20.000 | 1 | **1,01 detik** (lewat batas 1 detik) |

**Jadi apa penyebab runtime error di test 1?** Isi test case tidak terlihat, jadi tidak bisa dipastikan. Dugaan terkuat adalah masalah nomor 2: penilai memakai atau membebaskan list asli setelah memanggil `rotateRight`, padahal list itu sudah diubah dan nodenya dipakai bersama. Masalah nomor 3 tidak cocok dengan waktu error 0,08 detik, tapi tetap berbahaya untuk test case yang besar. Yang pasti, ketiga masalah ini melanggar spesifikasi, dan versi berikut menghindari semuanya.

#### Versi yang disarankan

Rotasi kanan sebanyak `k` sama dengan memotong list menjadi dua bagian, lalu menukar urutannya:

```
l = [1,2,3,4,5], k = 2, n = 5, potong = n - k = 3

    [1,2,3 | 4,5]
     awal    akhir
             ▲
       potong di sini

hasil = akhir ++ awal = [4,5] ++ [1,2,3] = [4,5,1,2,3]
```

```c
/* salinan m elemen pertama l, disambung ke lanjut */
static List salinDepan(List l, int m, List lanjut) {
    if (m == 0) {
        return lanjut;
    } else {
        return konso(head(l), salinDepan(tail(l), m - 1, lanjut));
    }
}

/* l tanpa m elemen pertama (berbagi node, tidak menyalin) */
static List lewatiN(List l, int m) {
    if (m == 0) {
        return l;
    } else {
        return lewatiN(tail(l), m - 1);
    }
}

List rotateRight(List l, int k) {
    int n, potong;
    List bagianAwal;
    if (isEmpty(l)) {
        return NIL;
    }
    n = length(l);
    k = k % n;                                    /* k bisa lebih besar dari n */
    potong = n - k;
    bagianAwal = salinDepan(l, potong, NIL);      /* salinan [1,2,3] */
    return salinDepan(lewatiN(l, potong), k, bagianAwal);   /* salinan [4,5] + [1,2,3] */
}
```

Cara kerjanya:

1. `bagianAwal = salinDepan(l, 3, NIL)` membuat **salinan** `[1,2,3]`.
2. `lewatiN(l, 3)` menunjuk ke node `4` di list asli. Fungsi ini hanya menggeser pointer, tidak mengubah apa pun.
3. `salinDepan(..., 2, bagianAwal)` menyalin `[4,5]`, lalu node `5` hasil salinan disambung ke `bagianAwal`.

Trik di `salinDepan` adalah parameter `lanjut`. Saat salinan sudah mencapai `m` elemen, ekor salinannya **langsung disambung ke `lanjut`**, bukan ke `NIL`. Dengan begitu tidak perlu `concat`, yang akan menyalin `bagianAwal` sekali lagi dan membuat salinan pertama bocor (tidak pernah di-`free`).

Hasil tes versi ini:

| Kasus | Hasil |
|---|---|
| `[1,2,3,4,5]`, k = 2 | `[4,5,1,2,3]` |
| `[1,2,3,4,5]`, k = 0 | `[1,2,3,4,5]` (list baru, bukan pointer yang sama) |
| `[1,2,3,4,5]`, k = 7 | `[4,5,1,2,3]` |
| `[1,2,3,4,5]`, k = 5 | `[1,2,3,4,5]` |
| `[1,2,3,4,5]`, k = 1 | `[5,1,2,3,4]` |
| `[]`, k = 3 | `[]` |
| `[9]`, k = 4 | `[9]` |
| 3000 list acak | hasil benar, **list asli tetap utuh**, **tidak ada node yang dipakai bersama** |
| `n = 20.000`, k = 1 | 0,00 detik |

`k = k % n` aman karena `n` dijamin tidak nol: list kosong sudah ditangani di awal. Kalau urutannya dibalik (modulo dihitung sebelum memeriksa list kosong), program akan membagi dengan nol.

---

### Soal 4: `duplicate.c` (sisakan kemunculan terakhir)

#### Soal

Diberikan bilangan bulat `n` dan list dengan `n` elemen. Hapus semua elemen duplikat **kecuali yang paling akhir**. Hasilnya dicetak dengan `displayList`.

| Input | Output |
|---|---|
| `5`<br>`1 2 1 3 1` | `2 3 1 ` |

Angka `1` muncul tiga kali, dan yang disisakan hanya yang terakhir (posisi ke-5). Urutan elemen lain tidak berubah.

#### Jawaban tahun lalu (nilai 100)

```c
List removeDuplicate(List l) {
    if (isEmpty(l)) return l;
    else {
        if (search(tail(l), head(l))) return removeDuplicate(tail(l));
        return konso(head(l), removeDuplicate(tail(l)));
    }
}
```

Jawaban ini bagus dan sudah tepat. Kuncinya ada di cara membaca soal:

> Sebuah elemen **disisakan** jika dan hanya jika ia **tidak muncul lagi** di sisa list sesudahnya.

Kalau `head(l)` masih muncul di `tail(l)`, berarti ada kemunculan yang lebih belakang, jadi `head` dibuang. Kalau tidak muncul lagi, berarti `head` adalah kemunculan terakhir, jadi `head` disimpan.

**Jejak `[1,2,1,3,1]`:**

| `head` | `tail` | Muncul lagi di `tail`? | Keputusan |
|---|---|---|---|
| 1 | `[2,1,3,1]` | ya | buang |
| 2 | `[1,3,1]` | tidak | **simpan** |
| 1 | `[3,1]` | ya | buang |
| 3 | `[1]` | tidak | **simpan** |
| 1 | `[]` | tidak | **simpan** |

Hasil: `[2,3,1]` ✔

Kasus lain yang dites: `[1,2]` → `[1,2]`, `[5,5,5]` → `[5]`, `[3,1,3,2,1]` → `[3,2,1]`, `[]` → `[]`. Uji 3000 list acak juga cocok semua.

Jawaban ini juga **tidak merusak list asli**: elemen yang disimpan dibuat ulang dengan `konso`, dan elemen yang dibuang hanya dilewati.

#### Variasi latihan: sisakan kemunculan **pertama**

Bagaimana kalau soalnya diubah menjadi "sisakan yang paling **awal**"? Contohnya `[1,2,1,3,1]` → `[1,2,3]`.

Sekarang `head(l)` **selalu** disimpan, karena ia pasti kemunculan pertama dari nilainya. Yang harus dibuang adalah kemunculan nilai yang sama **di sisa list**. Jadi sebelum memproses sisa list, hapus dulu semua `head(l)` dari `tail(l)`:

```c
/* salinan l tanpa semua elemen bernilai x */
static List hapusSemua(List l, ElType x) {
    if (isEmpty(l)) {
        return NIL;
    } else if (head(l) == x) {
        return hapusSemua(tail(l), x);
    } else {
        return konso(head(l), hapusSemua(tail(l), x));
    }
}

List removeDuplicateAwal(List l) {
    List sisa, hasil;
    if (isEmpty(l)) {
        return NIL;
    }
    sisa = hapusSemua(tail(l), head(l));            /* sisa tanpa kembaran head */
    hasil = konso(head(l), removeDuplicateAwal(sisa));
    while (sisa != NIL) {                           /* sisa hanya list sementara */
        Address q = sisa;
        sisa = NEXT(sisa);
        free(q);
    }
    return hasil;
}
```

Hasil tes: `[1,2,1,3,1]` → `[1,2,3]`, `[3,1,3,2,1]` → `[3,1,2]`, dan 3000 list acak cocok semua.

Perhatikan asimetrinya. "Sisakan yang terakhir" cukup **melihat ke depan** (apakah masih muncul di `tail`?), dan itu mudah karena list rekursif memang bergerak ke depan. "Sisakan yang pertama" butuh **mengingat yang sudah lewat**. Di sini caranya adalah membersihkan sisa list sebelum rekursi.

`hapusSemua` memakai `deleteX` versi fungsional: ia membuat salinan, bukan menghapus di tempat. Alasannya, `deleteX` dari `list.c` akan membebaskan node milik list asli.

---

### Pelajaran dari keempat soal

| Pelajaran | Muncul di |
|---|---|
| **Baca F.S. dengan teliti**: "list asli tidak dimodifikasi" berarti jangan pakai `deleteX` atau `konsDot` pada list masukan | soal 1, 3 |
| `konsDot` versi fungsi **mengubah** list yang diberikan | soal 3 |
| Mengembalikan `l` apa adanya berarti **berbagi node**, bukan membuat list baru | soal 3 |
| Manfaatkan informasi tambahan dari soal (list terurut → duplikat bersebelahan) | soal 1 |
| Cek `!isEmpty(tail(l))` sebelum `head(tail(l))` | soal 1 |
| Dua list masukan → dua basis (mana yang habis duluan) | soal 2 (`merge`) |
| Posisi pencetakan relatif terhadap rekursi menentukan urutan output | soal 2 |
| Pembagian `int` di C sudah membulatkan ke bawah | soal 2 |
| "Sisakan terakhir" = lihat ke depan; "sisakan pertama" = bersihkan ke depan | soal 4 |

---

## 18. Lampiran: kode lengkap untuk dites

Pakai file ADT yang ada di folder ini: `boolean.h`, `list.h`, dan `list.c`. Isinya sama dengan header di [bagian 4](#4-struktur-data-dan-header-listh), ditambah implementasi `displayList`, `length`, `search`, `deleteX`, `copy`, `mCopy`, `concat`, dan `mConcat`.

> ⚠️ Perbaiki dulu baris terakhir `list.h` dari `#endifm` menjadi `#endif`. Tanpa perbaikan ini, `list.c` tidak bisa dikompilasi.

Kode di bagian 7, 9, dan 10 memakai nama yang sama dengan fungsi yang sudah ada di `list.c` (`displayList`, `length`, `search`, dan seterusnya). Kode itu untuk dipelajari, jangan disalin ke file yang dikompilasi bersama `list.c`, karena akan terjadi *multiple definition*. Fungsi latihan di bagian 16 (`sumList`, `maxList`, dan lainnya) tidak bentrok dan bisa langsung dipakai.

### Contoh `main.c` untuk mencoba

```c
#include <stdio.h>
#include "list.h"

/* tempel fungsi yang ingin dites di sini, misalnya sumList dari latihan 1 */

int main(void) {
    List l = NIL;
    int n, x;
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%d", &x);
        l = konsDot(l, x);            /* bangun list sesuai urutan input */
    }
    printf("%d\n", sumList(l));
    return 0;
}
```

Compile dari PowerShell, di folder yang berisi `list.h`, `list.c`, dan `boolean.h`:

```bash
gcc -std=c99 -Wall -Wextra -o main main.c list.c
```

Untuk soal yang file jawabannya sudah berisi `main` (seperti `duplicate.c` dan `merge-sort.c`), cukup compile file itu bersama `list.c`:

```bash
gcc -std=c99 -Wall -Wextra -o duplicate duplicate.c list.c
```

Untuk soal yang file jawabannya **hanya** berisi fungsi (seperti `duplicates.c` dan `rotate-right.c`), buat `main.c` terpisah yang memanggil fungsi itu, lalu compile ketiganya:

```bash
gcc -std=c99 -Wall -Wextra -o rotate main.c rotate-right.c list.c
```

Simpan input di `input.txt`, lalu jalankan lewat `cmd` supaya tidak kena masalah BOM PowerShell:

```bash
cmd /c "duplicate.exe < input.txt"
```

---

> 💡 **Ringkasan satu kalimat:** list rekursif adalah list berkait yang dipandang sebagai "kosong, atau head + list lain". Setiap fungsi cukup mengurus `head` lalu memercayakan sisanya ke pemanggilan rekursif pada `tail`, dan setiap prosedur yang mengubah list cukup memberikan `&NEXT(*l)` ke pemanggilan rekursif berikutnya.
