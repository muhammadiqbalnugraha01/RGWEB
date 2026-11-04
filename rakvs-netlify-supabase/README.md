# RAKVS GARASI — Supabase + Netlify

Migrasi frontend PHP/MySQL menjadi aplikasi Vite/JavaScript dengan Supabase PostgreSQL, Auth, Storage dan RLS. Ini adalah **implementasi baru**, bukan kompatibilitas langsung dengan file PHP lama. Logo asli tersedia pada `assets/logo-rg.jpg`.

## 1. Buat Supabase
1. Buat project baru di https://supabase.com/dashboard.
2. Buka **SQL Editor**, jalankan seluruh isi `supabase/setup.sql` **satu kali pada project baru**. Jangan jalankan schema.sql MySQL lama.
3. Buka **Authentication > Users > Add user** dan buat user admin dengan email serta password kuat. Nonaktifkan public signup bila hanya staf internal yang boleh masuk.
4. Salin UUID user admin tersebut. Jalankan SQL berikut dengan UUID asli:
   ```sql
   insert into public.profiles(id,name,role,branch_id)
   values ('UUID-ADMIN-DARI-AUTH', 'Administrator', 'admin_pusat', null);
   ```
5. Di **Project Settings > API Keys** ambil project URL dan **publishable/anon key**. Jangan pernah gunakan `service_role` atau secret key di frontend.

## 2. Jalankan lokal (opsional)
```
cp .env.example .env
# isi VITE_SUPABASE_URL dan VITE_SUPABASE_ANON_KEY
npm install
npm run dev
```

## 3. Deploy ke Netlify
1. Unggah folder ini ke GitHub (jangan unggah `.env`).
2. Netlify > **Add new project** > **Import from Git** > pilih repository.
3. Build command `npm run build`, publish directory `dist` (juga sudah ada di `netlify.toml`).
4. **Site configuration > Environment variables**: tambah `VITE_SUPABASE_URL` dan `VITE_SUPABASE_ANON_KEY`.
5. Deploy. Tambahkan URL Netlify di **Supabase > Authentication > URL Configuration > Site URL**.
6. Login dengan email dan password admin yang dibuat di Supabase Auth.

## 4. Pengguna staf
Buat akun email/password staf melalui Supabase Authentication > Users. Kemudian login admin ke aplikasi, buka **Manajemen Akun** dan isi UUID user dari Supabase, role, cabang dan izin part. Akun admin pusat tanpa cabang.

## Fitur yang tersedia
Dashboard, absensi GPS/selfie, data karyawan dan status keluar, stok part/mutasi/transfer atomic, purchasing dan persetujuan, service/rincian/garansi/voucher atomic, dokumentasi media privat, laporan CSV, manajemen cabang, profil akun, voucher terbit/batal. UI responsif.

## Batasan yang perlu diperhatikan sebelum operasional produksi
- Belum dilakukan uji integrasi dengan project Supabase/Netlify sungguhan; Anda perlu uji setiap fitur dan role sebelum penggunaan operasional.
- Migrasi **data historis MySQL** tidak otomatis; perlu proses ETL tersendiri. SQL `setup.sql` menyiapkan struktur baru dan 9 cabang awal.
- Absensi menggunakan GPS browser, bukan verifikasi radius/geofencing atau deteksi GPS palsu; selfie belum diverifikasi wajah.
- Form akun menggunakan UUID dari Auth, bukan pembuatan Auth user otomatis (demi menjaga service-role key tetap rahasia).
- Foto/video disimpan privat di Supabase Storage; fitur pratinjau dan penghapusan media belum tersedia di antarmuka.
- QR voucher dan pencetakan voucher belum diimplementasikan di UI baru; voucher dapat diketik manual.
- Pembatasan akses role/validasi tingkat database harus diaudit kembali sebelum menyimpan data personal karyawan/pelanggan secara produksi.
- Pembuatan service dan voucher menggunakan transaksi RPC; upload media terpisah dan mungkin perlu pembersihan manual jika penyimpanan metadata gagal.
- Kebijakan Storage mengizinkan file pada cabang pengguna, bukan hanya file yang diunggah sendiri. Sesuaikan bila butuh batasan lebih ketat.
- CSV saat ini menggunakan field database (bukan seluruh label export PHP sebelumnya).
- Untuk restore atau migrasi data, backup dan petakan ID serta user auth lama secara eksplisit.
