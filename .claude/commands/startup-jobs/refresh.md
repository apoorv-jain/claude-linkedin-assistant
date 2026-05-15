# REFRESH — startup-jobs

Scrapes all fund portfolio pages and reports newly discovered companies compared to what was found in the last `/startup-jobs find` run. Does NOT search for jobs or write to `job_tracker.csv`.

Use this when you want to see what new companies have been added to a fund's portfolio without running a full job search.

---

## Step 1 — Resolve fund scope

Use the `--fund` arg if given. Otherwise scrape all funds from `_shared.md`.

---

## Step 2 — Scrape every fund portfolio page

For each fund in scope, follow the exact same scraping procedure as `find.md` Step 2:

1. Navigate Chrome to the portfolio URL from `_shared.md`.
2. Call `mcp__Claude_in_Chrome__get_page_text`.
3. Scroll if JS-lazy. Fall back to `WebSearch` if sparse.
4. Extract: company name, sector, stage, location (all fields available).

Merge into a single deduplicated list.

---

## Step 3 — Report

Show the full company list as a markdown table grouped by fund:

`Company | Sector | Stage | Location | Fund`

Print totals: `Found <N> companies total across <M> funds.`

Tell the user:
> "Run `/startup-jobs find` to search for open roles at these companies."
