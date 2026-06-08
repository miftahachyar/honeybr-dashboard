# Dashboard Akuntansi UMKM — Honeybr Cheesecake (SAK EMKM)

Aplikasi pembukuan double-entry sesuai SAK EMKM dalam **satu file HTML**, dengan
**database cloud Firebase Firestore** sehingga data tersinkron realtime antara
akuntan dan pemilik usaha. Tanpa konfigurasi Firebase pun aplikasi tetap jalan
memakai penyimpanan lokal browser (localStorage).

| File | Fungsi |
|------|--------|
| `index.html` | Aplikasi dashboard (inilah yang di-hosting & dibagikan) |
| `firestore.rules` | Aturan keamanan database |
| `README.md` | Panduan ini |

---

## A. Membuat Cloud Sendiri di Firebase (sekali saja, ±10 menit)

### 1. Buat project Firebase
1. Buka <https://console.firebase.google.com> → **Add project** → beri nama, mis. `honeybr-akuntansi`.
2. Google Analytics boleh dimatikan (tidak perlu). Klik **Create project**.

### 2. Aktifkan Firestore Database
1. Menu kiri → **Build → Firestore Database** → **Create database**.
2. Pilih lokasi server (mis. `asia-southeast2` / Jakarta) → **Next**.
3. Pilih **Start in production mode** → **Enable**.
4. Buka tab **Rules**, ganti seluruh isinya dengan isi file `firestore.rules`
   (di repo ini) → **Publish**.

### 3. Aktifkan Login Anonim
1. Menu kiri → **Build → Authentication** → **Get started**.
2. Tab **Sign-in method** → klik **Anonymous** → **Enable** → **Save**.

> Login anonim membuat tiap pengunjung punya sesi aman tanpa harus daftar.
> Cukup untuk alat internal. Untuk mengunci ke email tertentu, lihat catatan di `firestore.rules`.

### 4. Ambil konfigurasi Web App
1. Klik ikon ⚙️ (**Project settings**) → scroll ke **Your apps** → klik ikon **</>** (Web).
2. Beri nama app (mis. `dashboard`) → **Register app** (tidak perlu Hosting).
3. Salin objek `firebaseConfig` yang muncul, contohnya:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSyD...xxxx",
     authDomain: "honeybr-akuntansi.firebaseapp.com",
     projectId: "honeybr-akuntansi",
     storageBucket: "honeybr-akuntansi.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcd1234"
   };
   ```

### 5. Tempel ke `index.html`
1. Buka `index.html` dengan editor teks (VS Code / Notepad).
2. Cari blok **`const FIREBASE_CONFIG = {`** (di dekat atas, ada komentar ☁️).
3. Ganti nilai `PASTE_...` dengan nilai dari langkah 4.
4. (Opsional) Ubah `WORKSPACE_ID` bila ingin nama ruang kerja lain. **Akuntan dan
   owner harus memakai WORKSPACE_ID yang sama** agar melihat data yang sama.
5. Simpan.

> Nilai `firebaseConfig` **aman untuk dipublikasikan** — keamanan data dijaga oleh
> Firestore Rules, bukan oleh kerahasiaan config.

---

## B. Hosting & Share lewat GitHub Pages (gratis)

### 1. Buat repository
1. Login GitHub → **New repository** → nama mis. `dashboard-honeybr` → **Public** → **Create**.
2. Upload `index.html` (dan `README.md`, `firestore.rules`) lewat **Add file → Upload files** → **Commit**.

   > GitHub Pages mencari `index.html` di root. Pastikan namanya persis `index.html`.

### 2. Aktifkan Pages
1. Repo → **Settings → Pages**.
2. **Source**: `Deploy from a branch`.
3. **Branch**: `main` , folder `/ (root)` → **Save**.
4. Tunggu ±1 menit. Akan muncul tautan, mis:
   **`https://username.github.io/dashboard-honeybr/`**

### 3. Bagikan ke owner
Kirim tautan tersebut ke pemilik UMKM. Saat Anda input transaksi, dashboard owner
ikut ter-update otomatis (realtime). Badge ☁️ **Tersinkron** di pojok kanan atas
menandakan data sudah tersimpan di cloud Anda.

---

## C. Cara kerja sinkronisasi

- Seluruh pembukuan disimpan sebagai **satu dokumen** Firestore:
  `workspaces/<WORKSPACE_ID>`.
- Setiap perubahan (input jurnal, prive, edit akun) otomatis ditulis ke cloud,
  lalu disiarkan realtime ke semua yang membuka link.
- Tetap ada **cadangan lokal** (localStorage) dan tombol **Backup/Restore (JSON)**
  serta **Export Excel** di header — berguna bila offline.
- Status koneksi:
  - 💾 **Lokal** — Firebase belum dikonfigurasi (jalan offline saja).
  - ☁️ **Menyambung…** — sedang konek.
  - ☁️ **Tersinkron** — data aman di cloud.
  - ⚠️ **Cloud error** — gagal konek; data tetap tersimpan lokal.

---

## D. Batasan & tips

- **Last-write-wins**: jika akuntan dan owner mengedit di detik yang sama,
  perubahan terakhir menang. Untuk UMKM (umumnya 1 penginput) ini aman. Sebaiknya
  hanya akuntan yang input; owner cukup memantau.
- **Ukuran data**: 1 dokumen Firestore maksimal 1 MB (≈ ribuan baris jurnal) —
  lebih dari cukup untuk operasional bulanan. Lakukan Backup/arsip per tahun bila perlu.
- **Kuota gratis (Spark plan)** Firestore sangat besar untuk skala UMKM
  (puluhan ribu operasi/hari) dan tidak berbayar.
- **Reset data**: tombol ↺ Reset mengembalikan ke contoh Mei 2026 dan ikut
  menimpa cloud. Buat Backup dulu sebelum reset.

---

## E. Keamanan singkat

- Data hanya bisa diakses lewat sesi login (anonim) — tidak terbuka publik mentah.
- Siapa pun yang punya link bisa membaca/menulis (mode anonim). Bagikan link
  secukupnya. Untuk membatasi ke email tertentu, ikuti catatan di `firestore.rules`.
- `firebaseConfig` boleh publik; jangan pernah menaruh kredensial/Service Account
  (file JSON privat) di repo.
