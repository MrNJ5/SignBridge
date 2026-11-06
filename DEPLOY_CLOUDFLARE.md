# Publish SignBridge for free with Cloudflare Pages + Supabase

Cloudflare Pages hosts the static website, while Supabase handles authentication, the sample catalogue and frame-file storage. No Firebase account is needed.

## Configure Supabase first

1. Create a free project at https://supabase.com/dashboard.
2. Copy the Project URL and public anon/publishable API key.
3. Add both to `www/supabase-config.js` (and `cloudflare-pages/supabase-config.js` for direct upload).
4. Run `supabase/schema.sql` in the Supabase SQL Editor.
5. Create an email/password admin under Authentication → Users, copy its UUID, then add it using SQL:
   `insert into public.admin_users (user_id) values ('ADMIN-USER-UUID');`

Do not paste a `service_role` key into the website. RLS policies allow public reads but restrict online sample writes to admin UUIDs.

## Direct-upload method

1. Create a free Cloudflare account at https://dash.cloudflare.com/sign-up.
2. Open Workers & Pages → Create application → Pages → Direct Upload.
3. Name the project `signbridge-isl`.
4. Upload `SignBridge_Cloudflare_Pages_Upload.zip` from the project root.
5. Deploy and use the public `https://signbridge-isl.pages.dev` URL Cloudflare assigns.

The zip contains the site-root files (`index.html`, tracker worker, PWA manifest, icons, Supabase config, service worker, `_headers` and `_redirects`). It is a static site; no build command is needed.

## CLI method

Install Node.js 22+, then from this folder run:

```bash
npx wrangler login
./scripts/deploy-cloudflare.sh signbridge-isl
```

Windows users can run `scripts\deploy-cloudflare.bat signbridge-isl` after authenticating Wrangler. The script deploys the `www/` folder.

## Free-plan limitations

Supabase Free currently offers 500 MB database, 1 GB Storage, and at most 50 MB per file. Free projects may pause after a week of inactivity. Check the Supabase dashboard for current usage. Cloudflare Pages Free currently lists up to 500 builds per month; check Cloudflare's current limits page. Free tier quotas can change.

No live public URL exists until you perform the Cloudflare deployment from an account you control.
