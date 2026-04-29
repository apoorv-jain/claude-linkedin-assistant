# CHECK (daily dashboard)

Read `job_tracker.csv`. Show the sections below as markdown tables. Empty section → "None."

---

**1. Needs outreach soon** (action required)
Jobs where `Referral Needed=YES`, `Referral Status=Outreach Pending`, `Referral Deadline` ≤ today+2.
→ "Run `/jobs outreach <Company>` to send a first DM to your existing connections there. Deadline approaching."

---

**2. Awaiting connection acceptance**
Jobs where `Referral Status=Connection Pending` and the connection request was sent ≥2 days ago (check `outreach/<Company>_contacts.md` for stage `1. Connection Pending` rows).
→ "Run `/jobs outreach <Company>` to check who's accepted. Step 5 of that flow re-checks each pending profile and sends the first DM to anyone who's now 1st-degree."

---

**3. Referral deadlines passed**
Jobs where `Referral Needed=YES`, `Referral Status` is `Outreach Pending` / `Outreach Sent` / `Connection Pending`, AND `Referral Deadline` ≤ today.
→ "Deadline passed — apply without a referral when ready, or mark `Got Referral` if one came in. Use `/jobs update <Company>` to change status."

---

**4. Ready to apply now**
Jobs where `Status=To Apply` AND (`Referral Needed=NO` OR `Referral Status` ∈ `Got Referral` / `No Referral` / `Declined` / deadline passed).
→ Surface these for the user to apply manually. This repo does NOT submit applications.

---

**5. Active outreach**
Read all `outreach/<Company>_contacts.md` files. Show counts by stage:

```
Active conversations:
  1. Connection Pending     <N> contacts across <K> companies (sent, awaiting accept)
  2. Connection Accepted    <N> (ready for /jobs outreach DM step)
  3. Cold DM Sent           <N> (awaiting reply)
  4. Reply Received         <N> (user to handle manually)

Terminal:
  5. <N> closed contacts (declined / no response / skipped)
```

For Stages 2 (ready for outreach) and 4 (replies waiting), list each contact with company + last action date so the user knows exactly which threads to act on.

**Connection requests sent in last 7 days:** count entries across all `outreach/<Company>_contacts.md` files where `Stage = 1. Connection Pending` AND `Last Action Date ≥ today − 7`. Show the raw count for visibility (no cap is enforced — LinkedIn's own throttling will surface as a CAPTCHA or "limit reached" notice during outreach if you hit it).
