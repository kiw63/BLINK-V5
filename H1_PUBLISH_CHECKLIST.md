# BLINK V5 — H-1 publikasi: bukti, batas, dan langkah operasional

Tanggal penyiapan: 2 November 2026. Tujuan: satu repository, fokus pada website publik, AI 4K nyata, dan pembayaran QRIS nyata.

## Hasil yang BENAR-BENAR teruji di lingkungan lokal

- **71/71 backend regression tests lulus** (database uji, API, worker dengan test doubles untuk sebagian pengujian).
- **11/11 tes tambahan Launch Gate + QRIS integrity lulus** (logika validasi URL publik, penolakan status AI palsu, sifat pembayaran manual).
- `scripts/check_unified_repo.py`, `scripts/check_static.py`, `scripts/check_contract.py`, `scripts/prepare_pages.py` lulus.
- Pemeriksaan sintaks `docs/assets/*.js` dengan Node lulus.
- Chromium sintetis lulus: tampilan desktop & mobile, animasi pembuka, Photo Lab, file 3840×2160 **non-AI**, foto kamera, Live Photo, perekaman WebM.
- Browser QA diperbaiki agar memuat modul lokal Ultra 4K ketika diuji melalui `about:blank`. **Tidak ada perubahan desain, label, akun, token, Premium, admin atau QRIS.**
- QRIS asli berhasil **dipindai dan didekode**; merchant **NANZERA**, mata uang **IDR**, QR **statis**, CRC-16 benar; QRIS byte-identik di website, backend, dan frontend. QR statis tidak membawa nominal transaksi: pengguna harus mengisi **Rp20.000** di aplikasi pembayaran. Ini **bukan** bukti dana diterima.

## Hal yang tidak boleh diberi label LULUS

| Target | Status | Bukti yang diperlukan |
|---|---|---|
| AI Foto/Video 4K nyata | **BELUM LULUS** | Binary dan bobot Real-ESRGAN asli, GPU/Vulkan kompatibel, inference sukses dan quality benchmark foto/video nyata |
| Website **online pada URL GitHub** | **BELUM DIVERIFIKASI** | Repository telah diunggah; GitHub Actions hijau, URL website HTTPS bisa dibuka dari luar |
| Pembayaran **Rp20.000 benar-benar diterima** | **BELUM DIVERIFIKASI** | QRIS merchant benar, order masuk DB produksi, dana benar-benar terlihat di rekening/merchant penerima, admin MFA menyetujui sekali, saldo premium pengguna bertambah tepat 20 |
| Sistem produksi server | **BELUM DIVERIFIKASI** | Host backend/API HTTPS, PostgreSQL, SMTP, private storage S3, admin Next.js, backup & monitoring |
| Browser HP fisik | **BELUM DIVERIFIKASI** | Uji Chrome Android + Safari iPhone (jika perlu dukungan iOS) |

## Langkah untuk pemilik BLINK via HP Android

1. Ekstrak ZIP **BLINK-V5-H1-ONE-REPO.zip**; upload semua isi *ke root* satu repo GitHub. Jangan upload ZIP sebagai satu file repo. Pastikan folder tersembunyi `.github/workflows/` ikut.
2. Di repo: **Settings → Pages → Source: GitHub Actions**. Workflow kini memiliki job tambahan **`production_smoke`** sesudah deploy; job ini **sengaja gagal** bila API/admin belum dikonfigurasi atau preflight model AI asli belum terverifikasi. Coba link `https://USERNAME.github.io/NAMA-REPO/` lewat jaringan lain. Status hijau job lain bukan bukti ketiganya siap.
3. Siapkan host API HTTPS, PostgreSQL, SMTP, S3 privat dan server admin. **GitHub Pages tidak dapat menjalankan backend, database, worker GPU atau menerima uang.** Jaga `.env` dan kunci di luar repo; jangan tanam password admin yang pernah dibagikan.
4. `Settings → Secrets and variables → Actions → Variables` tambah `BLINK_API_BASE_URL` = origin HTTPS backend, contoh `https://api.domainanda.com`, dan **`BLINK_ADMIN_URL`** = origin HTTPS panel Next.js admin, contoh `https://admin.domainanda.com`. Untuk backend atur `STUDIO_ORIGIN=https://USERNAME.github.io`, `STUDIO_PUBLIC_URL=https://USERNAME.github.io/NAMA-REPO/` dan `PUBLIC_ORIGIN` untuk server Next.js/admin.
5. Di host GPU instal binary resmi `realesrgan-ncnn-vulkan` + `realesrgan-x4plus.bin`/`.param`; jalankan `cd backend && python -m app.verify_ai_engine`. Jalankan benchmark kualitas AI pada foto/video referensi. JANGAN klaim AI 4K hanya berdasarkan ekspor 4K interpolasi.
6. QRIS gambar terdekode untuk merchant **NANZERA**, IDR, statis, CRC valid (pemeriksaan kode, bukan validasi bank). Lakukan uji *pembayaran riil* nominal **Rp20.000** yang diisi pelanggan saat memindai, melalui rekening/merchant milik owner. Cocokkan referensi dana masuk **dan nominal** dengan order, periksa panel admin MFA, setujui satu kali, pastikan pengguna mendapat **20 token dan 30 hari**. Jangan gunakan screenshot bukti sebagai verifikasi dana.
7. Verifikasi HTTPS, login/verifikasi email, CORS, download, audit log, rate limits, backup, dan kebijakan privasi sebelum menerima pelanggan berbayar.

## Verifikasi integritas QRIS (otomatis di GitHub Actions)

```bash
# Di Ubuntu: sudo apt-get install libzbar0
# python -m pip install pyzbar
python scripts/verify_qris_integrity.py
python -m pytest -q scripts/test_qris_integrity.py
```

Validasi QR code mencakup pembacaan nyata QR JPG, struktur TLV, CRC-16, merchant `NANZERA`, IDR, dan konsistensi file antarmodul. **Tidak** membuktikan akun penerima aktif atau dana dapat ditarik.

## Launch Gate read-only

Script ini memeriksa URL publik, aset, konfigurasi API, CORS, status preflight model asli, QRIS identik, email/config, dan admin page. Ia **tidak** mentransfer uang, memotong token, atau mensimulasikan hasil AI.

```bash
python scripts/launch_gate.py \
  --site https://USERNAME.github.io/NAMA-REPO/ \
  --api https://api.domainanda.com \
  --admin https://admin.domainanda.com \
  --report BLINK_LAUNCH_GATE_RESULT.json
```

Job `production_smoke` di GitHub Actions menjalankan skrip di atas seusai deploy (membutuhkan variabel `BLINK_API_BASE_URL` dan `BLINK_ADMIN_URL`). PASS otomatis berarti hanya pemeriksaan HTTP/config **lulus**. `commercial_go_live_verified` akan tetap `false` sampai verifikasi foto/video dan dana nyata dilakukan terpisah. Jalankan pada perangkat atau runner dengan akses internet. Tidak ada akses ke GitHub/host/rekening produksi di sesi pengerjaan ini.

## Keputusan untuk H-1

- **Boleh** menerbitkan frontend sebagai preview/beta statis setelah workflow hijau dan halaman bisa diakses; kamera dan filter lokal dapat diuji.
- **Jangan** menerima pembayaran komersial atau menjanjikan AI foto/video supertajam 4K sampai GPU, server, transaksi nyata, keamanan, dan kualitas hasil *benar-benar lulus*. Jika salah satu gagal, nonaktifkan penawaran berbayar/akses AI, bukan membuat klaim palsu.
- Untuk pengaduan dan developer: https://t.me/Marvel_sh
