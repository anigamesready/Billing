# Ani Games Billing — Direct Supabase Pre-Build v11.29.0

## Tujuan
Paket ini memindahkan jalur API runtime dari Netlify Functions ke Supabase. UI/UX master dipertahankan; perubahan berfokus pada transport API, sinkronisasi data, dan build metadata.

## Jalur runtime
- Aplikasi Billing PC (`v11-cloud.js`) memanggil Supabase Edge Function `https://wzkjpycjizexxcxrfdvr.supabase.co/functions/v1/ani-games-api`.
- Pairing, heartbeat, antrean command, sync event idempoten, pull rental, dan full rental snapshot memakai RPC Supabase yang sudah ada.
- Dashboard Admin/Billing Control dan laporan tetap menggunakan Supabase API/RPC langsung.
- IoT Blynk beralih ke Supabase Edge Function dengan validasi sesi Supabase Auth dan role Owner.
- Android WebView menyertakan UI master versi v11.29.0; sumber API di asset APK sama dengan sumber yang diuji statis.
- Kunci yang disertakan di client adalah publishable key publik. Tidak ada `service_role` atau secret key di source/app.

## Supabase production
Project: `wzkjpycjizexxcxrfdvr` (Billing Ani Games Ready).

- Edge Function `ani-games-api`: aktif, versi 6, `verify_jwt=false`.
- `verify_jwt=false` diperlukan karena panggilan perangkat memvalidasi token perangkat melalui RPC di payload; tindakan admin IoT tetap memeriksa JWT pengguna dan role Owner di dalam fungsi.
- Rute didukung: `/health`, `/auth/me`, `/device/enroll`, `/device/heartbeat`, `/device/ack`, `/device/sync`, `/device/pull`, `/device/rentals`, `/iot/status`, `/iot/power`.
- `/device/rentals` memakai RPC `api.device_rentals_snapshot`, yang mengembalikan rental untuk perangkat yang dipairing.

## Status migrasi data produksi
Backfill sudah dijalankan dan diverifikasi pada 10 Oktober 2026, tanpa menghapus atau membuat ulang data:

- 29 rental tetap ada; 0 rental dengan `device_id` kosong.
- 12 rental historis mendapat metadata audit `_migration_v11_29_device_backfill` dan ditautkan ke PC utama `Ani Games Ready` karena PC tersebut sebelumnya memiliki jumlah rental-terikat terbanyak (17; perangkat kedua 0).
- 3 sesi yang cocok dengan rental tersebut dan sebelumnya belum punya `device_id` ikut ditautkan.
- 7 transaksi yang terhubung ke rental-rental tersebut ikut ditautkan.
- 0 usage event berhasil dipasangkan melalui rental account; 4 usage event historis yang tidak punya `device_id` masih dibiarkan apa adanya agar tidak menebak asal PC.
- Satu sesi `free_test` berstatus `ended` masih tanpa `device_id`; itu adalah record tes historis yang tidak dipakai sebagai sesi aktif.
- Pemeriksaan sebelum paket: satu sesi aktif di PC utama, tidak ditemukan sesi aktif/paused ganda.

SQL backfill dan rollback bersyarat tersimpan di `docs/V11.29-DEVICE-ASSIGNMENT-BACKFILL.sql`. SQL itu adalah referensi/audit; backfill produksi sudah diterapkan, jangan jalankan rollback kecuali memang sengaja membatalkan migrasi.

## Keamanan IoT — satu konfigurasi manual yang masih diperlukan
Tambahkan secret `BLYNK_AUTH_TOKEN` ke **Supabase Dashboard → Edge Functions → Secrets** dengan nilai token Blynk yang saat ini digunakan. Jangan kirim token itu ke chat atau taruh di HTML/ZIP/source. Secret ini tidak dapat dibaca oleh sesi alat ini, jadi endpoint IoT akan mengembalikan `BLYNK_AUTH_TOKEN_NOT_CONFIGURED_IN_SUPABASE` sampai secret diisi. Fitur di luar IoT tidak bergantung pada secret itu.

## Hasil pemeriksaan
- Pemeriksaan sintaks JavaScript statis dilakukan pada source PC/Web dan skrip inline HTML.
- Referensi endpoint Netlify di jalur runtime yang diuji telah diganti.
- Source Edge Function aktif sudah dicek di Supabase dan mendukung snapshot rental.
- Akses HTTP langsung ke endpoint Supabase dan uji perangkat fisik tidak tersedia dari lingkungan build ini; karena itu status login, pairing, IoT fisik, timer, dan APK belum boleh dianggap lulus end-to-end sebelum dites di perangkat target.

## Build artefak
Paket ini adalah source **pre-build**, bukan file biner `.exe`/`.apk` yang sudah dikompilasi.

### PC Windows portable
Pada Windows dengan internet aktif, dari folder project PC:
```powershell
npm ci
npm run build
```
Electron Builder menghasilkan Windows portable `.exe` pada folder `dist/`.

### APK Android
Buka folder `android/` pada paket Android di Android Studio. Pasang Android SDK 35 dan gunakan Java 17 + Gradle 8.9 (sesuai Android Gradle Plugin 8.7.3). Lakukan Gradle Sync, lalu build APK dari menu **Build → Build APK(s)** untuk debug atau **Generate Signed Bundle / APK** untuk APK rilis. Source tidak menyertakan keystore rilis.
