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

No terminal, no Node, no config files. The dashboard saves to your computer through a
one-click **Filesystem** extension (it bundles everything it needs).

### 1. Make your data folder

Create an empty folder named **`HealthTracker`** in your home folder:

- **Mac:** `~/HealthTracker`  (i.e. `/Users/<you>/HealthTracker`)
- **Windows:** `C:\Users\<you>\HealthTracker`

This is where all your data lives, as plain `.md` files.

### 2. Install the Filesystem extension and point it at that folder

This is what lets the dashboard read and write your files (no Node, no setup files — the
extension is self-contained).

1. In Claude/Cowork, open **Settings → Extensions** (a.k.a. Connectors).
2. Click **Browse extensions**, find the official **Filesystem** extension, and click **Install**.
3. When it asks which folder it may access, choose your **`HealthTracker`** folder.
4. Make sure it's **enabled for this chat** (the "+" → Connectors panel).

### 3. Install the plugin

In Cowork, type these two commands:

```
/plugin marketplace add pbrosen/brads-health-tracker
/plugin install health-tracker@brad-health
```

### 4. Connect your folder

Point Cowork at your `~/HealthTracker` folder for the session, so chat-based logging can
read and write it too.

### 5. Open your dashboard

Say to Claude: **"Set up my Health Tracker dashboard."** Claude creates the dashboard
(a live page you can pin and reopen), wiring it to your installed Filesystem extension and
finding your data folder automatically.

### 6. Enter your numbers

In the dashboard, open **⚙ Calorie-burn settings** and enter your weight, height, and age
(these drive the resting-calorie math).

### 7. Connect Strava  (optional)

Strava lets Claude count your runs and rides toward your burn and net calories.

1. In Cowork, open the **connector store / directory**.
2. Find **Strava** and click **Connect**.
3. You'll be sent to Strava's site — **sign in** and click **Authorize** to grant access.
4. Back in Cowork, make sure **Strava is enabled for this chat** (the "+" → Connectors panel).

Once connected, just ask Claude things like *"what's my net today?"* and it pulls your Strava
workouts in.

> **Want the *dashboard's* burn number to include Strava too?** By default Strava shows up
> when you ask Claude in chat, but the dashboard's "Burned today" only counts manual exercise
> + resting. To have your runs/rides feed the dashboard automatically, just say **"set up my
> Strava auto-sync"** — Claude creates a daily task that pulls Strava into your folder, and the
> dashboard picks it up. Optional, and you can add it anytime.

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

- **"Couldn't find your Health Tracker folder."** Almost always means the Filesystem
  extension isn't installed/enabled, or it isn't pointed at your `HealthTracker` folder, or
  the folder doesn't exist. Re-check steps 1 and 2.
- **Dashboard shows but won't save.** The Filesystem extension didn't connect — confirm it's
  installed, enabled for this chat, and granted access to the folder. If saving still fails,
  the dashboard's connector name may differ on your machine; tell Claude "the dashboard can't
  save" and it can re-wire it.

---

## Notes

- This is a personal tracker, **not medical advice**. The food↔symptom correlation is a
  rough statistical hint to discuss with a doctor or dietitian, not a diagnosis.
- Everything stays on your computer — the Filesystem extension reads/writes only the one
  `HealthTracker` folder you granted it.
