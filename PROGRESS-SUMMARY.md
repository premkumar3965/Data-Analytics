# Project Data Analytics — Progress Summary

**This file is the single source of truth for "where did we leave off?"**
When you say "summarize today" or start a new session and want to resume,
this is the file to read first — it should always reflect the current
state, not a day-by-day log (that lives in `E:\Learnings\General\` instead).

## Goal & deadline

Build working, hands-on comfort with the Data Analytics stack before
joining **Infosys on 21 September 2026**:

- **Git & GitHub** — refresher, used daily via this repo/workflow
- **Python** — refresher
- **Pandas** — new
- **NumPy** — new
- **Matplotlib** — new
- **Power BI** — new (learning directly, not via R)
- **SQL** — already strong from previous role; light practice only, no
  dedicated course needed

## Status per topic

| Topic | Status | Notes |
|---|---|---|
| Git | In progress | `Git/gh-cli-commands.md` — reference notes on `gh auth`/`repo list`/`repo delete`/`auth refresh`/`logout`/`login`, written by the user himself while cleaning up his GitHub account |
| Python | Has prior practice | `Project Data Analytics/Python/Practice.ipynb` carried over from before this project started |
| Pandas | Not started | Folder ready |
| NumPy | Not started | Folder ready |
| Matplotlib | Not started | Folder ready |
| Power BI | In progress | `SuperStore_Analysis.pbip` (Orders + Date tables, relationship, 8 measures) saved in `PowerBI/`. "Overview" report page built: Year tile slicer + Region/Category dropdown slicers, a 5-metric KPI card, sales trend line chart, sales-by-category bar chart, 2 donut charts, and a sub-category profit table — all on a custom punchy-purple theme (`PowerBI/SuperStore_Analysis.Report/StaticResources/RegisteredResources/Superstore-Custom-Theme.json`). Pages 2 ("Category & Product Analysis") and 3 ("Customer & Regional Analysis") not built yet. Commands/techniques used are documented separately in `PowerBI/Commands/`. |
| SQL | Already strong | No dedicated folder planned; light refreshers only if needed |

## Environment / tooling status — all done

- GitHub CLI (`gh`) installed and authenticated as **premkumar3965**.
- Claude Code CLI (`claude`) installed and working from a normal terminal
  (the earlier sandboxing issue is resolved — reinstalled directly by the
  user in his own terminal).
- GitHub account cleaned up: 5 old repos deleted, only
  **[premkumar3965/Data-Analytics](https://github.com/premkumar3965/Data-Analytics)**
  remains.
- This folder is connected to that repo (reused rather than creating a new
  one). Old Coursera test files were removed and replaced with this
  project's structure — old history is still there if ever needed.
  `git init` done, `origin` set, multiple commits pushed to `main`.
- `CLAUDE.md` session-start instructions in place at both `E:\Learnings\`
  and this folder — new sessions auto-identify machine + interface and
  resume from this file.

## What to do next

1. **Power BI (in progress, pick this up first):** user is reviewing the
   "Overview" page (will report back). Once confirmed, build **Page 2 —
   "Category & Product Analysis"** and **Page 3 — "Customer & Regional
   Analysis"** per the plan already agreed (sub-category bars, profit-margin
   chart, discount-vs-profit scatter, top products table on Page 2;
   sales-by-state bar, ship-mode chart, segment×category table on Page 3),
   matching the same purple theme/card styling as the Overview page.
2. Continue Git refresher (already underway via `Git/gh-cli-commands.md`).
3. Python refresher, then Pandas/NumPy/Matplotlib in whatever order feels
   right.

## How this file should be maintained

- Update the **Status per topic** table whenever real progress happens on
  a topic (a script written, a concept practiced, a notebook filled in) —
  not for every small action.
- Keep this file itself short and current. Detailed day-by-day narration
  belongs in `E:\Learnings\General\`, one dated entry at a time — this file
  should only ever describe *where things stand right now*.
