> **Pembaruan teknis 2 November 2026:** Setelah dokumen ini ditulis, modul CNN AI-lite lokal dan backup IndexedDB ditambahkan. Baca `AI_ONDEVICE_STATUS_AND_EVIDENCE.md` untuk status terbaru. Fitur AI Ultra 4K berkualitas penuh **masih belum lulus**, meskipun inferensi CNN kecil dan encode video 4K sintetis berhasil.

# BLINK V5 — H-1: manual QRIS, katalog akun, dan rancangan AI HP

## Katalog admin dan registrasi — lulus tes lokal

- Registrasi `/register` (Next.js) dan form registrasi di GitHub Pages tetap ada.
- Setiap akun yang terdaftar harus melalui verifikasi email SMTP sebelum dapat masuk; sistem akun dan autentikasi tidak diganti.
- Backend tetap mewajibkan MFA untuk admin. Akun biasa tidak bisa membaca katalog pelanggan.
- QRIS adalah **manual**: user buat order Rp20.000, membayar, mengirim screenshot; admin membuka bukti, lalu **secara manual** memeriksa mutasi rekening penerima. Jika tidak ada dana atau bukti palsu, admin menolak disertai alasan. Screenshot saja **tidak** memicu akses Premium.
- Admin baru bisa menyetujui saat `funds_verified=true`, nomor referensi mutasi rekening terisi dan unik, dan order berstatus `submitted`. Akun mendapat 20 token dan 30 hari hanya setelah approve. Ulangi approve tidak menggandakan token.
- Menu `/admin/users` dan `/admin/subscriptions` kini menampilkan katalog berdasarkan status **Premium Aktif**, **Pernah Premium (kedaluwarsa)**, dan **Gratis (belum pernah bayar)**, termasuk jumlah transaksi disetujui, email, token, dan masa berlaku. Pagination membantu melihat semua akun, bukan hanya 200 pertama.
- Endpoint tambahan hanya untuk admin: `GET /api/admin/account-catalog?status=all|premium|former|free&page=1&page_size=50`; respons memuat `items`, `counts`, `total`, `page`, `page_size`, `status`.
- Tidak ada perubahan tabel database, kontrak API yang lama, harga, aturan token, logo, animasi, atau frontend GitHub Pages.

## BLINK Ultra 4K AI on-device — pilihan pengguna, BELUM LULUS produksi

Kesepakatan arsitektur: model AI benar-benar berjalan di **CPU/GPU HP pengguna** (ONNX Runtime Web / WebGPU bila kompatibel, WASM sebagai alternatif). Website GitHub Pages hanya mengirim kode dan bobot model serta mengatur antrean; **website statis bukan GPU tambahan**. Smart Backup berarti hasil intermediate dan checkpoint disimpan **lokal** (OPFS/IndexedDB) untuk mengurangi lonjakan penggunaan RAM dan membantu pemulihan progres. Processing tiling dan adaptive inference diperlukan, terutama video.

Batasan penting: mesin `docs/assets/ultra4k.js` yang saat ini berfungsi adalah **upscaler NON-AI** (WebGL2/Canvas2D). Backend `backend/app/worker.py` masih mempunyai alur AI Real-ESRGAN melalui layanan worker. Menyebut keduanya sebagai AI Ultra 4K on-device adalah keliru. Paket ini tidak berisi file model AI on-device terlatih, sehingga fitur AI on-device tidak bisa dinyatakan lulus hanya karena tes katalog/pembayaran berhasil.

Untuk dinyatakan lulus produksi, operator masih perlu memasang model AI on-device yang valid dan terlatih (dengan hak distribusi), menguji inferensi nyata pada Android/WebGPU/WASM, memperlihatkan hasil foto nyata yang membaik tanpa artefak, dan menguji video serta limit memori. Website publik dan alur SMTP/QRIS juga harus diverifikasi pada URL dan rekening riil. **Jangan aktifkan pembayaran berbayar jika hasil AI yang dijanjikan belum berfungsi**.

## Hasil pengujian lokal

- Backend keseluruhan: 74/74 passed (termasuk tiga tes katalog manual QRIS baru).
- Launch Gate dan QRIS image: 11/11 passed.
- Pemeriksaan integrasi kontrak/static: lulus.
- Frontend TypeScript: pemeriksaan sintaks TSX sukses; build Next produksi belum diuji dalam sesi ini karena dependensi npm belum tersedia.
- URL publik, inferensi AI nyata, SMTP nyata, dan dana QRIS masuk: tidak dapat disertifikasi dengan tes lokal.
