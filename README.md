# Pengajuan Pelatihan PPA

Aplikasi web (PWA) untuk pengajuan dan approval pelatihan dengan alur tiga lapis: **Kandidat → User (atasan langsung) → Departemen Head (Human Capital Development)**. Login pakai username & password yang dibuat dan dibagikan sendiri oleh HCD — tidak perlu Google Cloud, tidak ada biaya.

## Fitur

- Login username & password, dibuat dan dibagikan sendiri (tidak bergantung email/akun Google)
- **Pembatasan akses ditegakkan di server (Apps Script), bukan cuma di tampilan:**
  - Kandidat hanya melihat pengajuannya sendiri
  - User hanya melihat & bisa approve pengajuan dari karyawan yang akunnya terhubung ke dirinya sebagai atasan
  - Dept. Head (HCD) melihat semua pengajuan yang menunggu approval final, lintas departemen
- Atasan langsung setiap kandidat sudah ditetapkan di akunnya — tidak perlu diketik ulang setiap mengajukan
- Tanda tangan digital di setiap tahap
- Notifikasi email otomatis di setiap perubahan status
- Data tersimpan di Google Sheets
- Bisa diinstall sebagai aplikasi di HP (PWA)

## Struktur file

```
index.html               Aplikasi utama (login + tiga tab peran)
manifest.json              Konfigurasi PWA
service-worker.js          Cache app shell untuk instalasi PWA
icon-192.png / icon-512.png  Ikon aplikasi
generate-password.html     Alat lokal untuk membuat hash password akun baru
```

## Setup — Google Sheets

Buat 2 tab di satu Google Sheet:

**Tab "Pengajuan"** — baris header:
```
id, nama, departemen, pelatihan, penyelenggara, mulai, selesai, biaya, alasan, status, signatureKandidat, submittedAt, catatanUser, approverUser, signatureUser, decidedAtUser, catatanDept, approverDept, signatureDept, decidedAtDept, usernameKandidat, usernameAtasan
```

**Tab "Users"** — daftar akun:
```
Username | PasswordHash | Nama | Role | Email | AtasanUsername
```
- `Role` diisi salah satu: `kandidat`, `user`, atau `hcd`
- `AtasanUsername` hanya diisi untuk baris berperan `kandidat` — isi dengan Username dari baris berperan `user` yang menjadi atasannya
- `Email` dipakai untuk mengirim notifikasi, boleh email pribadi/kerja apa saja

Contoh isi:
| Username | PasswordHash | Nama | Role | Email | AtasanUsername |
|---|---|---|---|---|---|
| budi.s | (hasil generate) | Budi Santoso | user | budi@mail.com | |
| andi.w | (hasil generate) | Andi Wijaya | kandidat | andi@mail.com | budi.s |
| sari.hcd | (hasil generate) | Sari Wijaya | hcd | sari@mail.com | |

## Cara membuat akun baru

1. Buka `generate-password.html` di browser mana saja (tidak perlu online, cukup dobel klik filenya).
2. Isi username dan password yang ingin diberikan ke orang tersebut, klik **Generate hash**.
3. Salin hasil hash yang muncul, tempel ke kolom **PasswordHash** di tab Users pada Google Sheet (bukan passwordnya — hash-nya).
4. Beri tahu orang tersebut **username dan password aslinya** (bukan hash) lewat cara apa pun yang aman — chat pribadi, kertas, dsb.
5. Isi juga kolom Nama, Role, Email, dan AtasanUsername (jika Role = kandidat) di baris yang sama.

## Setup — Apps Script

1. Extensions → Apps Script, hapus kode lama, tempel isi `apps_script_username_password.txt`.
2. Deploy → New deployment → Web app. Execute as **Me**, Who has access **Anyone**.
3. Copy URL Web App, tempel ke `SHEETS_WEB_APP_URL` di `index.html` (kalau URL-nya sama dengan sebelumnya, tidak perlu diubah).

## Setup — Hosting (GitHub Pages)

1. Upload semua file (`index.html`, `manifest.json`, `service-worker.js`, `icon-192.png`, `icon-512.png`) ke root repository GitHub. (`generate-password.html` tidak perlu diupload — cukup dipakai lokal oleh HCD saat membuat akun.)
2. Settings → Pages → Branch `main`, folder `/ (root)` → Save.
3. Buka URL yang dihasilkan, login dengan username & password yang sudah dibuat.

## Cara pakai

| Peran | Yang dilihat & bisa dilakukan |
|---|---|
| Kandidat | Isi & kirim form (tanpa perlu isi nama/atasan — otomatis dari akun). Hanya melihat status pengajuan miliknya sendiri. |
| User | Hanya melihat pengajuan dari akun kandidat yang `AtasanUsername`-nya diisi sesuai username-nya. Tab selalu tampil; kalau tidak ada bawahan yang mengajukan, daftarnya kosong. |
| Dept. Head (HCD) | Tab ini hanya muncul untuk akun dengan Role = `hcd`. Melihat semua pengajuan lintas departemen yang menunggu approval final. |

## Keterbatasan saat ini

- Password disimpan dalam bentuk hash (SHA-256) di Google Sheets, cukup aman dari yang sekadar melihat sheet-nya, tapi ini bukan tingkat keamanan bank — cukup untuk alat internal tim kecil.
- Sesi login bertahan maksimal 6 jam (batas teknis Apps Script), setelah itu perlu login ulang.
- Tidak ada fitur "lupa password" otomatis — kalau lupa, HCD perlu generate password baru dan update hash-nya di sheet.
- Tanda tangan berupa gambar coretan, belum tentu punya kekuatan hukum sebagai tanda tangan elektronik tersertifikasi.
- Kalau dua approval terjadi nyaris bersamaan pada baris yang sama, ada kemungkinan kecil data saling menimpa.

## Pengembangan lanjutan

- **AppSheet** — alternatif no-code dengan kontrol akses per baris yang lebih matang.
- **Backend + database + auth penuh** — untuk skala besar dan kebutuhan audit yang lebih ketat.
