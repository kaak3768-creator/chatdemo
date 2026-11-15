# ChatKu Tahap 1: Chat Live dengan Supabase + Vercel

Hasil akhir: website chat yang bisa dipakai kamu dan teman secara live.
Fitur tahap ini: daftar/login (username + password), tambah kontak lewat username,
chat 1:1 real-time, status online, jumlah pesan belum dibaca, dan edit profil (nama + info).

## Isi folder

| File | Fungsi |
|---|---|
| `index.html` | Seluruh tampilan dan logika ChatKu |
| `config.js` | Tempat kamu mengisi alamat dan kunci Supabase |
| `supabase.sql` | Membuat tabel, aturan keamanan, dan fitur realtime di database |
| `README.md` | Panduan ini |

## Cara kerjanya (singkat)

```
Browser kamu  <-->  Supabase (login + database + realtime)  <-->  Browser teman
      ^
      |  Vercel hanya menyajikan file index.html
```

Tidak ada server yang harus kamu jaga. Vercel menyajikan halaman webnya,
dan Supabase mengurus akun, penyimpanan pesan, dan pengiriman pesan secara live.

---

## Langkah 1: Buat project Supabase

1. Buka **supabase.com**, daftar (bisa pakai akun GitHub), lalu klik **New project**.
2. Isi nama project (misalnya `chatku`) dan buat **Database Password** (simpan, tapi tidak dipakai di kode).
3. Untuk **Region**, pilih yang terdekat dengan Indonesia (misalnya Singapore) supaya lebih cepat.
4. Klik **Create new project** dan tunggu 1 sampai 2 menit sampai selesai.

## Langkah 2: Matikan konfirmasi email (WAJIB)

ChatKu memakai username, bukan email asli. Di balik layar, akun dibuat dengan email palsu
`username@chatku.app`. Supabase tidak boleh meminta konfirmasi email, kalau tidak, tidak ada yang bisa login.

1. Di dashboard, buka **Authentication**, lalu bagian pengaturan **Email** (lokasi menu bisa berubah antar versi Supabase).
2. Matikan **Confirm email**, lalu simpan.

> **Catatan:** karena emailnya palsu, **fitur "lupa password" tidak ada**.
> Kalau ada yang lupa password, kamu bisa mereset manual lewat **Authentication > Users** di dashboard.

## Langkah 3: Jalankan SQL

1. Buka **SQL Editor** di sidebar Supabase, lalu klik **New query**.
2. Buka file `supabase.sql`, salin **semua** isinya, tempel ke editor.
3. Klik **Run**. Hasil yang benar: tertulis **Success. No rows returned**.
4. Cek di **Table Editor**: harus ada tiga tabel, yaitu `profiles`, `contacts`, dan `messages`.

Kalau muncul error `already exists`, berarti SQL sudah pernah dijalankan. Jangan dijalankan dua kali.

## Langkah 4: Ambil kunci dan isi `config.js`

1. Buka **Project Settings > API** (atau **API Keys** pada tampilan baru).
2. Salin **Project URL** (contoh: `https://abcdxyz.supabase.co`).
3. Salin **anon public key** (kunci panjang berawalan `eyJ...`).
   Pada dashboard terbaru, kunci ini mungkin diberi nama **publishable key**. Itu yang dimaksud.
4. Buka `config.js` dengan Notepad atau VS Code, ganti dua nilai:

```js
window.CHATKU = {
  SUPABASE_URL: "https://abcdxyz.supabase.co",
  SUPABASE_ANON_KEY: "eyJ....kunci-panjang-kamu....",
  EMAIL_DOMAIN: "chatku.app"
};
```

> **JANGAN** memakai `service_role` / **secret key**. Kunci itu melewati semua aturan keamanan.
> Kunci anon aman ada di kode publik karena dilindungi aturan RLS di `supabase.sql`.

## Langkah 5 (opsional): Tes di laptop

Membuka `index.html` dengan klik dua kali kadang bekerja, tapi cara yang lebih pasti:

- **VS Code:** pasang ekstensi **Live Server**, klik kanan `index.html`, pilih **Open with Live Server**.
- **Atau** langsung lewati ke deploy (Langkah 6 dan 7), lalu tes di alamat Vercel.

## Langkah 6: Upload ke GitHub

1. Buka **github.com**, daftar atau login, lalu klik **New repository**. Beri nama `chatku`, pilih Public atau Private, klik **Create**.
2. Di halaman repo yang kosong, klik **uploading an existing file**.
3. Seret keempat file (`index.html`, `config.js`, `supabase.sql`, `README.md`) ke halaman itu.
4. Klik **Commit changes**.

## Langkah 7: Deploy ke Vercel

1. Buka **vercel.com**, login dengan GitHub.
2. Klik **Add New > Project**, pilih repo `chatku`, klik **Import**.
3. **Framework Preset:** `Other`. Kosongkan Build Command dan Output Directory.
4. Klik **Deploy**. Setelah selesai, kamu dapat alamat seperti `https://chatku-xxxx.vercel.app`.

Setiap kali kamu mengubah file di GitHub, Vercel otomatis memperbarui website.

## Langkah 8: Tes live

1. Buka alamat Vercel di jendela biasa, klik **Daftar**, buat akun `andi`.
2. Buka alamat yang sama di **jendela Incognito** (atau HP temanmu), daftar akun `budi`.
3. Di akun `andi`, ketik `budi` di kolom "Tambah kontak", klik **Tambah**.
4. Kirim pesan. Di jendela `budi` pesan harus muncul langsung tanpa refresh, dan sebaliknya.

---

## Masalah yang sering muncul

| Gejala | Penyebab dan solusi |
|---|---|
| Tulisan "Belum dikonfigurasi" | `config.js` belum diisi, atau file lama masih di cache. Isi lalu muat ulang (Ctrl+F5). |
| "Akun dibuat, tetapi Supabase meminta konfirmasi email" | Langkah 2 belum dilakukan. Matikan **Confirm email**. |
| "Domain email ditolak Supabase" | Ubah `EMAIL_DOMAIN` di `config.js` ke domain lain, misalnya `chatku.id`. Akun yang sudah ada harus dibuat ulang. |
| "Username sudah dipakai atau formatnya tidak valid" | Username 3 sampai 20 karakter: huruf kecil, angka, garis bawah. Atau memang sudah dipakai. |
| "Profil tidak ditemukan" | `supabase.sql` belum dijalankan, atau dijalankan setelah akun dibuat. Hapus user di **Authentication > Users**, lalu daftar ulang. |
| Pesan terkirim tapi tidak muncul live di sisi teman | Pastikan baris terakhir SQL (`alter publication supabase_realtime ...`) sukses. Cek juga di **Database > Replication** bahwa tabel `messages` aktif. |
| "Koneksi live bermasalah" | Muat ulang halaman. Cek apakah project Supabase gratis kamu sedang dijeda karena lama tidak dipakai, lalu aktifkan lagi dari dashboard. |
| Halaman putih di Vercel | Pastikan `index.html` ada di folder utama repo, bukan di dalam subfolder. |

## Keamanan (baca sebentar)

- Aturan **Row Level Security** di `supabase.sql` memastikan orang hanya bisa membaca pesan yang mereka kirim atau terima.
  Jangan mematikan RLS di dashboard.
- Pesan tersimpan di database Supabase kamu dan **tidak terenkripsi ujung ke ujung**. Kamu sebagai pemilik project bisa melihatnya.
  Jangan menjanjikan privasi total ke pengguna, dan jangan dipakai untuk data sensitif.
- Daftar username bisa dicari oleh semua pengguna yang login (itu yang membuat fitur "tambah kontak" bekerja).
- Batas gratis Supabase dan Vercel bisa berubah. Cek halaman harga masing-masing sebelum mengundang banyak orang.

## Belum ada di tahap 1 (rencana berikutnya)

1. **Foto profil** (Supabase Storage), kirim foto, balas pesan, reaksi, edit/hapus pesan, centang baca.
2. **Grup**, admin, izin anggota, dan tautan undangan.
3. **Telepon suara** (WebRTC, sinyalnya lewat Supabase Realtime, plus server TURN).
4. **Jadi APK** (PWABuilder atau Capacitor) beserta notifikasi push.
