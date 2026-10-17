# Boiler Rooms: setup (Supabase + Vercel)

A static site (`index.html`) that talks to Supabase. All booking rules run in the database
(`supabase/schema.sql`), so they can't be bypassed from the browser.
**Reminder emails are intentionally not included yet.**

## 1. Supabase (about 10 minutes)
1. Create a project at supabase.com.
2. **SQL Editor** > New query > paste all of `supabase/schema.sql` > **Run**.
3. **Authentication > Providers > Email**: make sure it's enabled.
4. **Authentication > Emails > Templates > Magic Link**: edit the message so it contains `{{ .Token }}`
   (for example: `Your Boiler Rooms sign-in code is {{ .Token }}`). Students type this code in.
5. **Project Settings > API**: copy the **Project URL** and the **anon public** key into `config.js`.
   (The anon key is designed to be public. Never put the `service_role` key in this site.)
6. **Before real students use it:** **Authentication > Emails > SMTP Settings** (older dashboards: Project Settings > Authentication) > turn on **Enable Custom SMTP** and enter a mail provider's details
   (Resend, SendGrid, or Purdue's). Supabase's built-in sender is for testing only and allows very few
   emails per hour, which is why sign-in codes would stop arriving. This is for sign-in codes only, not reminders.

## 2. Vercel
1. Put this folder in a GitHub repo (or run `npx vercel` inside it).
2. Vercel > Add New > Project > import the repo. **Framework preset: Other.** No build command, no output directory. Deploy.
3. In Supabase > **Authentication > URL Configuration**, set **Site URL** to your Vercel URL.

## 3. Make yourself staff
Sign in once on the live site, then in the SQL Editor run:
```sql
update public.profiles set role = 'staff' where email = 'you@purdue.edu';
```
Reload: a **Staff** item appears in the menu (help inbox, cancel any booking, room hours, closed dates, usage + CSV).
Repeat for each staff member.

## What the server enforces
@purdue.edu accounts only | no double-booking | no past times | weekday closures (weekends open) and closed dates |
2 reservations per week (Sunday to Saturday) | 6 upcoming max | check-in window (15 min before to 15 min after start) |
no-show slots auto-release | students see only first names (never emails or ids) |
help requests readable by staff only | "Delete my data" removes the whole account.

## Later
- **Reminder emails:** bookings have exact times in the database. Add a Supabase Edge Function on a schedule (cron) that emails via Resend.
- **Purdue sign-in (CAS):** submit Purdue's Single Sign-on Request Form through your sponsoring office or department. Swap out the email-code sign-in once approved.
- **Free-tier note:** Supabase pauses free projects after about a week with no activity. Restart it from the dashboard or use the $25/mo Pro plan.
