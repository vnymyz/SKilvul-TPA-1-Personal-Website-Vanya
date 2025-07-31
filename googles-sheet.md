# LANGKAH-LANGKAH: Hubungkan Form HTML ke Google Sheets

## 1. Siapkan Google Spreadsheet

1. Buka Google Sheets : https://docs.google.com/spreadsheets/u/0/

2. Buat spreadsheet baru

3. Ubah nama sheet (opsional) — misalnya: Sheet1 atau Contact

4. Buat header kolom di baris pertama:

```pgsql
Timestamp | Name | Email | Message
```

## 2. Buka Apps Script dari Spreadsheet

1. Klik menu Extensions > Apps Script

2. Hapus semua isi default (function myFunction() {}), lalu ganti dengan:

```js
const sheetName = "Sheet1"; // ganti jika nama sheet berbeda
const scriptProp = PropertiesService.getScriptProperties();

function initialSetup() {
  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty("key", activeSpreadsheet.getId());
}

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    const doc = SpreadsheetApp.openById(scriptProp.getProperty("key"));
    const sheet = doc.getSheetByName(sheetName);

    const name = e.parameter.name;
    const email = e.parameter.email;
    const message = e.parameter.message;

    sheet.appendRow([new Date(), name, email, message]);

    return ContentService.createTextOutput(
      JSON.stringify({ result: "success", data: e.parameter })
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(
      JSON.stringify({ result: "error", error: err })
    ).setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

## 3. Jalankan initialSetup()

1. Di dropdown fungsi, pilih initialSetup

2. Klik tombol ▶️ (Run)

3. Izinkan akses kalau diminta

## 4. Deploy sebagai Web App

1. Klik Deploy > Manage Deployments

2. Klik New Deployment

3. Pilih:

   - Select type: Web app

   - Description: Form ke Sheets

   - Execute as: Me

   - Who has access: Anyone

4. Klik Deploy

5. Salin URL Web App yang muncul (berformat https://script.google.com/macros/s/.../exec)

## 5. Siapkan Form HTML

Contoh Kode Form html (sesuaikan dengan form atau contact html mu):

```html
<form name="submit-to-google-sheet">
  <input type="text" name="name" placeholder="Your Name" required />
  <input type="email" name="email" placeholder="Your Email" required />
  <textarea
    name="message"
    rows="6"
    placeholder="Your Message"
    required
  ></textarea>
  <button type="submit">Submit</button>
</form>
<span id="msg"></span>
```

tambahkan javascript berikut :

```js
<script>
  const scriptURL = 'https://script.google.com/macros/s/AKfycb.../exec'; // ganti dengan Web App URL kamu
  const form = document.forms['submit-to-google-sheet'];
  const msg = document.getElementById("msg");

  form.addEventListener('submit', e => {
    e.preventDefault();
    fetch(scriptURL, { method: 'POST', body: new FormData(form)})
      .then(response => {
        msg.innerHTML = "Thank you, your message has been sent.";
        setTimeout(() => msg.innerHTML = "", 5000);
        form.reset();
      })
      .catch(error => {
        msg.innerHTML = "Error! Message not sent.";
        console.error('Error!', error.message);
      });
  });
</script>
```

## 6. Test Form-nya

1. Buka file HTML kamu di browser (pakai Live Server lebih baik)

1. Isi form lalu submit

1. Cek Google Sheet → data akan otomatis masuk
