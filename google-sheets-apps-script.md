# Integrasi Google Sheets dan Google Apps Script

Panduan ini menambahkan opsi cadangan menggunakan Google Sheets dan Apps Script setelah foto diunggah ke Google Drive.

## 1. Buat Google Sheet Baru

1. Buka `https://docs.google.com/spreadsheets/`
2. Buat spreadsheet baru.
3. Ganti nama sheet menjadi `museum_data`.
4. Isi baris pertama dengan header berikut:
   - A1: `nama_benda`
   - B1: `kategori`
   - C1: `keterangan`
   - D1: `foto_url`
   - E1: `drive_file_id`
   - F1: `created_at`

## 2. Buat Google Apps Script

1. Di Google Sheet, pilih menu `Extensions` -> `Apps Script`.
2. Hapus konten default dan ganti dengan kode berikut:

```javascript
function doPost(e) {
  try {
    var body = JSON.parse(e.postData.contents);
    var namaBenda = body.nama_benda;
    var kategori = body.kategori;
    var keterangan = body.keterangan;
    var fileName = body.file_name;
    var fileType = body.file_type;
    var fileBase64 = body.file_base64;

    if (!namaBenda || !kategori || !keterangan || !fileName || !fileType || !fileBase64) {
      return ContentService
        .createTextOutput(JSON.stringify({ success: false, message: 'Data tidak lengkap.' }))
        .setMimeType(ContentService.MimeType.JSON);
    }

    var folderId = 'YOUR_DRIVE_FOLDER_ID';
    var folder = DriveApp.getFolderById(folderId);
    var blob = Utilities.newBlob(Utilities.base64Decode(fileBase64), fileType, fileName);
    var driveFile = folder.createFile(blob);
    driveFile.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);

    var fotoUrl = 'https://drive.google.com/uc?export=view&id=' + driveFile.getId();
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('museum_data');
    sheet.appendRow([namaBenda, kategori, keterangan, fotoUrl, driveFile.getId(), new Date()]);

    return ContentService
      .createTextOutput(JSON.stringify({ success: true, message: 'Data berhasil ditambahkan.', foto_url: fotoUrl, drive_file_id: driveFile.getId() }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, message: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Ganti `YOUR_DRIVE_FOLDER_ID` dengan ID folder Google Drive Anda.
4. Klik `Save`.
5. Ganti nama proyek Apps Script, misalnya `MuseumSheetApi`.

## 3. Deploy sebagai Web App

1. Klik `Deploy` -> `New deployment`.
2. Pilih `Web app`.
3. Pada `Description`, tulis `Museum data logger`.
4. `Execute as`: pilih `Me`.
5. `Who has access`: pilih `Anyone`.
6. Klik `Deploy`.
7. Copy URL yang muncul; ini adalah endpoint webhook Apps Script.

## 4. Hubungkan Apps Script dengan `admin.html`

1. Buka `admin.html`.
2. Ganti baris `APP_SCRIPT_URL` dengan URL Web App Anda:

```javascript
const APP_SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```

3. Simpan perubahan.

## 5. Pengujian Manual Web App

Anda dapat menguji dengan `curl` atau Postman jika ingin memastikan Web App berfungsi:

```bash
curl -X POST '<WEB_APP_URL>' \
  -H 'Content-Type: application/json' \
  -d '{
    "nama_benda": "Alat Musik Tradisional",
    "kategori": "Alat Musik",
    "keterangan": "Terbuat dari bambu.",
    "file_name": "alat_musik.jpg",
    "file_type": "image/jpeg",
    "file_base64": "<BASE64_DATA>"
  }'
```

Response yang sukses akan berbentuk JSON:

```json
{
  "success": true,
  "message": "Data berhasil ditambahkan.",
  "foto_url": "https://drive.google.com/uc?export=view&id=FILE_ID",
  "drive_file_id": "FILE_ID"
}
```

## 6. Langkah penuh tanpa PHP

Alur lengkap saat tombol `Submit` ditekan di `admin.html`:
1. Form validasi di browser.
2. Foto diubah menjadi Base64 oleh JavaScript.
3. Data dikirim ke Apps Script Web App.
4. Apps Script membuat file di Google Drive.
5. Apps Script menyimpan metadata ke Google Sheets.
6. Browser menerima response dan menampilkan pesan sukses atau error.

Jika ada error, status akan muncul di halaman `admin.html`.

## 7. Menjalankan secara manual

1. Pastikan `service-account.json` sudah ada.
2. Pastikan `composer install` sudah dijalankan.
3. Jalankan `php -S localhost:8080` di folder `d:\museum`.
4. Buka `http://localhost:8080/login.php`.
5. Login sebagai admin.
6. Buka `admin.php`, isi data, unggah foto.
7. Pastikan data muncul di `Google Sheet` dan `MySQL`.

## 8. Catatan penting

- `Apps Script` sudah cukup untuk menyimpan metadata ke Google Sheets. Anda tidak perlu API Google Sheets yang kompleks.
- Pastikan Web App Google Apps Script sudah di-deploy ulang jika Anda mengubah kode.
- Jangan bagikan `service-account.json` dan URL endpoint Apps Script kepada publik.

## 9. Jika ingin saya bantu selanjutnya

Saya bisa bantu membuatkan:
- contoh kode PHP lengkap untuk memanggil Apps Script setelah upload Drive,
- contoh form HTML lengkap di `admin.php`,
- contoh Google Apps Script dengan validasi tambahan,
- contoh `curl` test dan debugging pesan error.
