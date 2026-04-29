# Profile

Copy this file to `profile/profile.md` and fill in. Loaded by `/jobs find` and `/jobs outreach`.

`profile/profile.md` is `.gitignore`d by default — your filled-in copy stays local.

---

## LinkedIn identity

The name on your LinkedIn profile, exactly as it appears on `linkedin.com/in/me`. Used by `/jobs` Step 0 to verify the right account is active in Chrome before any flow runs.

```
LinkedIn name: <First Last>
LinkedIn URL:  https://www.linkedin.com/in/<your-handle>
```

## Background (used in outreach DMs)

A short pitch about you, in **third person**, that gets pasted into outreach messages. Keep it 1-2 sentences. No em-dashes.

Example:
> "I'm a senior data scientist with 5+ years of experience in experimentation and product analytics, currently at <Company>."

Yours:

```
<your pitch here>
```

## Target roles

The job titles `/jobs find` should search for. Group by resume profile if you have multiple resumes; otherwise just list 3-6 titles.

```
- Senior Data Scientist
- Senior Product Data Scientist
- Staff Data Scientist
```

## Top keywords / skills

The 10-15 keywords `/jobs find` uses for scoring job matches and the web-search sweep. Pull these from your resume.

```
SQL, Python, A/B testing, experimentation, causal inference, product analytics,
Snowflake, dbt, Airflow, Tableau, Power BI, Mixpanel, Amplitude, Databricks
```

## Locations

Where you'd take a job. `/jobs find` runs each location separately on LinkedIn.

```
- San Francisco Bay Area
- Remote
```

## Salary range (used for scoring + screening questions)

```
Min:  $150,000
Max:  $260,000
Note: Negotiable depending on total comp
```

## Anything else worth knowing

Optional. Put any non-obvious context here that should shape outreach or job filtering (e.g. "open to early-stage startups", "won't relocate from Bay Area", "primary stack is Python not R").

```
<notes>
```
