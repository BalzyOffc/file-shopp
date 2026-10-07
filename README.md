# Shopp Digital V5

Perubahan V5: login/daftar hanya **username + password** (tanpa nomor telepon), checkout cukup **username**, **approve/reject** pesanan berdasarkan bukti pembayaran, dan **kelola owner/admin** (tambah/ubah/hapus) dari Admin Panel.

## Setup baru
1. Edit `assets/js/config.js` (Supabase URL + anon key). `AUTH_EMAIL_DOMAIN` boleh dibiarkan.
2. Jalankan `supabase/schema.sql` di SQL Editor. Kolom pemilik pesanan yang digunakan adalah `orders.user_id` (bukan `buyer_id`), sesuai skema database lama dan seluruh aplikasi.
3. Supabase Auth > Providers > **Email**: aktifkan Email, **matikan "Confirm email"**. (Phone provider tidak dipakai lagi.)
4. Secret Edge Function: `GEMINI_API_KEY`. (Opsional `AUTH_EMAIL_DOMAIN` jika kamu ubah nilainya di config.js — harus sama.)
5. Deploy dua function: `supabase functions deploy ai-chat` dan `supabase functions deploy manage-staff`.
6. Daftar akun lewat `register.html`, lalu jadikan owner pertama:
   `update public.profiles set role='owner' where username='username_kamu';`
7. Upload isi folder ke Netlify.

## Upgrade dari V4
Jalankan `supabase/migration_v5.sql` (menggantikan migration_v4.sql), lalu langkah 3 dan 5 di atas. Migrasi menyelaraskan kolom pesanan ke `user_id`; jangan menjalankan schema.sql sebagai pengganti migrasi pada database produksi.
Catatan: akun V4 login pakai nomor telepon dan **tidak otomatis bisa login** dengan username. Hapus akun lama di Auth > Users dan daftar ulang.

## Alur pesanan
User checkout (username + bukti bayar) -> status `pending` -> admin/owner buka Admin > orders -> **Approve** atau **Reject** (alasan wajib) -> user melihat status di Profil; jika disetujui muncul link download.

## Manajemen staff
Admin > **staff** (khusus owner): tambah admin/owner, ubah username/password/role, hapus. Owner terakhir dan akun sendiri tidak bisa dihapus.

## Pemecahan masalah chat & produk
- Produk: Admin > products, pilih kategori saat menambahkan. Error penyimpanan ditampilkan di form. Pastikan produk aktif agar tampil di toko.
- Chat global: jalankan schema/migrasi yang sesuai agar policy `chat_insert` tersedia. Jika realtime tidak aktif, pengiriman tetap dapat dilakukan; pesan gagal menampilkan error.
- AI Chat: deploy folder `supabase/functions/ai-chat` dengan nama function `ai-chat`, lalu atur secret `GEMINI_API_KEY` di Supabase. Contoh: `supabase secrets set GEMINI_API_KEY=...` dan `supabase functions deploy ai-chat`. Jangan taruh API key di frontend.
- Database baru: `schema.sql`; database yang sudah berjalan: hanya `migration_v5.sql`.


## Perbaikan checkout username
Jika database lama masih memiliki kolom `orders.buyer_contact` dengan constraint NOT NULL, jalankan migration_v5.sql sekali. Migration tersebut akan menghapus kolom legacy `buyer_contact`; checkout versi ini hanya menggunakan username.
