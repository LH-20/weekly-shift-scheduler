# Weekly Shift Schedule — Supabase live version

This bundle contains:
- `index.html`: responsive app with live availability and shared published schedule.
- `supabase/functions/weekly-schedule/index.ts`: server-side API for reads, availability updates, and manager-only actions.

## Important security note

This version intentionally allows anyone with the shared link to change any employee's availability. Since employees do not log in, the app cannot verify who made a mark. The manager PIN protects renaming employees, clearing marks, and publishing schedules, but does not identify employees. If individual employee ownership is required, add per-employee access codes or authentication.

## A. Configure the database

The SQL you already ran should create `public.weekly_schedule_state` with `id`, `state`, and `updated_at`, enable RLS, grant anonymous SELECT, and create the public read policy. The table must be included in the `supabase_realtime` publication.

For the Edge Function, browser clients do not write directly to this table; the function uses the server-side service role key. Do not expose the service role key in HTML.

## B. Deploy the Edge Function

1. Install the Supabase CLI: https://supabase.com/docs/guides/cli
2. In a terminal, log in:
   `supabase login`
3. In this folder, link your project:
   `supabase link --project-ref YOUR_PROJECT_REF`
   The project ref is the subdomain in your Supabase project URL.
4. Set a long private manager PIN/password:
   `supabase secrets set MANAGER_PIN="replace-with-a-long-private-password"`
5. Deploy:
   `supabase functions deploy weekly-schedule`

The project URL and service role key are normally supplied to deployed Edge Functions by Supabase. If your function logs say a required environment value is missing, check the project's Edge Function secrets/configuration.

## C. Configure the HTML

Open `index.html` in a text editor and find:
- `PASTE_YOUR_SUPABASE_PROJECT_URL_HERE`
- `PASTE_YOUR_SUPABASE_PUBLISHABLE_KEY_HERE`

Replace them with the Project URL and publishable key from the Supabase dashboard. An older project may show an `anon` key instead. Never paste a `service_role` or secret key into this file.

## D. Publish

Upload `index.html` to a static host such as Cloudflare Pages, Netlify, or another HTTPS static hosting provider. Share the resulting HTTPS link with the team.

## E. Test

1. Open the link in two different browsers/devices.
2. Mark availability in one and check the other updates.
3. Refresh and confirm marks persist.
4. Unlock manager controls and test renaming, clearing, and publishing.
5. Confirm a visitor without the manager PIN cannot perform manager actions.
6. Test print and CSV export.

## Limitations to know

- The manager PIN is sent to the Edge Function for each protected operation over HTTPS. Use a strong PIN/password and do not share it.
- The HTML shows/hides manager controls for convenience only. The Edge Function enforces manager permission.
- Anyone with the public link can change any availability mark by design. For true per-employee access control, add employee authentication or per-employee codes.
- The app uses one shared state row. This is suitable for a small team; for larger or more concurrent teams, normalize employees and availability into separate tables and add conflict handling.
- The scheduler preserves the original greedy scheduling algorithm. It avoids assigning one person to multiple shifts on the same day, but it is not a mathematical optimizer and may leave shifts uncovered even if a different assignment could cover more.
