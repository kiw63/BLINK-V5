# BLINK V5 — H-1 Final Verification Evidence

Date: 2026-11-02. Scope: one repository; keep all existing design, branding, animation, UI, admin, token, premium, payments and AI-media production code unchanged. This build changes *only* the CI workflow and release documentation; adds one QRIS integrity inspector and its tests.

## Local automated evidence — PASS

- Backend: **71/71 PASS** (`cd backend && python -m pytest -q`), 2 nonfatal FastAPI deprecation warnings.
- Launch gate logic + real machine-readable QR image test: **11/11 PASS** (`python -m pytest -q scripts/test_launch_gate.py scripts/test_qris_integrity.py`).
- Static repository, cross-service API contracts, public build checks: **PASS**.
- Browser Chromium smoke, run separately: desktop/mobile website, Photo Lab, intro animation, Premium panel: **PASS**; camera, Live Photo, video WebM: **PASS**; local **3840×2160 non-AI** export and video recording: **PASS**.
- QRIS image decoded with actual QR scanner: merchant **NANZERA**, country `ID`, currency IDR, static QR, CRC-16 `A53E` matches, bytes identical across customer/admin/backend. As a static QR, it contains **no embedded Rp20.000 amount**. This does not verify merchant account ownership, acceptability in a payment app, bank settlement or a live order.
- Public-facing visual source files, worker AI, payment API, and membership logic: **unchanged** from previous H-1 ZIP.
- GitHub workflow: added mandatory (failing if not configured) `production_smoke` job after GitHub Pages publish, checking the live public site, backend/API/CORS, trained-model preflight, QRIS bytes, Premium terms, SMTP configured and admin route. Set Actions variables `BLINK_API_BASE_URL` and `BLINK_ADMIN_URL` to HTTPS origins.

## External production evidence — NOT VERIFIED

- **AI photo/video 4K:** No genuine Real-ESRGAN binary, `.bin`/`.param` weights, real GPU inference or output-quality evidence in this runtime. Local 4K resolution export is not AI restoration.
- **Website:** No connected GitHub repository or active GitHub Pages URL of the owner; deploy and external HTTPS reachability cannot be checked here.
- **Real QRIS payment:** No payment provider or bank-account access and no real Rp20.000 settlement. The successful local order + admin MFA + token regression tests are not proof of real money received.

**GO-LIVE VERDICT: NOT CERTIFIED**. The three priority services cannot be declared all-pass without external owner-controlled infrastructure and a verified real bank transaction. The zip is a tested *release candidate*, not a certified production deployment. The `production_smoke` job further discloses missing live integration instead of pretending success; bank settlement and AI quality still require separate real-world evidence.

## Owner's shortest route to certification

1. Connect/publish BLINK repository on GitHub and enable Pages Actions. Set `BLINK_API_BASE_URL`, `BLINK_ADMIN_URL`.
2. Deploy backend HTTPS, admin Next.js, PostgreSQL/S3/SMTP and compatible AI worker with genuine model; execute `python -m app.verify_ai_engine` on AI host and benchmark output with real blurry media. Confirm `production_smoke` PASS.
3. Make one owner-authorized QRIS transaction of **Rp20.000**; verify it settled to **NANZERA** merchant in the payment app/bank before admin approval. Confirm 20 tokens + 30 days, exact reference and no double-credit. Check Android Chrome production page and actual download.

Do not treat any check that merely returns a success string as proof of real GPU, live hosting or settled bank funds.
