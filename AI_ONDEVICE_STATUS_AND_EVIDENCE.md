# BLINK V5 — On-Device AI photo/video dan Smart Backup (eksperimental)

## Fokus perubahan
Hanya pengolahan foto/video dan alat tes, tanpa merombak desain, logo, intro, dashboard admin, registrasi, pembayaran QRIS manual, token, Premium, atau kontrak backend yang sudah ada.

## Hasil yang diverifikasi

- **CNN terlatih asli:** `docs/assets/tiny-restore-v1.json` berisi parameter hasil pelatihan PyTorch dari gambar contoh dan degradasi Gaussian sintetis. Arsitektur 2 convolution layers, 4 hidden channels, ReLU + residual. Inference aktual di `docs/assets/ondevice-ai.js` pada CPU browser. **Bukan Real-ESRGAN, bukan model generatif, dan bukan model neural super-resolution kelas produksi**.
- **Foto AI-lite lokal:** model dipanggil dari `localEnhance()` pada pilihan foto 4K, menghasilkan JPG/PNG dengan dimensi hingga 3840×2160 (landscape) atau 2160×3840 (portrait). Detail sumber dipertahankan dengan blending. Tes Chromium 144×80 → 3840×2133 berjalan. Media tidak pernah diunggah untuk jalur lokal ini.
- **Video AI-lite lokal:** model diterapkan ke frame terpisah sebelum ekspor kanvas. Tes Chromium menghasilkan WebM nyata **3840×2160** dari 4 frame uji yang melewati CNN dan metadata diverifikasi `ffprobe`. Restorasi neural diproses pada resolusi kerja terbatas; hasil 4K adalah **interpolasi 4K yang mempertahankan bagian detail sumber**, bukan AI 4K yang menjamin rincian asli. Tidak ada hasil rekaman HP nyata yang diuji.
- **Smart Backup lokal:** hasil foto dapat disimpan/diambil dari IndexedDB (hash SHA-256 sumber di secure origin). Chunk video dipentaskan bertahap ke IndexedDB, kemudian dirakit sesuai urutan; jika storage ditolak, fallback RAM. Mock browser test `save/load`, `stage/recover/clean` lulus. **IndexedDB asli pada Android belum diuji.** Pemulihan video jika browser mati sebelum menghasilkan trailer format tidak dijamin.
- **Backend + integrasi:** 85/85 tes lulus. Browser smoke foto dan video standar, mobile/desktop lulus di Chromium. Tidak ada regresi visual yang disengaja.

## Gate kualitas (sengaja tidak dinyatakan lulus palsu)

1. **Pemulihan foto tanaman sangat buram: TIDAK LULUS.** Uji pada `/mnt/data/347686.jpg` memang menghasilkan foto 2160×2880 yang dapat diunduh, tetapi tetap buram dan tidak setara foto fokus. Hasil sebenarnya: `/mnt/data/BLINK-V5-OnDeviceCNN-Plant-Actual.jpg`.
2. **Video AI 4K kualitas profesional: BELUM LULUS.** Bukti encoding 3840×2160 sintetis bukan bukti restorasi nyata dari video HD rendah, kontinuitas frame pada HP, atau kualitas detail 4K.
3. **Smart Backup pada Android pengguna: BELUM LULUS perangkat nyata.** Browser mock menguji alur logika, tetapi batas quota, quota eviction, dan latensi pada Chrome Android belum dicoba.

## Pengukuran model pada gambar test di luar pelatihan
Pada skenario *synthetic Gaussian blur* pada gambar contoh yang tidak digunakan saat pelatihan, PSNR meningkat kira-kira **0,76 dB** (sigma=1), **0,52 dB** (sigma=1.8), dan **0,14 dB** (sigma=2.8). Ini peningkatan kecil dan **tidak membuktikan kemampuan mengembalikan foto sangat buram**. AI-lite mengandung komponen pelatihan neural yang nyata, tetapi belum memenuhi standar janji BLINK AI Ultra 4K.

## Pengujian ulang
```
python -m pytest -q backend/tests scripts/test_launch_gate.py scripts/test_qris_integrity.py
python scripts/prepare_pages.py
python scripts/test_ondevice_assets.py
python qa/browser_ondevice_ai.py
python qa/browser_ondevice_video_encode.py
python qa/browser_smart_backup_mock.py
python qa/browser_site_smoke.py
python qa/browser_media_smoke.py
python qa/browser_capture_smoke.py
```
Catatan: QA media menggunakan source sintetis dan virtual Chromium; web search/network browser sandbox diblokir. Inferensi Android WebGPU, model kelas Real-ESRGAN, dan uji pencairan/hosting nyata tidak dilakukan. Untuk pernyataan "LULUS" pada ketiga sasaran pengguna perlu model AI restorasi/super-resolusi yang jauh lebih kuat, evaluasi terhadap data asli, video sesungguhnya, serta tes Chrome Android dengan IndexedDB asli.
