# BLINK V5 — AI Photo & Video Engine Update

**Scope lock respected:** exactly one existing file was modified (`backend/app/worker.py`). All 76 other original source files remain byte-for-byte identical, including `docs/`, its logo and animations, user/admin dashboards, authentication, QRIS, premium, database migrations, GitHub Pages workflows, and Docker configuration.

## Improvements

- `backend/app/ai_engine.py` — one Real-ESRGAN NCNN/Vulkan subprocess adapter for genuine file or directory inference, actual output/inferred-dimension verification, batch completeness, periodic worker heartbeat, safe subprocess timeouts, and strict production readiness checks. No browser resize or CPU filter is presented as trained AI.
- `backend/app/ai_photo.py` — photo quality measurements, conservative dark-photo exposure lift, EXIF orientation handling, bounded inference inputs for very large originals, and model-scale planning (2× or 4×). Any exposure adjustments are expressly **not** AI. Natural portrait textures are preserved; no unsupported GFPGAN face-restoration claims.
- `backend/app/ai_video.py` — inference of real video frames in batches to avoid loading the model for each frame. Validates every frame and deletes large intermediate files promptly. Supports 4K landscape and portrait layouts.
- `backend/app/worker.py` — integrated the modules into the existing job worker, encoded AI videos in bounded chunks, preserved source audio, applied moderate temporal denoising across the whole video, checked output dimensions/duration, allocated meaningful progress ranges to both parts of a Live Photo, and cleaned up partially uploaded AI media after failure. Token and job database logic are unchanged.
- `backend/tests/test_ai_media_engine.py` — 11 additional automated tests. **Test doubles are used solely for validating encoding, frame completeness, and error handling; they are not evidence of successful Real-ESRGAN inference.**
- `qa/verify_ai_video_gpu.py` — on-GPU smoke tool for actual model inference on video, 4K export, audio, decoded output, and comparison against ordinary interpolation. It does not claim measured perceptual improvement without human and dataset evaluation.

## Actual testing (this environment)

- Backend regression tests: **50 passed, 0 failed**, with 2 pre-existing FastAPI deprecation warnings.
- Existing API contract / static structure / one-repository checks: **passed**.
- Actual FFmpeg exported and validated 4K landscape and portrait H.264 video with AAC using **test-double upscaled frames**. This tests real encoding, not AI reconstruction.
- Actual trained-model inference preflight: **NOT PASSED — executable and `.bin`/`.param` model assets absent**. No compatible GPU was available. Genuine AI restoration quality, face restoration, GPU speed, video flicker on actual enhanced frames, and satisfactory real-world HD+/4K improvements are therefore **UNVERIFIED**.
- Android/iPhone camera testing, live S3/SMTP/QRIS, production server deployment/security testing: **not repeated** (locked systems outside the authorized changes).

## GPU setup required (unchanged one-repo deployment)

1. Provision a Linux GPU machine supporting Vulkan and a correctly configured container GPU runtime/device access. GitHub Pages hosts only the static website, not AI compute.
2. Install the **genuine** `realesrgan-ncnn-vulkan` executable and its compatible `realesrgan-x4plus.bin` and `realesrgan-x4plus.param` weights into the private `AI_MODELS_DIR` used by the existing Docker Compose mount (`infra/ai-models` is currently placeholders). Verify execution permission and any Vulkan library/driver requirements from inside the worker container. Do not commit model secrets or credentials.
3. Configure the existing `.env` according to the original README and run the existing Docker deployment. Set production server secrets and PostgreSQL/S3/SMTP separately, as required by the existing architecture.
4. Run `PYTHONPATH=backend python -m app.verify_ai_engine` **on the actual worker host with GPU access**, and ensure it writes a current verified report where the backend and worker can see it. The worker now refuses production AI inference when this model verification is absent or stale.
5. Perform reference-image AI quality evaluation with `PYTHONPATH=backend python qa/ai_quality_gate.py --reference /path/to/clean-4k-reference.png`. Evaluate a dataset of faces, blurred/low-light/compressed photos and motion scenes; report PSNR/SSIM or equivalent and subjective artifact review. A single metric cannot guarantee good results for every photo.
6. Run `PYTHONPATH=backend python qa/verify_ai_video_gpu.py --input sample.mp4 --output-dir ./gpu-qa --target 4k` on a short real video. Inspect the actual output, audio synchronization, temporal flicker, and file integrity. Do not enable paid AI until verified.

## Limitations that remain

- Model weights, GPU drivers and live server infrastructure are **not bundled**; they cannot be certified from this CPU-only sandbox.
- For a very low-resolution or severely blurry source, a 4K pixel export cannot guarantee recovered real-world details. AI super-resolution can hallucinate textures; quality needs a real validation dataset.
- Video input above 30 fps remains normalized to 30 fps by the original worker cost policy. The public API, token pricing, user interface, and database are unchanged.
- Temporal consistency currently uses real frame-by-frame AI reconstruction plus a temporal FFmpeg filter, **not** a learned recurrent video model. Dedicated temporal neural models could improve results but require additional real model deployment/evaluation.
- Face identity-aware restoration is not active without a dedicated compatible face-restoration model; pretending otherwise would be misleading.
- Live Photo motion is processed after the still image, with progress allocated to both stages; genuine enhancement requires the same working AI GPU.

**Release status:** AI engine integration optimized and regression-tested, but **NOT ready to advertise or sell guaranteed AI HD+/4K quality** until the genuine GPU quality tests succeed and the production server has been verified.
