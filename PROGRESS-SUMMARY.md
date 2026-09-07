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
| Power BI | In progress | Data model built for the Superstore dataset (`Orders` + `Date` tables, relationship, 8 measures) via a live connection to Power BI Desktop — see `PowerBI/superstore-data-model.md`. Report visuals not built yet — see next steps. |
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

1. **Power BI (in progress, pick this up first):** in the open Power BI
   Desktop window, do **File → Save As → Power BI project files (.pbip)**,
   saving into `PowerBI/Superstore-Sales-Report`. This splits the file
   into plain JSON/TMDL so the report visuals (charts, slicers, layout)
   can be authored directly as files instead of by hand — no API exists
   for that otherwise. Once saved, resume from there to build the actual
   report page (KPI cards, sales-by-category bar chart, sales-over-time
   line chart, region breakdown, sub-category table — see
   `PowerBI/superstore-data-model.md` for the fields/measures available).
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
