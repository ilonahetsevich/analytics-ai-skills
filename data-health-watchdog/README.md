# Data Health Watchdog

> **Confidential work.** Company data and internal assets are withheld. All examples are fictional and were created to show the approach.

**Recurring task:** checking whether marketing dashboard data can be trusted before people use it.

An AI skill that checks marketing dashboards every business day for missing or unusual data. When something looks wrong, it posts an alert that explains the likely cause. It helps analysts and dashboard owners catch problems before stakeholders do. Each run produces one status message per dashboard, which is either an all-clear or an alert with a short diagnosis.

**My contribution:** I designed the detection logic, alert format and failure handling. I configured and validated each metric, tested the skill against historical data, wrote the detailed guide, and maintain the configuration.

**Status:** Used by a team. It runs on a schedule across three dashboards, and thresholds are in a two-quarter pilot.

---

## Why I built it

Data pipelines rarely fail with an error. Data stops arriving, or arrives in unusual amounts, and the dashboard looks normal. We usually found out when a stakeholder asked about a strange number.

Checking was manual and inconsistent. It depended on someone remembering to look and knowing what "normal" was for each metric, and that knowledge lived in a few analysts' heads.

A reusable skill made the checks run the same way every day. It also put the definition of "normal" in writing, so a colleague could run the checks without me.

---

## What it does

**Used when:** on a schedule every weekday, or on request ("is the data on this dashboard healthy?").

**Inputs and context:**

- A list of metrics per dashboard, each with its own date field, filters and thresholds.
- Shared rules for schedules and baselines.
- Read-only access to the BI tool.
- A messaging channel for each dashboard.

**Produces:** one message per dashboard per run. Every metric gets a status: Healthy, Warning, Critical, Skipped, Config drift or Check failed. New issues come first, each with a short explanation of what drove the change. Ongoing and closed issues each take a single line.

**Scope:** it catches missing data and sudden volume changes. It doesn't catch slow, long-term declines, and it never changes or refreshes anything in the BI tool.

---

## How it works

1. **Understand the request.** Works out which dashboard to check, and which checks apply based on the day. The freshness check runs daily, and the volume check also runs on Mondays.
2. **Apply relevant context.** Loads each metric's own date field, filters and thresholds. It uses complete periods only and leaves out the week still in progress.
3. **Carry out the task.** Queries the BI tool and applies two checks:
   - **Freshness:** zero rows for two business days on a metric that normally has data → Critical.
   - **Volume:** the last 4 weeks compared with a typical 4 weeks, calculated as the average of thirteen rolling 4-week totals.
4. **Check the result.** Separates three cases that look similar:
   - Zero rows: the data is missing.
   - No response: the check failed.
   - A field no longer exists: config drift. The skill doesn't guess a replacement.

   It also compares each issue with last week to classify it as New, Ongoing or Closed.
5. **Prepare the output.** Breaks each new issue down by source, channel and campaign, and states the likely cause. A person then decides whether it's a data fault to escalate or a business change to ask about.

---

## How I designed and refined it

- **Compare the last 4 weeks, not just last week.** One week on its own is too jumpy. If a big campaign launches one week and nothing launches the next, the numbers swing a lot even though nothing is broken. Looking at 4 weeks together evens this out, so alerts only fire when something has really changed.
- **Compare against the last 13 weeks.** To judge whether the last 4 weeks are normal, I need to know what a normal 4 weeks looks like. The skill looks back over the last 13 weeks to work that out. I first tried looking back a full year. That didn't work, because our numbers had grown over the year. Old, lower numbers made recent numbers look strange when they weren't. Thirteen weeks reflects where the numbers are now.
- **Freshness runs every day.** This check asks one simple question: did any data arrive? If data stops arriving, every report using it is wrong, so we need to know within a day or two. When everything is fine, it sends nothing, so checking daily doesn't create noise.
- **The volume check runs only on Mondays.** This check only uses full weeks. A new full week is only available on Monday, so running it on other days would just repeat Monday's result.
- **The two checks work together.** A missing week of data doesn't always show up in a 4-week comparison, because the other three weeks hide it. The daily freshness check catches that case, so the two checks together don't miss it.
- **Revision: too many repeat alerts.** Because the skill looks at 4 weeks, one unusual week stayed in view for four Mondays and caused four alerts in a row. I changed the skill to label each issue as New, Ongoing or Closed. Now the same issue only sends one full alert, and after that it shows as a single line.

---

## How I checked its quality

**What I checked:**

- Whether each metric matched the live dashboard.
- Whether it used the correct date field and aggregation.
- How many alerts the logic would have produced on historical data, and how many real events those alerts represented.

**Human review:** the analyst confirms the suggested cause, contacts the source owner or escalates to data engineering, and decides what to tell stakeholders. Threshold changes are also a human decision.

---

## Value

Monitoring went from occasional manual checking to a daily routine that can be handed over. When an alert arrives, it already says where to look, and a quiet run confirms the data arrived.

---

## Illustrative example

*Fictional example. No company data.*

**Request:** "Run the data health check for the campaign dashboard."

**Expected output:**

> 🟡 **Campaign Dashboard: 1 new, 0 ongoing**
>
> **Campaign Responses** | 6,500 vs 13,000 typical | −50% (warn 50% / crit 75%)
> Drop concentrated in two vendor-loaded sources. Other sources are steady. Likely a paused delivery, not a pipeline fault.
>
> ✅ Closed: Inquiries, back within threshold
> ✅ 6 of 7 metrics healthy · All metrics have recent data

---

## What I learned

The biggest lesson was that "is the data OK?" is really several different questions. Whether the data arrived and whether the amount is normal need separate checks. Missing data is urgent, while unusual volume needs more context.

The comparison period matters as much as the number itself. The same drop can look alarming or normal depending on what you compare it with.

Even on one dashboard, each metric needs its own rules. Data arrives on different schedules, some metrics keep updating after the period ends, and some sources only have weekly snapshots instead of daily data.

Alerts also need balance. Too many and people stop reading them; too few and real problems slip through.

Finally, explaining what happened is what makes an alert useful. It saves the analyst time and shows where to look next.

---

## Keywords

`data quality monitoring` · `data observability` · `anomaly detection` · `freshness checks` · `run-rate baselines` · `alert design` · `Tableau` · `Claude skills` · `MCP` · `Slack alerting` · `YAML configuration` · `workflow documentation`

---

## Confidentiality

All examples, numbers and alert messages in this case study are fictional or illustrative. They were recreated to show how the skill works and how I approached the problem, not to show real business results. The design decisions, logic and lessons described here reflect the real work.
