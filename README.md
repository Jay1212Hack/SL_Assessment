# SL Assessment

## Run locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

Without Supabase secrets, the app uses the checked-in `db.json` file as a local fallback.

## Persistent cloud storage

The deployed app stores the complete portal state in one Supabase `jsonb` row. Create the table in the Supabase SQL editor:

```sql
create table if not exists public.portal_state (
	id text primary key,
	payload jsonb not null,
	updated_at timestamptz not null default now()
);

alter table public.portal_state enable row level security;

create policy "portal state access"
on public.portal_state
for all
to anon
using (true)
with check (true);
```

In Streamlit Cloud, open **Settings > Secrets** and add:

```toml
SUPABASE_URL = "https://your-project.supabase.co"
SUPABASE_KEY = "your-supabase-anon-key"
```

The app loads the latest cloud state on every Streamlit rerun and writes registration, score, question, edit, and delete changes immediately to Supabase. Do not put the Supabase service-role key in Streamlit secrets; the anon key is sufficient for this table policy.

## Deploy on Streamlit Community Cloud

1. Push this repository to GitHub.
2. In Streamlit Community Cloud, choose **Create app** and select this repository's `main` branch and `app.py` as the main file.
3. Add the Supabase secrets shown above in the app settings.
4. Deploy or reboot the app.

The repository `db.json` is only seed/fallback data. New accounts are stored in Supabase and will not be committed back to GitHub.