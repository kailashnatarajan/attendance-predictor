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

## Timetable data

The page already contains the college timetables (SRM Institute of Science and Technology, Tiruchirappalli, Faculty of Engineering and Technology), typed in from the provided PDFs. The same data is in [`timetables.csv`](timetables.csv) and [`timetables.json`](timetables.json).

Sections included (14):

| Year | Sections |
|------|----------|
| I (2024-25 sheets) | ECE-A, ECE-B / EEE, ECE-DS, Biotech-B, Biomed (from the Biotech-B sheet) |
| II (2026-27) | BME, ECE-DS A, ECE-DS B |
| III (2026-27) | BME, ECE-A, ECE-B, ECE-DS |
| IV (2026-27) | ECE-A, ECE-B |

To use different data, open **Timetable data and holidays** on the page and load a file.

**CSV** (one row per class period):

```csv
section,day,period,subject
III ECE-A,Mon,1,Machine Learning for All
III ECE-A,Mon,2,Microprocessor & Microcontroller
```

**JSON:**

```json
{
  "sections": { "III ECE-A": { "Mon": ["Machine Learning for All", "Microprocessor & Microcontroller"] } },
  "holidays": ["2026-10-02"]
}
```

### How the PDFs were read
- Slot letters (A, B, C ...) were replaced with the subject names from each sheet's subject table.
- Every period a class occupies counts as one class. A two-period lab counts as two.
- Blank cells and lunch have no class.
- Labs and non-slot classes (Workshop, NSS, Yoga, German, Japanese, CDC and so on) are separate subjects.
- In IV ECE, the lab period is counted under *Computer Communication & Network Security*, because it shares that course code.
- The I ECE-B / EEE sheet has one shared cell for the ECE and EEE subjects, so it is one section.
- The Biotech-B sheet is split into two sections, Biotech-B and Biomed, because their subjects differ in the shared cells.

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
- Only 2 Oct 2026 is listed as a holiday by default, because the dataset has no holiday list. Edit the holidays box on the page for your college.
- Today's classes count as still to come (untick the box if they are already over).

## Project structure

```
.
├── index.html          # the whole app (HTML, CSS, JS, timetables built in)
├── timetables.csv      # all 14 sections, one row per class period
├── timetables.json     # same data in JSON form
├── LICENSE
└── README.md
```

## Tech

Plain HTML, CSS and JavaScript. Charts are hand-drawn SVG. Fonts: Press Start 2P and VT323 from Google Fonts.

## License

MIT
