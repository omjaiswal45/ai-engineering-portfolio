# AI-Assisted Engineering Portfolio
**Om Jaiswal** | omjai11022000@gmail.com 
---

## Feature: Financial Reporting Dashboard — ParivahanMitra CRM
**Stack:** Node.js, MongoDB, Express.js  
**AI Tool:** Claude (claude.ai)  
**Context:** Internal CRM for vehicle RC transfer case management

---

## Background

ParivahanMitra is a CRM I built for managing vehicle RC transfer cases. The ops team needed a financial dashboard — they wanted to see revenue and costs broken down by month and financial year, split between two case types: Transfer Completed and NOC Issued.

I used Claude throughout this feature — not to write code for me, but to think through the design before I touched the editor.

---

## Planning Phase — Before Any Code

My first question to Claude was about approach. I explained the data model — cases have a cost object with multiple fields, and a separate receipt array — and asked whether to aggregate in MongoDB or just fetch everything and calculate in JavaScript.

Claude recommended aggregation pipeline. I pushed back and asked why, since my collection wasn't huge at that point. The explanation made sense — the dashboard was being hit multiple times a day and a full collection scan every time would get painful fast. The team was already noticing slowness.

That conversation made me commit to the aggregation approach before writing a single line.

---

## First Real Bug — Wrong Totals

When I built the first version, the cost totals were wrong. Lower than expected.

I went back to Claude and explained the issue. After walking through my data model together, we figured out the problem — my aggregation was only summing the cost object fields and completely ignoring the receipt array. Each receipt had its own cost, and those were being silently skipped.

The fix involved nesting two separate reduce operations inside the aggregation — one for the cost object, one for the receipt array — and adding them together. Claude helped me with the MongoDB operator syntax here because nested reduces are genuinely tricky to write from memory.

What I made sure to do: I validated the fixed output against cases I knew the correct totals for before shipping. I didn't just trust that the fix was right.

---

## Performance Problem — 60 Seconds to 2 Seconds

After v1 shipped, the dashboard was still slow. Some months took over a minute to load.

I asked Claude what indexes I should add given my query patterns — filtering by status, by transfer date range, by NOC issued date. It pointed out I only had indexes on _id and carNumber, so every query was doing a full collection scan.

I asked a follow-up question about whether the compound index on status + NOC date was actually needed if I had individual indexes. Claude explained the query planner behavior — individual indexes can sometimes be intersected, but a compound index is more predictable for a query that always filters on both fields together.

I added four indexes. Load time dropped to under 2 seconds.

---

## Financial Year Date Logic — A Bug in AI's Output

Indian financial year runs April to March. So FY 2024-25 means April 2024 to March 2025. When a user selects "January," it should map to January 2025, not January 2024.

I asked Claude to help write this date logic. The first version it gave me had a subtle bug — months April through December were being assigned the wrong year. I caught it by manually testing with a few month values before it went anywhere near the API.

I fixed the year assignment logic myself. This was one of those cases where AI got the structure right but the business logic detail wrong — and I had to own that part.

---

## What I Learned About Working With AI

The pattern that worked for me on this feature:

- Use AI to validate architecture decisions before committing to them
- Use AI for complex syntax I'd have to look up anyway (MongoDB aggregation operators)
- Always test AI output against known values — don't assume correctness
- Business logic that's specific to the domain (Indian financial year rules, cost categorization) needs me to own it, not AI

The dashboard is in production and used daily by the ops team. The 60-second load time was a real complaint. The wrong cost totals would have been a real trust problem. Both got caught and fixed before they became bigger issues.

---

*ParivahanMitra was built at Car Wizard Pvt. Limited where I worked as a Software Engineer.*
