# Catatan Perubahan GUI (paket `algeo.gui`)

Dokumen belajar untuk demo. Isinya: apa yang ada di tiap berkas, method yang dipakai, dan alasan di balik tiap keputusan.

---

## 1. Gambaran Besar

GUI adalah klien baru yang setara dengan CLI. Seluruh kode ada di `src/main/java/algeo/gui/` (12 berkas, satu paket datar). Engine (`matrix`, `solver`, `interpolation`, `regression`) tidak diubah.

**Aturan yang dipegang selama mengerjakan:**
- Hanya boleh menambah berkas di `algeo/gui/`. `cli`, engine, `App.java`, dan `pom.xml` tidak disentuh.
- `gui` boleh mengimpor engine, `io`, dan `utils`. `gui` tidak boleh mengimpor `cli`.
- Hanya Swing dan JDK. Tidak ada pustaka baru.
- `Locale.setDefault(Locale.US)` di `GuiApp.main`. Angka ditampilkan lewat `Format.decimals`.
- Nol dibandingkan dengan `EPSILON`, bukan `== 0`. Ini sudah diurus engine dan kode yang disalin dari CLI.
- Semua identifier dan komentar berbahasa Inggris. Teks yang tampil ke pengguna berbahasa Indonesia.

**Cara menjalankan (masa pengembangan):**
- `mvn -q compile`
- `java -cp target/classes algeo.gui.GuiApp`
- `mvn exec:java -Dexec.mainClass=algeo.gui.GuiApp` TIDAK bisa, karena `pom.xml` mengunci `mainClass` ke `algeo.App`.

**Alur satu perhitungan:**
1. Pengguna mengisi `MatrixInputPanel` atau `PointsInputPanel`, lalu menekan Hitung.
2. Panel fitur membaca isian dan memanggil `compute(...)` (method static murni).
3. `compute` memanggil `InputReader` untuk mengubah teks menjadi `Matrix` atau titik, lalu memanggil engine bersama `StepLog`.
4. `compute` mengembalikan `ResultPanel.Report` (fitur, metode, masukan, keluaran) atau melempar exception.
5. Panel menyerahkan `Report` ke `ResultPanel` untuk ditampilkan. Exception berubah menjadi dialog.
6. Tombol Simpan hasil menulis `Report` ke `.txt` dengan format yang sama dengan CLI.

---

## 2. Keputusan Desain Lintas Berkas

| Keputusan | Alasan |
|---|---|
| `compute` static murni di tiap panel (tanpa tipe Swing) | Bisa dijalankan di thread lain dan bisa diuji tanpa membuka jendela. Kode Swing hanya merangkai. |
| `ResultPanel.Report` (record 4 String) | Satu bentuk hasil untuk semua panel. Isinya sekaligus bahan berkas simpan. |
| Teks pemformat disalin dari menu CLI ke panel | `gui` dilarang mengimpor `cli`. Duplikasi dijaga dengan pengujian terhadap menu CLI asli. |
| Format keluaran mengikuti CLI persis | Aturan tugas: masukan sama harus menghasilkan keluaran sama. Contoh: judul `Hasil:` hanya di SPL, tidak di Determinan dan Invers. |
| `SwingWorker` hanya di panel matriks | Matriks bisa besar (file sampai 1001). Tanpa worker jendela membeku. |
| Panel titik berjalan langsung di thread UI | Maksimal 10 titik dan 5 knot, hanya sekitar 1 sampai 2 milidetik. Worker cuma menambah kode. |
| Snapshot untuk prediksi | Prediksi harus memakai model yang sama dengan persamaan di layar, walau isian diedit setelah Hitung. |
| Spline dan Regresi: satu prediksi (diganti tiap klik). Polinomial: prediksi bertumpuk | CLI hanya bertanya sekali untuk spline dan regresi, tetapi berulang untuk polinomial. Dengan begitu berkas simpan selalu bentuk yang bisa dibuat CLI. |
| File dibaca lewat `InputReader.readFileText`, bukan `FileHandler.bacaFile*` | `FileHandler` mencetak galat ke konsol dan mengembalikan `null`, jadi pesannya tidak sampai ke pengguna GUI. |
| Batas ukuran ditegakkan di GUI sendiri | Engine tidak memeriksa batas. Di CLI batas ada di `ConsoleIO` dan `FileHandler`. |
| Satu `JFileChooser` statis, dibuat saat pertama dipakai | Membuatnya lambat di Windows (terukur 1,1 sampai 1,3 detik untuk yang pertama, sekitar 0,35 detik tiap tambahan). Satu instance juga mengingat folder terakhir di semua tab. |
| Hasil lama dikosongkan bila Hitung gagal | Supaya tidak ada hasil yang tidak cocok dengan masukan terbaru yang bisa tersimpan. |
| Prediksi yang gagal tidak menghapus hasil | Persamaan tetap valid. Sama seperti CLI yang lanjut ke iterasi berikutnya. |

---

## 3. Log per Berkas

### 3.1 `GuiApp.java`
**Isi:** kelas `final` dengan konstruktor private (kelas utilitas, tidak untuk dibuat objeknya).

**Method:**
- `main(String[] args)`: memanggil `Locale.setDefault(Locale.US)` lebih dulu, lalu `useSystemLookAndFeel()`, lalu `SwingUtilities.invokeLater(() -> new MainFrame().setVisible(true))`.
- `useSystemLookAndFeel()`: memasang tampilan bawaan sistem (`UIManager.getSystemLookAndFeelClassName()`). Bila gagal, tampilan standar Swing tetap dipakai.

**Alasan:**
- `Locale.US` supaya komputer berbahasa Indonesia tidak mencetak `3,500`.
- `invokeLater` karena komponen Swing harus dibuat di thread khusus UI (Event Dispatch Thread).
- Gagal memasang tampilan sistem tidak boleh menghentikan program, jadi galatnya diabaikan.

**Riwayat:** dibuat di langkah pertama bersama `MainFrame` sampai jendela kosong bisa dijalankan.

### 3.2 `MainFrame.java`
**Isi:** `extends JFrame`. Judul "Kalkulator Aljabar Linier". Enam tab dalam `JTabbedPane`.

**Method:** konstruktor.
- `setDefaultCloseOperation(EXIT_ON_CLOSE)`.
- Tab berurutan: SPL (`SplPanel`), Determinan (`DeterminantPanel`), Invers (`InversePanel`), Interpolasi Polinomial (`PolynomialPanel`), Spline Kubik Natural (`SplinePanel`), Regresi Spline (`RegressionPanel`).
- Ukuran 1000 x 700, ukuran minimum 800 x 600, dibuka di tengah layar.

**Alasan:** satu tab per fitur memudahkan pengguna berpindah tanpa menu bertingkat. Nama tab bisa diubah kapan saja lewat argumen pertama `addTab`, dan itu tidak mengubah judul `Fitur:` di berkas simpan (konstanta terpisah di tiap panel).

**Riwayat:** awalnya enam tab berisi placeholder "(belum dibuat)". Tiap panel dipasang satu per satu dengan mengganti satu baris `addTab`.

### 3.3 `InputReader.java`
**Isi:** kelas utilitas tanpa Swing. Konstanta `MAX_POINTS = 10`. Enum `Source { TYPED(11), FILE(1001) }`.

**Method:**
- `Source.maxRows()` dan `Source.maxCols(boolean augmented)`: batas baris dan kolom. Kolom konstanta `b` tidak dihitung sebagai dimensi utama, jadi matriks augmented boleh satu kolom lebih lebar (11 x 12 untuk ketikan, 1001 x 1002 untuk file).
- `readFileText(File)`: membaca file jadi `String`, membuang tanda BOM UTF-8 di awal, dan mengubah kegagalan baca menjadi `IllegalArgumentException` berpesan jelas.
- `parseMatrix(String text, Source source, boolean augmented)`: memecah teks per baris (`split("\\R")`), melewati baris kosong, memanggil `InputParser.parseBarisAngka` per baris, memeriksa batas kolom, kesamaan lebar dengan baris pertama, dan batas baris. Setiap galat memuat nomor baris ("Baris 2: ...").
- `fitsInTextArea(Matrix, boolean augmented)`: benar bila ukuran matriks masih di bawah batas ketik. Dipakai `MatrixInputPanel` untuk memutuskan apakah isi file boleh tampil.
- `parsePoints(String)`: tiap baris harus tepat 2 angka. Maksimal 10 titik. Titik dikumpulkan dulu, baru dicek, supaya pesannya bisa menyebut total ("input berisi 11 titik"). Mengembalikan `FileHandler.HasilTitikSampel` (tipe yang sudah ada di `io`).
- `parseNumberList(String)`: satu baris angka, dipakai untuk knot. Kosong menghasilkan array kosong. Jumlah knot K sama dengan banyaknya angka.
- `parseNumber(String text, String name)`: satu angka (nilai x). Kosong menghasilkan pesan "<name> tidak boleh kosong."
- Private: `requireNotBlank`, `parseLine` (menambah awalan "Baris k:"), `limitHint` (menyarankan Buka file bila ketikan terlalu besar).

**Alasan:** semua pemeriksaan masukan ada di satu tempat, bebas Swing, jadi bisa diuji sendiri. Engine tidak punya batas, dan `FileHandler` menelan pesan galat.

**Riwayat:**
- Dibuat dengan kerangka lalu diisi. Awalnya batas file 1001 x 1001.
- Diperbaiki sesuai spesifikasi butir 7: augmented dari file boleh 1001 x 1002.
- Satu kata diubah: "masukan berisi" menjadi "input berisi" agar sama dengan pesan CLI. Selisih ini ketahuan lewat perbandingan dengan `RegressionMenu`.

### 3.4 `MatrixInputPanel.java`
**Isi:** `extends JPanel`. Kotak teks monospasi, tombol "Buka file..." dan "Bersihkan", label petunjuk. Dipakai oleh SPL, Determinan, Invers. Konstruktor `MatrixInputPanel(boolean augmented)`.

**Method:**
- `getText()`: teks yang dipakai untuk dihitung. Bila ada file besar, teks file dikembalikan, bukan isi kotak.
- `getSource()`: `FILE` hanya saat file besar sedang dimuat. Selain itu `TYPED`.
- `isAugmented()`.
- Private: `buildHint`, `openFile` (dialog + penanganan galat), `loadFile(File)` (baca, validasi dengan batas file, lalu putuskan tampil atau ringkasan), `showTypedText`, `showLargeFileSummary`, `clear`, `setLocked`.

**Alasan:**
- File divalidasi saat dibuka, jadi galatnya muncul sebelum Hitung ditekan.
- Matriks file yang lebih besar dari batas ketik tidak ditampilkan. Kotak hanya menampilkan nama berkas dan ukurannya, dan terkunci (abu-abu, tidak bisa diedit) sampai Bersihkan ditekan. Teks sebesar jutaan angka tidak cocok untuk `JTextArea`.
- File kecil (sampai 11 x 11 atau 11 x 12) masuk kotak sebagai ketikan biasa dan tetap bisa diedit.
- `loadFile` dipisah dari `openFile` supaya bisa diuji tanpa dialog.

**Riwayat:** `JFileChooser` awalnya dibuat di dalam panel. Dipindah ke `ResultPanel` sebagai satu instance bersama (lihat 3.6).

### 3.5 `PointsInputPanel.java`
**Isi:** kotak teks titik "x y" per baris, tombol "Buka file..." dan "Bersihkan". Dipakai oleh Polinomial, Spline, Regresi.

**Method:**
- `getText()`.
- `isFromFile()`: benar hanya selama teks persis hasil muat file. `DocumentListener` mereset penanda ini saat pengguna mengetik atau menekan Bersihkan.
- `static buildInputText(double[] x, double[] y, boolean fromFile)`: menyusun bagian "Input:" berkas simpan. Persis `stringInput` di CLI: "Titik Data (N=n):" bila diketik, "Titik Data (dari file):" bila dari file, lalu satu baris `x<TAB>y` per titik (angka lewat `Double.toString`, urutan asli, tidak diurutkan).
- Private: `onTextEdited`, `openFile`, `loadFile`.

**Alasan:** di CLI, bagian Input berkas simpan berbeda untuk titik ketikan dan titik file, jadi panel perlu tahu asalnya. Titik maksimal 10, sehingga isi file selalu muat di kotak dan tidak perlu mode terkunci.

### 3.6 `ResultPanel.java`
**Isi:** area hasil (hanya baca, monospasi), tombol "Simpan hasil...", label status. Memuat `record Report(feature, method, input, output)` dan satu `JFileChooser` statis.

**Method instance:**
- `showBusy()`: status "Menghitung...", Simpan nonaktif.
- `showReport(Report)`: menampilkan hasil, Simpan aktif.
- `showReportAtEnd(Report)`: sama, lalu menggulir ke baris terakhir (dipakai setelah prediksi).
- `showFailure(Throwable)`: mengosongkan hasil lalu menampilkan dialog galat.
- `showError(Throwable)`: hanya dialog, hasil di layar tetap (dipakai untuk prediksi yang gagal).
- `clear()`.
- Private: `saveReport`, `saveTo(File)` (memakai `FileHandler.tulisHasilKeFile`).

**Method static:**
- `buildFileContent(Report)`: isi berkas simpan: "Fitur: ...", "Metode: ...", "Input:", "Hasil:", persis `ConsoleIO.offerSaveToFile`.
- `withTxtExtension(String)`: menambah `.txt` bila belum berakhiran `.txt`.
- `describeError(Throwable)`: `OutOfMemoryError` menjadi pesan memori. `IllegalArgumentException` (termasuk `NumberFormatException`) dan `ArithmeticException` memakai pesan aslinya. Lainnya menjadi "Terjadi kesalahan tak terduga: ...".
- `chooseFileToOpen(Component)`, `chooseFileToSave(Component)`: dialog berkas. Saat menyimpan, `.txt` ditambahkan otomatis, dan bila berkas sudah ada muncul konfirmasi Timpa/Batal (default Batal). Memilih Batal kembali ke dialog simpan.
- Private: `sharedFileChooser`, `confirmOverwrite`.

**Alasan:** semua urusan hasil dan berkas ada di satu kelas (menggantikan `SaveFormat` dan `FileDialogs` yang semula direncanakan terpisah). `showError` dan `showFailure` dibedakan karena prediksi yang salah tidak boleh menghapus persamaan, sedangkan Hitung yang gagal harus menghapus hasil lama.

**Riwayat:**
- Awal: kerangka berisi `JFileChooser` bersama dan `chooseFileToOpen`.
- Diperluas: `Report`, tampilan hasil, simpan, `.txt` otomatis, konfirmasi timpa.
- Ditambah `showError` dan `showReportAtEnd` saat panel prediksi dibuat.

### 3.7 `SplPanel.java`
**Isi:** `enum Method { GAUSS, GAUSS_JORDAN, INVERSE, CRAMER }` (label persis menu CLI), `MatrixInputPanel(true)` (augmented), pilihan metode, tombol Hitung, label peringatan Cramer.

**Method:**
- `static compute(String text, InputReader.Source source, Method method)`: membaca matriks augmented, memanggil `SPLSolver.solveGauss/solveGaussJordan/solveInverse/solveCramer` dengan `StepLog`, lalu menyusun `"Langkah:\n" + log + "\n\nHasil:\n" + hasil`.
- Private static (disalin dari `SPLMenu`): `renderResult`, `buildParamSymbols` (variabel bebas dinamai t, s, a3, a4, ...), `formatLinearCombo` (koefisien 1 ditulis tanpa angka).
- Private: `showWarningIfCramer`, `calculate` (memakai `SwingWorker`).

**Alasan:** tanpa solusi menghasilkan "Solusi tidak ada."; tak hingga menghasilkan bentuk parametrik. Pesan galat engine dipakai apa adanya (matriks non-persegi atau singular pada metode balikan dan Cramer). Peringatan Cramer ada karena Cramer menghitung n + 1 determinan: 100 x 100 memerlukan sekitar 43,6 detik, sedangkan Gauss di bawah 1 detik.

### 3.8 `DeterminantPanel.java`
**Isi:** `enum Method { OBE, COFACTOR }` ("Reduksi Baris (OBE)", "Ekspansi Kofaktor"), `MatrixInputPanel(false)`, label peringatan kofaktor.

**Method:**
- `static compute(...)`: menolak matriks non-persegi dengan pesan CLI "Matriks tidak memiliki determinan", lalu memanggil `Determinant.OBE` atau `Determinant.cofactor` dengan `StepLog`. Hasil: `"Langkah:\n" + log + "\n\n" + "Determinan = ..."` (tanpa judul `Hasil:`). Bila log kosong (kofaktor 1 x 1 atau 2 x 2), hanya hasil.
- Private: `showWarningIfCofactor`, `calculate` (`SwingWorker`).

**Alasan:** peringatan kofaktor karena waktunya naik sekitar 10 kali lipat per tambahan ukuran: 10 x 10 sekitar 3,3 detik, 11 x 11 (batas ketik) sekitar 34,5 detik, sedangkan OBE 11 x 11 sekitar 0,001 detik. Perhitungan tidak bisa dibatalkan di tengah jalan.

### 3.9 `InversePanel.java`
**Isi:** `enum Method { GAUSS_JORDAN, ADJOINT }` ("Augmentasi [A|I] + Gauss-Jordan", "Metode Adjoin"), `MatrixInputPanel(false)`, label peringatan adjoin.

**Method:**
- `static compute(...)`: memanggil `Inverse.gaussJordan` atau `Inverse.adjoint`. Bila engine melempar `ArithmeticException` (matriks singular), `compute` melempar ulang dengan teks tetap CLI "Matriks tidak memiliki balikan". Hasil: `"Langkah:\n" + log + "\n\nMatriks Balikan (Invers):\n" + matriks`.
- Private: `showWarningIfAdjoint`, `calculate` (`SwingWorker`).

**Alasan:** CLI memakai teks tetap untuk matriks singular, bukan pesan engine yang berbeda tiap metode, jadi GUI mengikutinya. Matriks non-persegi memakai pesan engine ("Invers hanya dapat dicari pada matriks persegi."). Adjoin diukur sekitar n pangkat 5: 11 x 11 sekitar 0,27 detik, 25 x 25 sekitar 4,3 detik, jadi peringatan tetap perlu untuk matriks besar dari file.

### 3.10 `PolynomialPanel.java`
**Isi:** `PointsInputPanel`, kolom nilai x, tombol Hitung dan Prediksi. Fitur "Interpolasi Polinomial", metode "SPL Vandermonde". Field `snapshotPoints` dan `currentReport`.

**Method:**
- `static compute(String pointsText, boolean fromFile)`: minimal 2 titik ("Interpolasi polinomial membutuhkan minimal 2 titik data."), memanggil `PolynomialInterpolation.interpolate`, menyusun "Persamaan Polinomial:\nP(x) = ...".
- `static predict(Report current, String pointsText, String xText)`: menambah satu blok `"\n\nPrediksi:\n" + "Estimasi P(x) = y"` ke hasil. Model dibangun ulang dari teks snapshot.
- Private: `formatPolynomial` (suku nol dilewati, tanda antar-suku, `x`, `x^n`), `calculate`, `predictNext`.

**Alasan:**
- Tiap prediksi menambah judul "Prediksi:" sendiri (bertumpuk), meniru loop prediksi CLI.
- Snapshot: `calculate` menyimpan teks titik. `predictNext` selalu memakai snapshot itu, bukan isi kotak saat ini. Hitung yang gagal membuang snapshot.
- Prediksi sebelum Hitung menampilkan dialog "Hitung dulu: tekan tombol Hitung sebelum memprediksi."
- x kosong menghasilkan "Nilai x tidak boleh kosong." (pesan milik GUI; CLI menganggap baris kosong sebagai selesai). x bukan angka memakai pesan parser CLI: `Bilangan tidak valid: "abc"`.
- Ekstrapolasi (x di luar rentang titik) diizinkan.
- Enter di kolom x juga memprediksi.

### 3.11 `SplinePanel.java`
**Isi:** sama polanya dengan `PolynomialPanel`. Fitur "Natural Cubic Spline", metode "Sistem Tridiagonal". Field `snapshotPoints` dan `baseReport`.

**Method:**
- `static compute(String pointsText, boolean fromFile)`: minimal 2 titik ("Interpolasi spline kubik membutuhkan minimal 2 titik data."), `CubicSpline.interpolate`.
- `static predict(Report base, String pointsText, String xText)`: menambah SATU blok `"\nPrediksi:\n" + "Estimasi f(x) = y"` ke `base`.
- Private: `formatSpline` (turunan kedua per titik, lalu fungsi per interval `a + b(x - xi) + c(x - xi)^2 + d(x - xi)^3`), `calculate`, `predictNext`.

**Alasan:**
- CLI hanya bertanya prediksi sekali untuk spline, dan pemisahnya satu baris baru. Karena itu prediksi selalu ditambahkan ke `baseReport` (hasil tepat setelah Hitung), jadi klik berikutnya menggantikan, bukan menumpuk.
- x di luar rentang titik ditolak engine: "Nilai 5.0 berada di luar domain interpolasi [0.0, 3.0]." Hasil di layar tetap.
- Contoh jawaban: titik (0,0),(1,1),(2,4),(3,9) menghasilkan turunan kedua 0, 2.4, 2.4, 0 dan S(1.5) = 2.200.

### 3.12 `RegressionPanel.java`
**Isi:** `PointsInputPanel`, kolom knot (dipisah spasi, kosong berarti tanpa knot), kolom nilai x, Hitung dan Prediksi. Fitur "Regresi Spline Berganda", metode "Persamaan Normal". Field `snapshotPoints`, `snapshotKnots`, `baseReport`.

**Method:**
- `static compute(String pointsText, boolean fromFile, String knotsText)`: memanggil `fit`, lalu menyusun bagian Input (titik + "Knot (K=k): " diikuti tiap knot dan satu spasi, atau "(tidak ada)") dan teks hasil.
- `static predict(Report base, String pointsText, String knotsText, String xText)`: satu blok `"\n\nPrediksi:\n" + "Estimasi f(x) = y"`.
- Private static: `fit` (validasi + membuat `SplineRegression`), `formatModel` ("Koefisien:" b0.., "Knot:" k1.. atau "(tidak ada)", "Persamaan Regresi:").
- Private: `calculate`, `predictNext`.

**Validasi di `fit`, urutan dan pesan persis `RegressionMenu`:**
1. N < 5: "Syarat regresi adalah N > 4 + K. Jumlah titik data (N) minimal adalah 5."
2. Knot bukan angka: "Error parsing knot: " + pesan parser.
3. K + 4 >= N: "Sistem harus overdetermined (N > 4 + K). Untuk N=<n>, maksimal K yang diizinkan adalah <n-5>."
4. Sisanya pesan engine apa adanya, misalnya "Matriks tidak memiliki balikan (pivot bernilai nol)." untuk knot ganda, knot jauh di luar data, atau semua x sama.

**Alasan:**
- K tidak diketik terpisah, melainkan jumlah angka di kolom knot.
- CLI tidak memeriksa urutan, duplikat, atau rentang knot, jadi GUI juga tidak (knot terbalik "4 2" diterima dan dicetak sesuai urutan).
- Panjang `y` selalu sama dengan `x` karena titik dibaca berpasangan.
- Snapshot menyimpan teks titik DAN teks knot. Mengubah knot setelah Hitung tidak memengaruhi prediksi sampai Hitung ditekan lagi.
- Regresi boleh ekstrapolasi (tidak ada batas domain).

---

## 4. Pengujian yang Pernah Dilakukan

Dilakukan dengan program uji sementara di luar repositori. Program itu menjalankan menu CLI yang asli dengan masukan terskrip (`System.setIn`), menangkap keluaran (`System.setOut`), lalu membandingkannya dengan hasil `compute`. Untuk berkas simpan, CLI menulis ke berkas sementara dan isinya dibandingkan dengan `ResultPanel.buildFileContent`.

| Fitur | Skenario | Yang dibandingkan |
|---|---|---|
| Determinan | 7 matriks x 2 metode | teks hasil dan pesan galat |
| Invers | 9 matriks x 2 metode | teks hasil dan pesan galat |
| SPL | 11 sistem x 4 metode | 32 teks hasil dan 12 pesan galat |
| Polinomial | 11 sesi | 8 berkas simpan dan 3 pesan galat |
| Spline | 16 sesi | 11 berkas simpan dan 5 pesan galat |
| Regresi | 22 sesi | 11 berkas simpan dan 11 pesan galat |

Jawaban yang dihitung tangan:
- P(3) = 9.000 untuk titik (0,0),(1,1),(2,4).
- Spline natural titik (0,0),(1,1),(2,4),(3,9): turunan kedua 0, 2.4, 2.4, 0 dan S(1.5) = 2.200.
- Data kubik persis y = 2 - x + 0.5x^2 + 0.25x^3 menghasilkan koefisien tepat, dan kurva melewati semua titik.
- Data y = x^3 + 2(x-3)^3_+ dengan knot 3 menghasilkan b3 = 1, b4 = 2, dan f(5.5) = 197.625.

Alur panel yang diuji lewat tombol: tombol nonaktif saat worker jalan, peringatan muncul sesuai metode, dialog galat, snapshot, prediksi mengganti atau menumpuk, dan berkas simpan dari panel identik dengan berkas CLI.

---

## 5. Batas dan Keterbatasan (jujur saat demo)

- **Matriks sangat besar dari file.** `StepLog` menyimpan salinan teks matriks di tiap kolom. Pada pengukuran, SPL 1001 x 1002 dengan log lengkap kehabisan memori setelah sekitar 271 detik (heap bawaan 3,9 GB). `OutOfMemoryError` ditangkap dan muncul dialog, jadi program tidak crash. Log ringkas (`CompactStepLog`) belum dibuat.
- **Metode lambat tidak bisa dibatalkan:** kofaktor (11 x 11 sekitar 35 detik), adjoin, dan Cramer. Hanya ada label peringatan.
- **Dialog `JFileChooser` berbahasa Inggris** karena `Locale.US` wajib. Hanya dialog buatan sendiri (Timpa/Batal, galat) yang berbahasa Indonesia.
- **CLI menolak SPL augmented 1001 x 1002 dari file** (`FileHandler` membatasi 1001 angka per baris). GUI mengikuti spesifikasi (1002). CLI tidak diubah.
- **Argumen `gui` pada JAR belum dipasang di `App.main`.** Saat ini GUI dijalankan dengan `java -cp target/classes algeo.gui.GuiApp`.
- **`pom.xml` masih memuat dependensi JavaFX** bawaan templat. GUI tidak memakainya (hanya Swing).
- **Tiga panel matriks hampir kembar** di bagian tampilan. Refaktor ke kelas dasar bersama belum dikerjakan.
- **Menu Image Inpainting (CLI nomor 7) tidak ada di GUI.** GUI hanya mencakup enam fitur aljabar linier.

---

## 6. Skrip Demo (urutan klik)

1. Jalankan GUI. Tunjukkan enam tab.
2. **Determinan:** ketik `1 2` dan `3 4.5`, metode OBE, Hitung. Hasil `Determinan = -1.500` beserta langkah. Pilih Kofaktor: label peringatan oranye muncul. Ketik matriks 2 x 3, Hitung: dialog "Matriks tidak memiliki determinan".
3. **Invers:** `1 2` dan `3 4`: hasil `-2.000 1.000` dan `1.500 -0.500`. Lalu `1 2` dan `2 4`: dialog "Matriks tidak memiliki balikan".
4. **SPL:** `1 1 3` dan `1 -1 1` di keempat metode: `x1 = 2.000`, `x2 = 1.000`. Lalu `1 1 2` dan `2 2 4` dengan Gauss-Jordan: `x1 = 2.000 - t`, `x2 = t`. Lalu `1 1 2` dan `1 1 3`: `Solusi tidak ada.`
5. **Polinomial:** titik `0 0`, `1 1`, `2 4`. Hitung: `P(x) = 1.000x^2`. Nilai x `3`: `9.000`. Tekan Prediksi lagi dengan x `2`: bertumpuk.
6. **Spline:** titik `0 0`, `1 1`, `2 4`, `3 9`. x `1.5`: `2.200`. x `2`: prediksi diganti. x `5`: dialog di luar domain.
7. **Regresi:** titik `0 0`, `1 1`, `2 8`, `3 27`, `4 66`, `5 141`, `6 270`, `7 471`, `8 762`, knot `3`. Persamaan `y = 1.000x^3 + 2.000(x - 3.000)^3_+`. x `5.5`: `197.625`. Ubah knot jadi `5` tanpa menekan Hitung, Prediksi lagi: tetap `197.625` (snapshot). Tekan Hitung, baru berubah.
8. **File:** Buka file memakai berkas di folder `test/` yang sesuai fitur. Simpan hasil tanpa mengetik `.txt` (otomatis), lalu simpan lagi dengan nama sama untuk menunjukkan konfirmasi Timpa.
9. **Matriks besar:** muat file lebih dari 11 x 11. Kotak berubah menjadi ringkasan dan terkunci sampai Bersihkan ditekan.

---

## 7. Pertanyaan yang Mungkin Muncul

- **Kenapa `compute` dibuat static dan tanpa tipe Swing?** Supaya bisa dijalankan di thread latar belakang dan diuji tanpa jendela.
- **Kenapa SwingWorker hanya di panel matriks?** Matriks bisa sampai 1001 x 1001 dan beberapa metode lambat. Titik maksimal 10, jadi hanya 1 sampai 2 milidetik.
- **Kenapa kode format ditulis ulang, bukan dipanggil dari CLI?** `gui` dilarang mengimpor `cli`, agar dua klien tidak saling bergantung. Kesamaan dijaga lewat pengujian terhadap menu CLI asli.
- **Kenapa ada snapshot?** Supaya prediksi dan persamaan di layar selalu berasal dari model yang sama, dan berkas simpan tidak memuat dua model berbeda.
- **Kenapa spline hanya satu prediksi sementara polinomial bertumpuk?** Itu perilaku CLI. GUI mengikutinya agar berkas simpan bisa dihasilkan CLI juga.
- **Bagaimana GUI menangani masukan salah?** Semua exception menjadi dialog (via `ResultPanel.describeError`). Pesan menyebut nomor baris. Program tidak crash.
- **Kenapa tidak memakai `FileHandler` untuk membaca file?** Ia mencetak galat ke konsol dan mengembalikan `null`, jadi pesan tidak sampai ke GUI.
- **Kenapa ada JavaFX di `pom.xml`?** Sisa templat. GUI memakai Swing dari JDK.
- **Bagaimana membuktikan keluaran GUI sama dengan CLI?** Dengan menjalankan menu CLI asli untuk masukan yang sama dan membandingkan teks serta isi berkas simpan (Bagian 4).
- **Apa yang terjadi bila matriks sangat besar?** Memori bisa habis karena log langkah. Galatnya ditangkap dan ditampilkan sebagai dialog. Perbaikan dengan log ringkas belum dibuat.