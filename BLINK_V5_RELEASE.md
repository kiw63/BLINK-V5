# BLINK V5 — Release, acceptance gates, and production deployment

**Project owner:** BLINK  
**Developer / technical support & bug reports:** https://t.me/Marvel_sh  
**Primary UI:** Indonesian; **timezone:** Asia/Jakarta  
**Front end:** static `docs/` deployable to GitHub Pages. **Backend:** private FastAPI, PostgreSQL, FFmpeg, Real-ESRGAN worker. These must be hosted separately.

## What is actually working and verified here

- Original BLINK logo; ~4-second cinematic animated landing screen with sequential logo, BLINK letters, “Selamat Datang,” light beams, orbit, layered glow, pointer parallax, reduced-motion setting, responsive mobile/desktop, Telegram contact.
- Real camera control, still capture, 3-second Live Photo (a separate still plus moving clip), and browser cinematic recording. Verified in Chromium with a *synthetic camera stream*, not physical phone hardware.
- Automatic local finishing after ordinary camera capture and the still image of Live Photo is now implemented, with downloadable files even before AI GPU is connected. Local pixel editing and sharpening also produces a real modified PNG. Chromium decoded a 3840×2160 output. **It is ordinary upscaling/interpolation, NOT neural 4K reconstruction.**
- Browser MediaRecorder generated actual moving WebM files, both Live Photo motion and a graded cinematic recording.
- Real FFmpeg/H.264 output at 3840×2160 with AAC audio was decoded by FFprobe. **That test uses FFmpeg color grading and resizing, NOT AI.**
- Backend photo/video queues, JWT scopes, free token reservation/refund, premium orders, QRIS proof, admin MFA and approval are implemented. The customer Studio polls payment status every 4 seconds while visible; public QRIS display is gated behind a valid logged-in order; admin Next.js interface receives SSE and polls as backup. A database integration test verifies that the customer sees admin approval and exactly 20 credits.
- **35 backend tests pass**, plus Chromium DOM/media smoke checks and a real FFmpeg 4K/AAC output check. `scripts/check_contract.py` and `scripts/check_static.py` pass.

## Hard blockers to calling this production-ready

1. This environment has **no NVIDIA CUDA GPU or installed Real-ESRGAN executable/weights**. AI inference could not be executed and the photo/video AI quality threshold could not be certified. **Backend refuses unverified AI jobs in production before charging tokens.**
2. No owner's GitHub authorization, public DNS/hosting, SMTP provider, private S3 object storage or payment provider/bank account integration is available here. No public deployment or money verification occurred.
3. Real Android/iPhone permission prompts, microphone, 4K native capture, iOS Safari MediaRecorder and device memory/thermal limits were not tested here.
4. The static GitHub Pages application cannot host an AI GPU, account database or protected admin UI. The private backend and Next.js admin must be deployed on separate secured infrastructure.
5. Tests do not prove a platform is free from all vulnerabilities or bugs. Commission a penetration test, WAF/rate limiting checks, backup restore drill and privacy review before serving customers or accepting money.

## Production AI acceptance procedure — required BEFORE a public AI launch

Install a GPU host with Vulkan support for the official `realesrgan-ncnn-vulkan` binary and **legitimate trained** `realesrgan-x4plus.bin` and `.param` files. Organize them under `infra/ai-models/bin/` and `infra/ai-models/models/` (see `infra/ai-models/`). Note: merely installing filenames is not proof that inference works.

After starting backend and worker:

```bash
# Must execute genuine 128×128 -> 512×512 trained-model inference,
# compare its pixels with ordinary Lanczos and save a hash-fingerprinted local report.
docker compose exec worker python -m app.verify_ai_engine
```

Both backend/worker share the private `/data/media/ai-verified.json` report. `/api/public/capabilities` reports `ai_gpu_verified:true` **only when the real-inference report is valid for the currently installed model and executable hashes**. Production photo, Live Photo and video jobs are rejected with HTTP 503 and zero token charges when this gate fails.

Next, prepare a **genuine clean 3840×2160 photo** representative of your customer's use case (reference test only, not the input to the model). Copy it into the worker's shared volume and run:

```bash
# Example: place an independently captured reference photo into the mounted volume.
# docker compose cp real-reference-4k.png worker:/data/media/real-reference-4k.png

docker compose exec worker python -m app.quality_gate \
  --reference /data/media/real-reference-4k.png \
  --min-psnr-gain 0.0 \
  --report /data/media/real-4k-quality-report.json
```

This deliberately creates a degraded 960×540 input, performs real Real-ESRGAN x4 inference, and compares the genuine 3840×2160 result with Lanczos interpolation against the independent 4K reference. It fails if PSNR does not beat that baseline. PSNR is *one diagnostic*, not sufficient by itself: inspect texture fidelity, faces, halos, motion blur, night scenes, frame flicker and temporal quality with a real collection of media. An enhancement system cannot reliably reconstruct detail that the camera never captured, and output pixel dimensions alone are not quality proof.

## GitHub Pages — front end

1. Use `BLINK-v5-GitHub-Pages.zip` and extract its **contents** at the repository root, or publish `/docs` from the fullstack source using GitHub Actions.
2. In GitHub → Settings → Pages, use branch `main` `/(root)` for the static archive, or **GitHub Actions** for fullstack source (workflow included).
3. Before enabling backend features, edit **only** the public `assets/config.js` with the HTTPS origin of the separately deployed API, e.g. `https://api.yourdomain.tld` (no path). **Never add admin passwords, JWT secrets, database credentials, AI API keys or SMTP passwords here.**
4. Backend `.env` must use the exact GitHub Pages `STUDIO_ORIGIN=https://YOURNAME.github.io` (origin only) and `STUDIO_PUBLIC_URL=https://YOURNAME.github.io/YOUR-REPO` (full link), plus proper HTTPS `PUBLIC_ORIGIN` for Next.js admin and private server.
5. Verify browser permissions from your real domain and devices. GitHub Pages cannot run Python/FFmpeg GPU worker code by itself.

## Backend — production basics

1. Copy `.env.example` to private `.env`, generate unique high-entropy secrets (`scripts/setup_dev.py` is for local development), set `BLINK_ENV=production`, `SECURE_COOKIES=true`, PostgreSQL, SMTP, private S3-compatible storage, accurate origins and strong MFA encryption key.
2. Place production API and Next.js admin behind HTTPS reverse proxies and CDN/WAF. Only expose intended HTTPS services; never expose PostgreSQL/Redis/GPU worker publicly. Configure rate limits, request-size limits, alerts, backups and recovery.
3. Install actual Vulkan GPU drivers and the Real-ESRGAN binary and weights. Provide necessary `/dev/dri`/GPU runtime access to the worker container. Verify the actual model command runs in the worker.
4. Run `docker compose up --build -d` and database migrations, then `docker compose exec worker python -m app.verify_ai_engine`. Verify `/api/public/capabilities` has `ai_gpu_verified:true`.
5. Bootstrap a new administrator **privately** with `python -m app.manage create-admin` using secure `BLINK_ADMIN_EMAIL`, `BLINK_ADMIN_PASSWORD` and `BLINK_ADMIN_NAME` environment variables, then enroll the printed TOTP secret in an authenticator. A previous account cannot be “logged into” unless its database exists and is connected. Do NOT commit shared passwords to GitHub.
6. Connect the QRIS supplied by the owner; admin must compare transfer reference and **money received** in the bank/provider before approving. A screenshot alone cannot authorize premium.
7. Conduct end-to-end tests on staging: user verification, payment submission, admin MFA approval, user premium status, failed jobs refunds, real AI output downloads, concurrent calls, upload abuse, 4K storage/VRAM/disk pressure, real phone camera/Live Photo/video, and recovery drills.

### Local development only

```bash
python3 scripts/setup_dev.py
# check .env; do not put development settings on a public server
docker compose up --build -d
# visit http://localhost:3000 for Next.js frontend; or serve docs/ locally for static studio
```

## Security note

The administrator password previously shared in the chat is not included in the repository or any public JavaScript. Use a new long secret (>=12 characters) in the backend and MFA. A public repository and GitHub Pages are readable by everyone.

## Audit/test commands

```bash
python -m pytest backend/tests -q
python scripts/check_contract.py
python scripts/check_static.py
python qa/browser_site_smoke.py
python qa/browser_media_smoke.py
python qa/browser_capture_smoke.py
python qa/ffmpeg_4k_smoke.py
```

Browsers/FFmpeg/Python must be installed for the QA scripts. `browser_site_smoke.py` uses in-memory module URLs because localhost navigation was blocked in this environment; it does not simulate backend AI results.

**Launch policy:** You can publish the static website to preview its animation and local features. Do not advertise BLINK as guaranteed AI HD++/4K, accept AI processing charges, or claim production security until the real GPU quality gate, staging integrations and physical-device tests pass.
