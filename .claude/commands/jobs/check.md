# CHECK (daily dashboard)

Read `job_tracker.csv`. Show the sections below as markdown tables. Empty section → "None."

---

**1. Needs outreach soon** (action required)
Jobs where `Referral Needed=YES`, `Referral Status=Outreach Pending`, `Referral Deadline` ≤ today+2.
→ "Run `/jobs outreach <Company>` to send a first DM to your existing connections there. Deadline approaching."

---

**2. Awaiting connection acceptance** (user action — outside this repo)
Jobs where `Referral Status=Connection Pending`. These are companies where this repo identified a 2nd-degree contact but cannot send the connection request itself.
→ "User to send connection requests manually on LinkedIn. Once accepted, run `/jobs outreach <Company>` to send the first DM."

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
  1. Connection Pending     <N> contacts across <K> companies (user to send request)
  2. Connection Accepted    <N> (ready for /jobs outreach)
  3. Cold DM Sent           <N> (awaiting reply)
  4. Reply Received         <N> (user to handle)

Terminal:
  5. <N> closed contacts (declined / no response / skipped)
```

For Stages 2 (ready for outreach) and 4 (replies waiting), list each contact with company + last action date so the user knows exactly which threads to act on.
