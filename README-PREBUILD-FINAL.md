# Ani Games Billing — Web/Admin & Android source v11.29.0

Baseline UI Web master yang dikunci: `Billing-WEB-MASTER-v11.28.11-dashboard-rental-iot-diagnostics.zip`. Paket ini mempertahankan tampilan existing dan mengubah jalur API dari Netlify ke Supabase. Untuk APK WebView, lihat `android/README.md`. Untuk ringkasan migrasi data dan catatan keterbatasan tes, lihat `SUPABASE-DIRECT-MIGRATION-v11.29.0.md`.

Jika halaman web tetap di-host secara statis di Netlify, itu hanya hosting aset; runtime device/IoT API sudah diarahkan ke Supabase. `netlify/functions` disimpan sebagai sumber lama untuk referensi, tetapi tidak dipanggil oleh client runtime yang diperiksa.
