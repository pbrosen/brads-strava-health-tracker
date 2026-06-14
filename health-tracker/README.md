# Health Tracker

A personal daily **health tracker** for Claude Cowork — log your food, exercise, and
digestive (gut) symptoms by chatting with Claude, with a visual dashboard. Everything is
stored as **plain Markdown files in a folder on your own computer**. Nothing is uploaded
anywhere.

It does three things:

1. **Food + calories.** Tell Claude what you ate ("turkey sandwich and chips for lunch")
   and it logs it with estimated calories and macros, learning your common foods over time.
2. **Exercise + Strava.** Log manual workouts (weights, yoga) by chat or in the dashboard.
   Connect **Strava** and Claude folds your runs/rides into your burn and net calories.
3. **Digestion tracking.** Log gut episodes (diarrhea, loose stool, constipation,
   gas/bloating) with severity and Bristol score, then use the dashboard's
   **"Find food correlations"** button to see which foods tend to precede your symptoms.

---

## Setup — do these in order

Follow these top to bottom. Steps 1 and 2 must happen **before** you install the plugin,
or the dashboard's file-saver won't start.

### 1. Install Node.js  (skip if you already have it)

The dashboard saves through a tiny helper that runs on Node.js.

- Easiest: open Cowork and say **"Help me install Node.js"** — Claude will check whether
  you already have it, open the download page, get the official installer, and walk you
  through it. (You'll click *Install* and enter your Mac/Windows password at one step —
  Claude can't do that part for you.)
- Or do it yourself: download the **LTS** installer from https://nodejs.org and run it.

### 2. Make your data folder

Create an empty folder named **`HealthTracker`** in your home folder:

- **Mac:** `~/HealthTracker`  (i.e. `/Users/<you>/HealthTracker`)
- **Windows:** `C:\Users\<you>\HealthTracker`

This is where all your data lives, as plain `.md` files.

> Want it somewhere else? After installing, open the plugin's `.mcp.json` and change the
> folder path on the last line, then restart Cowork. **Windows:** if `npx` doesn't launch,
> change the `.mcp.json` `command` from `"npx"` to `"cmd"` with `"/c"` then `"npx"` as the
> first args.

### 3. Install the plugin

In Cowork, type these two commands:

```
/plugin marketplace add pbrosen/brads-health-tracker
/plugin install health-tracker@brad-health
```

### 4. Approve the connector and restart

Cowork will ask to enable the plugin's **`health-files`** connector — approve it. Then
**quit and reopen Cowork** so the file-saver starts up with Node.

### 5. Connect your folder

Point Cowork at your `~/HealthTracker` folder for the session, so chat-based logging can
read and write it.

### 6. Open your dashboard

Say to Claude: **"Set up my Health Tracker dashboard."** Claude creates the dashboard
(a live page you can pin and reopen). It finds your data folder automatically.

### 7. Enter your numbers

In the dashboard, open **⚙ Calorie-burn settings** and enter your weight, height, and age
(these drive the resting-calorie math). *Optional:* connect **Strava** from Cowork's
connector store to count your runs/rides.

That's it — you're running.

---

## How to use it

**By chatting (works anywhere, even on mobile):**

- "I had two eggs and toast at 7:30 and a turkey sandwich for lunch."
- "Log 45 minutes of weight training, about 280 calories."
- "Log a bout of diarrhea this morning, Bristol 6, felt like it was the dairy."
- "What have I eaten so far today? How many calories left?"
- "What's my net calorie balance today?" *(pulls in Strava if connected)*
- "What foods might be triggering my stomach?"

**In the dashboard:** add food (leave macros blank to let Claude estimate them), exercise,
and gut episodes; see net calories, macros, VO₂ minutes, and a 7-day chart; and click
**🔎 Find food correlations** to scan which foods precede your diarrhea/loose episodes.
Logging by chat and in the dashboard write the **same files**, so they stay in sync.

---

## Where your data lives

Inside your `HealthTracker` folder:

```
HealthTracker/
├── food-log/
│   ├── 2026-06-13.md        ← one file per day (food + manual exercise)
│   └── _library.md          ← your personal food database (auto-built)
└── symptom-log.md           ← rolling log of gut episodes
```

Normal Markdown files — open, back up, or sync them however you like.

---

## Troubleshooting

- **"Couldn't find your Health Tracker folder."** Almost always means Node isn't installed,
  or the `~/HealthTracker` folder wasn't created/connected, or you didn't restart Cowork
  after installing. Re-check steps 1, 2, 4, and 5.
- **Dashboard shows but won't save.** The `health-files` connector didn't start — confirm
  Node is installed and you approved the connector, then restart Cowork.

---

## Notes

- This is a personal tracker, **not medical advice**. The food↔symptom correlation is a
  rough statistical hint to discuss with a doctor or dietitian, not a diagnosis.
- Everything stays on your computer — the dashboard reads/writes only your one
  `HealthTracker` folder.
