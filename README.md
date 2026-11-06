# SignBridge V2.2 — ISL camera-to-speech starter

This release contains the camera-first website, admin sample recorder/editor, two-hand MediaPipe tracker in a background worker, local IndexedDB cache, Supabase cloud sample backend, Cloudflare Pages deployment files and a Capacitor Android wrapper.

## Free hosting stack

- **Cloudflare Pages Free** hosts the public website over HTTPS. Current Pages documentation lists Pages on all plans, Git/direct upload deployment, and 500 deployments per month on Free. Current platform limits are documented by Cloudflare; static assets are served from its global network. [Cloudflare Pages docs](https://developers.cloudflare.com/pages/).
- **Supabase Free** provides email/password authentication, PostgreSQL metadata and object storage. Current published Free quotas include 500 MB database, 1 GB file storage, 50,000 monthly active users and 5 GB cached + 5 GB uncached egress. Free projects can pause after one week of inactivity. A Free project's maximum upload size is 50 MB per file. See [Supabase pricing](https://supabase.com/pricing) and [file limits](https://supabase.com/docs/guides/storage/uploads/file-limits).

This is a prototype/free-tier configuration, not a guaranteed always-on production SLA. Sample frame files count toward the 1 GB storage quota and the app rejects any single sample JSON upload above 50 MB. Local recordings remain stored in IndexedDB on that device until uploaded.

## 1. Run locally

```bash
python -m http.server 8000 --directory www
```

Open `http://localhost:8000`, allow camera access, and wait for MediaPipe to load. The model and WASM files are fetched from public CDNs on first use.

Before Supabase is configured, the local-only demo credentials are:
- Username: `admin`
- Password: `Bridge@123`

These are not production credentials. Configure Supabase before publishing to the public internet; when cloud configuration is active, authentication is handled by Supabase.

## 2. Configure Supabase (shared samples + admin login)

1. Create a free project at [Supabase](https://supabase.com/dashboard).
2. Copy the project URL and publishable/anon key from Project Settings → API.
3. Put them into `www/supabase-config.js` as `url` and `anonKey`. The anon/publishable key is designed for browser use. **Never put the `service_role` secret key in this file.**
4. Open Supabase SQL Editor and run `supabase/schema.sql` in full. It creates the `samples` metadata table, an admin allow-list, the public-read `sign-samples` bucket and RLS policies.
5. In Authentication → Users, create the admin email/password account.
6. Copy that user's UUID. In SQL Editor, run this statement with the actual UUID:

```sql
insert into public.admin_users (user_id) values ('ADMIN-USER-UUID');
```

7. A signed-in user who is not in `admin_users` is treated as a customer (read-only). Only users in that table can create, update or delete online samples.

Sign labels/frame counts are stored in Postgres; frame arrays are stored as JSON objects in the `sign-samples` Storage bucket. Reads are public so visitors can translate signs; writes are restricted by RLS. The supplied SQL can be reviewed before execution.

## 3. Publish the site worldwide with Cloudflare Pages

**Direct upload (simplest):**
1. Create a free account at [Cloudflare](https://dash.cloudflare.com/sign-up).
2. Open Workers & Pages → Create application → Pages → Direct Upload (dashboard wording can vary).
3. Name the Pages project (for example `signbridge-isl`), then upload `SignBridge_Cloudflare_Pages_Upload.zip` from this project folder.
4. Deploy. Cloudflare assigns a public HTTPS address like `https://signbridge-isl.pages.dev`.

**CLI upload:**
1. Install Node.js 22+ on a computer.
2. Open a terminal in the project root and run `npx wrangler login`.
3. Run `./scripts/deploy-cloudflare.sh signbridge-isl` on macOS/Linux, or `scripts\deploy-cloudflare.bat signbridge-isl` on Windows.
4. Open the `*.pages.dev` address printed by Wrangler.

The full project includes a Pages source folder at `cloudflare-pages/`; the CLI deploy script uploads `www/`. Both contain the same app. Configure `www/supabase-config.js` before deployment (and make the same edit in `cloudflare-pages/supabase-config.js` if you use Direct Upload).

I have not published a public URL because deployment must be authorized from your Cloudflare account. The package is ready for that upload, but it is not already live.

## 4. Build and install the Android APK

The project includes Capacitor 8.6.0, camera permission setup, and a GitHub Actions workflow at `.github/workflows/android-debug-apk.yml` that builds a debug APK on a hosted GitHub runner.

**Hosted build:**
1. Create a GitHub repository and upload/push the contents of this folder to its `main` branch.
2. Enable GitHub Actions if asked.
3. Open Actions → **Build SignBridge Android APK** → Run workflow.
4. When the run succeeds, download its `SignBridge-Android-debug-APK` artifact. Extract it; the artifact contains `app-debug.apk`.
5. Copy the APK to your Android device and install it, allowing installation from that source when prompted.

**Local build** requires Node.js 22+, JDK 21, Android SDK Platform 36 and Build Tools 36.0.0:

```bash
./scripts/build-android.sh
```

The output path is `android/app/build/outputs/apk/debug/app-debug.apk`. The current execution environment has Java but no Android SDK/Gradle and cannot reach the Android SDK download host; therefore **an APK binary has not been built in this chat environment**. The hosted workflow is included to compile it after this project is in GitHub.

The CI build targets stable Android API 36. It is intended to run on supported phones including Android 17, but is not an API 37 target build. Android's API 37 tooling has separate newer Studio/AGP requirements; test on the actual device. A debug APK is for testing; a Play Store/release APK or AAB must be signed with a private keystore.

## 5. Camera and recognition notes

- MediaPipe runs in a module Web Worker (GPU first, CPU fallback); only one frame is processed at a time so old frames do not queue behind live movement.
- The camera preview renders independently, hand tracking supports up to two hands, and the skeleton is mirrored with the preview.
- New two-hand samples store 126 landmark values per frame (63 per hand). Older 63-value samples remain usable for one-hand examples.
- Actual inference FPS depends on hardware, GPU, camera and browser. No fixed FPS can be guaranteed across all devices.
- This remains a nearest-landmark sample matcher, not a trained general-purpose ISL translation model. It recognises signs similar to the admin-recorded samples.
