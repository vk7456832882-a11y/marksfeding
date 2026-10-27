# HPJC Mid-Term Marks Portal (GitHub Pages + Supabase)

Static frontend for GitHub Pages, with Supabase Auth and Postgres as the shared backend.

## Files
- `index.html` — portal interface
- `app.js` — login, marks entry, CSV import/export, admin tools
- `styles.css` — responsive styling
- `logo.jpg` — school logo extracted from the existing portal
- `config.js` — Supabase project URL and publishable key

## Publish on GitHub Pages
1. Create a **private GitHub repository** for the school portal if possible. A publishable Supabase key is safe to expose in a frontend, but private repository access reduces accidental exposure of student data/configuration notes.
2. Upload all files in this folder to the repository root. Do not upload Excel files with student details or exported marks to a public repository.
3. In GitHub, open **Settings → Pages**. Under Build and deployment, choose **Deploy from a branch**, select `main` and `/ (root)`, then Save.
4. Wait for the Pages URL to appear. Use HTTPS.
5. In Supabase Dashboard → **Authentication → URL Configuration**, add the GitHub Pages URL to **Site URL** and **Redirect URLs**. Example: `https://YOUR-ORG.github.io/YOUR-REPOSITORY/`.

## First-time database setup already completed
The Supabase project URL in `config.js` is `https://fsumwedroxgzjogzqytm.supabase.co`. The publishable key in the same file is intended for browser use. Never add a database password or `service_role` / secret key to frontend files.

The initial schema has been applied to project `fsumwedroxgzjogzqytm`. It creates `profiles`, `classes`, `subjects`, `teacher_assignments`, `students`, `examinations`, `subject_mark_rules`, `marks`, and `marks_audit`, with RLS enabled. Class 10/12 and 24 subject rows are seeded.

## Create the first administrator safely
1. In Supabase Dashboard → **Authentication → Users**, create an Auth user for the administrator (use your school email and a strong password). Do not enable open public sign-ups for staff.
2. Copy the user's UUID from the Auth Users page.
3. Open **SQL Editor** and run this, replacing the UUID and name. This is an administrator provisioning operation and should only be performed by a trusted project owner:

```sql
insert into public.profiles (id, full_name, role, active)
values ('PASTE-AUTH-USER-UUID-HERE', 'Portal Administrator', 'admin', true);
```

4. Sign in to the portal using that Auth user's email/password.
5. Use the Administrator tools to import `HPJC_Class_10_Students.csv` and `HPJC_Class_12_Students.csv`. Those files must be obtained from the provided data separately and must **not** be committed to a public repository.
6. Create the examination and set maximum marks for every class/subject combination before teachers start entering marks.
7. For each teacher, create an Auth user in Supabase Authentication → Users, then add a profile row with their UUID and role `teacher` using SQL Editor, for example:

```sql
insert into public.profiles (id, full_name, role, active)
values ('TEACHER-AUTH-USER-UUID', 'Teacher Full Name', 'teacher', true);
```

8. Use the portal's Assign teacher tool with the teacher's Auth UUID and the assigned class/subject.

## Security notes
- Do not commit student data, mark exports, passwords, service-role keys, or database credentials to GitHub.
- Supabase Row Level Security is the actual data access boundary; frontend visibility alone is not security.
- Teachers should be given only their assigned class/subject. Test using a non-admin account before real marks entry.
- Keep the project owner account protected with MFA. Do not enable unrestricted staff self-registration.
- This is an initial functional frontend; test all workflows and security policies with sample data before production use. In particular, teacher assignments and marks visibility should be tested against your school's exact workflow before importing real marks.
