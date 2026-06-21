# Admin Panel dengan Google Drive Upload

## 1. Persiapan Database
1. Buat database MySQL baru, misalnya `museum`.
2. Buat user database dan berikan hak akses pada database tersebut.
3. Import `database.sql`:
   - `mysql -u root -p < database.sql`
4. Tambahkan admin user manual jika perlu:
   - Gunakan `password_hash('passwordAnda', PASSWORD_DEFAULT)` di PHP untuk menghasilkan password hash.

## 2. Konfigurasi `config.php`
1. Ganti `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS` sesuai database Anda.
2. Ganti `GOOGLE_DRIVE_FOLDER_ID` dengan folder ID di Google Drive.
3. Simpan file `service-account.json` ke folder proyek.

## 3. Menyiapkan Google Service Account dan Drive
1. Buka Google Cloud Console: https://console.cloud.google.com/
2. Buat project baru.
3. Aktifkan API Google Drive (`Drive API`).
4. Buat Service Account:
   - Identity -> Service Accounts -> Buat Service Account.
   - Beri nama dan tambahkan deskripsi.
5. Buat kunci JSON:
   - Pilih service account -> Keys -> Add key -> Create new key -> JSON.
   - Simpan file `service-account.json` ke folder proyek.
6. Atur folder Google Drive:
   - Buka Google Drive, buat folder baru.
   - Klik kanan folder -> Share -> Change to "Anyone with the link" -> Viewer.
   - Salin folder ID dari URL: `https://drive.google.com/drive/folders/<FOLDER_ID>`.

## 4. Konfigurasi Akses Drive untuk Service Account
1. Jika file diunggah ke folder pribadi, tambahkan service account sebagai viewer/editor folder:
   - Klik kanan folder -> Share -> masukkan alamat email service account.
   - Atur sebagai Viewer atau Editor.
2. Pastikan permission file menjadi "Anyone with the link can view".

## 5. Menjalankan Proyek
1. Pastikan PHP tersedia di sistem.
2. Jalankan server lokal di folder `d:\museum`:
   - `php -S localhost:8080`
3. Buka browser:
   - `http://localhost:8080/login.php`

## 6. Halaman yang tersedia
- `login.php` - halaman login admin.
- `admin.php` - form input data (butuh login).
- `category.php?kategori=...` - tampilkan data berdasarkan kategori.

## 7. Debugging dan Error Handling
- Jika upload Drive gagal, pesan kesalahan akan tampil di halaman admin.
- Pastikan `service-account.json` ada dan `GOOGLE_DRIVE_FOLDER_ID` benar.
- Pastikan `composer install` dijalankan untuk library Google Client.

## 8. Perhatian Keamanan
- Gunakan HTTPS pada server produksi.
- Jangan simpan password plaintext di file.
- Ganti admin password hash dengan nilai hash password Anda sendiri.
