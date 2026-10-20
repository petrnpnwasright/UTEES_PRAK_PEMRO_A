# Repositori Soal DOMjudge

Repositori ini menyimpan paket-paket soal yang akan diimpor ke instance DOMjudge. Setiap soal adalah satu folder di root repositori, dan tiap folder itu sendiri adalah isi dari satu problem archive (zip) yang diimpor lewat **Problems → Import problem archive**.

## Ketentuan upload soal
### Nama Soal
**Kesulitan - NamaSoal**
Upload di directory masing masing

## Struktur repositori

```
.
├── README.md
├── messi/
│   ├── problem.pdf
│   ├── problem.yaml
│   ├── domjudge-problem.ini
│   ├── data/
│   │   ├── sample/
│   │   │   ├── 1.in
│   │   │   └── 1.ans
│   │   └── secret/
│   │       ├── 01.in
│   │       ├── 01.ans
│   │       ├── 02.in
│   │       ├── 02.ans
│   │       └── ...
│   └── submissions/
│       ├── accepted/
│       │   └── sol.c
│       ├── wrong_answer/
│       └── time_limit_exceeded/
└── <nama-soal-lain>/
    └── ...
```

Satu folder = satu soal = satu zip. Jangan menaruh beberapa soal dalam satu zip, dan jangan membungkus isi folder dalam satu folder tambahan saat di-zip (lihat bagian **Cara mengemas untuk upload** di bawah).

## Penjelasan tiap file/folder

### `problem.pdf`
Naskah soal (deskripsi, format masukan/keluaran, contoh, batasan) dalam bentuk PDF. Ini yang ditampilkan ke peserta di DOMjudge. Bisa diganti `problem.html` atau `problem.txt` kalau tidak memakai PDF, tapi harus salah satu dari ketiganya dan ada tepat satu di root folder.

### `problem.yaml`
Metadata soal yang dibaca DOMjudge saat impor:

```yaml
name: messi
validation: default
validator_flags: case_sensitive float_tolerance 1e-6
limits:
  memory: 256
```

- `name` — nama soal.
- `validation: default` — pakai pembanding token-per-token bawaan DOMjudge (bukan custom checker).
- `validator_flags` — opsi untuk pembanding default. Yang umum dipakai: `case_sensitive`, `space_change_sensitive`, `float_tolerance <angka>`, `float_absolute_tolerance <angka>`. Hilangkan kalau soal hanya membandingkan teks persis tanpa angka desimal.
- `limits.memory` — batas memori dalam MB.
- File ini harus UTF-8 murni, tanpa BOM. Edit dan simpan sebagai UTF-8 biasa (bukan UTF-16) kalau pakai editor di Windows.

### `domjudge-problem.ini`
Metadata tambahan gaya lama, dibaca juga saat impor:

```ini
name='Messi'
timelimit='1'
externalid='Messi'
short-name='Messi'
color='#8E24AA'
```

- `timelimit` — batas waktu dalam detik.
- `externalid` — ID unik soal ini di sistem (dipakai untuk merujuk soal yang sama dari contest yang berbeda).
- `short-name` — label singkat soal, biasanya huruf (A, B, C, ...) sesuai urutan di contest.
- `color` — warna penanda soal di scoreboard.

### `data/sample/`
Contoh masukan-keluaran yang **ditampilkan ke peserta** di halaman soal. Pasangan file `N.in` dan `N.ans` dengan nama sama. Biasanya hanya 1–2 pasang, persis seperti contoh di PDF.

### `data/secret/`
Test case yang dipakai untuk menilai submission peserta, **tidak ditampilkan** ke mereka. Juga pasangan `nama.in` / `nama.ans`. Konvensi penamaan di repo ini: dua digit berurutan (`01`, `02`, ...), boleh ditambah keterangan singkat (`01_kasus_kecil.in`) selama nama `.in` dan `.ans` tetap sama persis.

Isi test secret sebaiknya mencakup:
- Kasus dari contoh di PDF (boleh diulang di sini).
- Kasus kecil/edge case (N minimum, nilai batas, dsb).
- Kasus acak ukuran sedang.
- Kasus ukuran maksimum sesuai batasan soal, untuk menguji batas waktu.

### `submissions/`
Kode solusi yang dipakai DOMjudge untuk **menguji soal itu sendiri** sebelum dipakai di contest sungguhan (lewat fitur "Test all submissions" / jury rejudge). Subfoldernya menentukan verdict yang diharapkan:

- `accepted/` — solusi benar; harus Accepted di semua test.
- `wrong_answer/` — solusi dengan kesalahan logika yang disengaja; harus Wrong Answer di minimal satu test.
- `time_limit_exceeded/` — solusi yang benar tapi terlalu lambat (misalnya kompleksitas O(N²) padahal perlu O(N log N)); harus TLE.
- Folder lain yang didukung DOMjudge kalau relevan: `run_time_error/`, `memory_limit/`, dll.

Minimal harus ada satu solusi di `accepted/`. Makin banyak variasi solusi (termasuk WA dan TLE yang disengaja), makin yakin test data sudah cukup ketat.

## Checklist sebelum commit sebuah soal

- [ ] `problem.yaml` dan `domjudge-problem.ini` valid dan UTF-8 murni (cek dengan `file nama_file` — harus `ASCII text` atau `UTF-8 Unicode text`, bukan `UTF-16` atau `ISO-8859`).
- [ ] Setiap `.in` di `data/sample` dan `data/secret` punya `.ans` pasangannya, nama persis sama.
- [ ] Solusi di `submissions/accepted/` sudah dijalankan manual terhadap semua test dan outputnya cocok dengan `.ans`.
- [ ] Tidak ada file biner hasil kompilasi (`.o`, `.exe`, atau binary tanpa ekstensi) ikut ter-commit di `submissions/`.
- [ ] Tidak ada folder bungkus tambahan (lihat bagian di bawah).

## Cara mengemas untuk upload

DOMjudge mengharapkan `problem.pdf`, `problem.yaml`, `domjudge-problem.ini`, dan folder `data/` ada **langsung di root zip**, bukan di dalam satu folder pembungkus. Dari dalam folder soal:

```bash
cd messi
zip -r ../messi.zip .
```

Bukan:

```bash
cd ..
zip -r messi.zip messi/   # SALAH — semua file akan berada di dalam messi/ di dalam zip
```

Zip hasilnya diupload lewat **Problems → Import problem archive** di DOMjudge, bukan lewat form "Import contest" atau "Import problems" (keduanya untuk file `.yaml`/`.json` tunggal berisi daftar soal, bukan untuk zip paket soal).
