# Attendance Predictor (VibeCraft, Round 1: The Overworld)

A Minecraft-themed attendance dashboard that helps college students plan their classes without falling into detention. It runs as a single static page with no build step.

**Semester:** 29 Aug 2026 to 29 Nov 2026 · **Detention line:** 75% · **Safe target:** 90%

## Features

### Phase 1: The Core Calculator
- Pick one of the 10 class sections and enter your current attendance % per subject.
- The page reads today's date and lets you choose a future planning date.
- For every subject it shows:
  - total classes left in the semester,
  - how many of them you must attend to stay above 75%,
  - how many you must attend to reach or keep 90%,
  - the same numbers up to your planning date.
- **Irreversible Detention** alert: a flashing warning if 75% cannot be reached even by attending every remaining class.

### Phase 2: Visuals, Leaves and the AI Assistant
- **Charts:** hearts meter for overall attendance, a projection line chart (attend all, miss all, your simulated days), and a per-subject bar chart with 75% and 90% markers.
- **OD simulator:** add On-Duty, Medical or Absent days and see final percentages update instantly.
- **Attendance Advisor chatbot:** a floating chat button. Ask things like *"If I take a 3-day sick leave starting tomorrow, will my Chemistry attendance drop below 75%?"* The bot reads the dashboard data and runs the maths with the page's own functions.

## Run it

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

To deploy, upload the folder to GitHub Pages, Netlify or Vercel. No build command is needed.

## Load the real timetables

The page starts with **sample** timetables. Open **Timetable data and holidays** on the page and load your dataset file.

**CSV** (one row per class period):

```csv
section,day,period,subject
Section A,Mon,1,Maths
Section A,Mon,2,Physics
```

**JSON:**

```json
{
  "sections": { "Section A": { "Mon": ["Maths", "Physics"], "Tue": ["Chemistry"] } },
  "holidays": ["2026-10-02"]
}
```

Examples: [`sample-timetable.csv`](sample-timetable.csv) and [`sample-timetable.json`](sample-timetable.json). If the official dataset is in another layout (for example an Excel grid), convert it to this CSV shape.

## How the maths works

For each subject:

- `held` = classes from the semester start up to (not including) today
- `attended = percentage × held`
- `left` = classes from today (or tomorrow) to the semester end, skipping holidays
- `T = held + left`
- classes needed for a target `t` = `ceil(t × T − attended)`
- if that number is greater than `left`, the target is unreachable. For 75% that means **Irreversible Detention**.

The OD simulator treats **OD** as present, **Medical** as removed from the total (condoned), and **Absent** as missed.

## The chatbot

- Inside a Claude artifact, the bot uses Claude with two tools, `get_dashboard` and `simulate`, so the model never does the arithmetic itself.
- Anywhere else (GitHub Pages, localhost) it falls back to a built-in parser that answers "N-day leave" questions using the same `simulate` function.
- To use an LLM in your own deployment, add a small backend that calls an LLM API with those two tools and point the `send()` function in `index.html` at it. **Never put an API key in the front-end code.**

## Assumptions

- Each period in the timetable counts as one class.
- Only 2 Oct 2026 is listed as a holiday by default. Edit the holidays box on the page for your college.
- Today's classes count as still to come (untick the box if they are already over).

## Project structure

```
.
├── index.html               # the whole app (HTML, CSS, JS)
├── sample-timetable.csv     # sample data, 10 sections
├── sample-timetable.json    # same data in JSON form
├── LICENSE
└── README.md
```

## Tech

Plain HTML, CSS and JavaScript. Charts are hand-drawn SVG. Fonts: Press Start 2P and VT323 from Google Fonts.

## License

MIT
