# Pengajuan Pelatihan PPA

Aplikasi web (PWA) untuk pengajuan dan approval pelatihan dengan alur tiga lapis: **Kandidat → User (atasan langsung) → Departemen Head (Human Capital Development)**. Login pakai username & password, dilengkapi pelacakan jam belajar tahunan dan pengumpulan bukti pembelajaran.

## Fitur

- Login username & password, dibuat dan dibagikan sendiri oleh HCD
- Pembatasan akses ditegakkan di server: Kandidat hanya lihat pengajuan sendiri, User hanya lihat tim yang ia awasi, HCD lihat semua yang pending final
- Tanda tangan digital di setiap tahap
- **Disclaimer wajib dicentang** sebelum tanda tangan: kandidat berkomitmen mengumpulkan materi pembelajaran dan menerapkannya dalam proyek nyata yang ia tentukan sendiri
- **Kolom lama pembelajaran (jam)** di setiap pengajuan
- **Progress bar jam belajar tahunan** — target 45 jam/tahun, dihitung dari pengajuan berstatus disetujui pada tahun berjalan
- **Upload materi pembelajaran & bukti pengerjaan proyek**, muncul otomatis setelah pengajuan disetujui penuh
- Notifikasi email otomatis di setiap perubahan status
- **Semua file (tanda tangan, materi, bukti proyek) disimpan sebagai file sungguhan di Google Drive** — Sheet hanya menyimpan link, bukan teks base64 panjang
- Field disesuaikan dengan formulir fisik "Permohonan Pelatihan Karyawan" PPA: Kategori Pelatihan (Internal/Eksternal), Tempat pelaksanaan, dan Jabatan peserta
- **Tombol "Unduh PDF"** di setiap pengajuan — menghasilkan dokumen PDF dengan tata letak menyerupai formulir fisik PPA, lengkap dengan tabel info pelatihan, tabel peserta, dan tabel approval 3 kolom berisi gambar tanda tangan asli
- Bisa diinstall sebagai aplikasi di HP (PWA)

## Struktur file

```
index.html               Aplikasi utama
manifest.json              Konfigurasi PWA
service-worker.js          Cache app shell untuk instalasi PWA
icon-192.png / icon-512.png  Ikon aplikasi
generate-password.html     Alat lokal untuk membuat hash password akun baru
```

## Setup — Google Sheets

**Tab "Pengajuan"** — baris header (8 kolom baru ditambahkan di akhir, kolom lama tidak berubah):
```
id, nama, departemen, pelatihan, penyelenggara, mulai, selesai, biaya, alasan, status, signatureKandidat, submittedAt, catatanUser, approverUser, signatureUser, decidedAtUser, catatanDept, approverDept, signatureDept, decidedAtDept, usernameKandidat, usernameAtasan, lamaJam, disclaimerAccepted, materiFileUrl, buktiProyekFileUrl, materiUploadedAt, kategoriPelatihan, tempat, jabatan
```

**Tab "Users"** — ada 1 kolom baru (`Jabatan`):
```
Username | PasswordHash | Nama | Jabatan | Role | Email | AtasanUsername
```
`Jabatan` diisi posisi/jabatan karyawan (misal "Staff Cost Control") — otomatis ikut terekam di setiap pengajuan tanpa perlu diketik ulang, sama seperti Nama.

## Setup — Apps Script

1. Extensions → Apps Script, hapus kode lama, tempel isi `apps_script_v5.txt`.
2. Klik **Save**.
3. **Deploy → Manage deployments → Edit (pensil) → Version: New version → Deploy.**
4. Google akan minta **otorisasi ulang** karena kode ini sekarang juga membuat & menghapus dokumen Google Docs sementara (untuk membuat PDF) — ikuti seperti biasa: pilih akun → Advanced → lanjutkan → Allow.

Sebuah folder baru bernama **"Lampiran Pengajuan Pelatihan PPA"** akan otomatis dibuat di Google Drive akun yang men-deploy script ini, tempat semua tanda tangan, materi, dan bukti proyek tersimpan.

## Setup — Hosting (GitHub Pages)

Sama seperti sebelumnya — upload `index.html` yang baru ke root repository (timpa yang lama). File lain tidak berubah.

## Cara pakai

| Peran | Yang dilihat & bisa dilakukan |
|---|---|
| Kandidat | Isi & kirim form pengajuan lengkap dengan lama jam dan centang disclaimer. Lihat progress jam belajar tahunan. Setelah pengajuan disetujui, muncul form upload materi & bukti proyek. |
| User | Lihat & approve pengajuan dari bawahannya. |
| Dept. Head (HCD) | Lihat & approve semua pengajuan yang sudah lolos User, lintas departemen. |

## Keterbatasan saat ini

- Ukuran file materi/bukti proyek dibatasi maksimal 8MB per file di sisi aplikasi (bisa disesuaikan) untuk menghindari timeout saat upload.
- Progress jam belajar dihitung dari tanggal **mulai** pelatihan, bukan tanggal disetujui — kalau pelatihan diajukan di akhir tahun untuk mulai tahun depan, akan terhitung di tahun berikutnya.
- File di Drive dibagikan dengan akses "Anyone with the link — view only" supaya bisa ditampilkan di aplikasi. Ini berarti siapa pun yang punya link filenya bisa membuka, meski link itu tidak dipublikasikan di mana pun.
- Sesi login bertahan maksimal 6 jam.
- Tanda tangan berupa gambar coretan, belum tentu punya kekuatan hukum sebagai tanda tangan elektronik tersertifikasi.
- PDF yang dihasilkan tidak menyertakan logo perusahaan (hanya teks nama perusahaan) — menambahkan logo grafis memerlukan penyesuaian kode lebih lanjut.

## Pengembangan lanjutan

- **AppSheet** — alternatif no-code dengan kontrol akses per baris yang lebih matang.
- **Backend + database + auth penuh** — untuk skala besar dan kebutuhan audit yang lebih ketat.
- **Reminder otomatis** untuk kandidat yang belum mengumpulkan materi/bukti proyek dalam X hari setelah disetujui.
