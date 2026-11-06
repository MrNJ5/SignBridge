# Free worldwide deployment (Cloudflare Pages + Supabase)

1. Create a free Supabase project and configure `www/supabase-config.js`.
2. Run `supabase/schema.sql` in Supabase SQL Editor.
3. Create an admin user in Supabase Auth and add that user's UUID to `public.admin_users` using SQL.
4. Create a free Cloudflare account.
5. Install Node.js 22+ and run `npx wrangler login`.
6. From this folder, run `./scripts/deploy-cloudflare.sh signbridge-isl-to-speech`.
7. Use the `https://<project>.pages.dev` URL printed by Wrangler.

The site files are public; sample frames are public-readable for translation; only admins can write sample data. Never place the Supabase `service_role` key in the browser.

Limits: Supabase Free has 500 MB database, 1 GB file storage, 50 MB max object size and projects may pause after a week of inactivity. Cloudflare Pages Free has 500 builds/month and 20,000 files per site; static asset requests are free/unlimited. These limits can change; check provider dashboards.
