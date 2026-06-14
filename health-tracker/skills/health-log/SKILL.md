---
name: health-log
description: Log food, drinks, exercise, or digestive (gut) symptoms to Jim's daily Health Tracker files, and answer questions about today's intake / burn / macros / gut symptoms. ALWAYS trigger this skill when Jim says he ate, drank, had, finished, or logged any food or drink ("I had a turkey sandwich for lunch", "log a Diet Coke", "two eggs and toast at 7:30"), when he logs exercise ("did 30 min weight training", "lifted for 45 min", "log my run"), when he logs a gut/digestive episode ("log a bout of diarrhea", "loose stool this morning", "bloated after lunch, bristol 6"), or when he asks about today's nutrition, burn, or gut history ("what have I eaten so far", "what's my net calorie balance", "how many calories left", "what foods might be triggering my stomach"). Also trigger on past-day variants ("log yesterday's dinner") and edits ("change lunch to 600 calories", "remove the Diet Coke"). This skill is the canonical way to manipulate the tracker when the Health Tracker dashboard isn't open — chat, mobile, and scheduled tasks all go through it to keep the file format consistent.
---

# Health log

Jim runs a daily food + exercise + gut-symptom tracker. The dashboard is a Cowork
artifact ("Health Tracker"), but the underlying data lives as **plain Markdown files
in a local folder on Jim's computer**. This skill writes to and reads from those files
directly — the same file format the dashboard uses, so dashboard and chat-driven
logging stay consistent.

This is the non-Obsidian build of the system: there is **no Obsidian and no vault**.
Use the normal file tools (Read / Write / Edit) on the local folder.

## Where the files live

Everything lives under Jim's Health Tracker folder, which is the folder he connected
to Cowork (default: `~/HealthTracker`). Call it `BASE` below.

- **Daily log:** `BASE/food-log/YYYY-MM-DD.md` — one file per day. Created on first write.
- **Food library:** `BASE/food-log/_library.md` — personal database of foods Jim has
  logged before, with their macros. Check this before guessing.
- **Gut symptom log:** `BASE/symptom-log.md` — one rolling, append-only file for all
  digestive episodes (see "Gut symptoms" below).

To find `BASE`: it is the folder Jim connected to this Cowork session. If you're unsure
of the absolute path, list the connected folder's contents — it will contain `food-log/`
and/or `symptom-log.md` once set up. If nothing exists yet, treat it as a fresh setup
(see "First-time setup").

Use the file tools:

- `Read` — read one file. If it doesn't exist, that's expected: treat a missing day file
  as an empty day `{food: [], exercise: []}`, a missing symptom log as `{symptoms: []}`.
- `Write` — write/overwrite one file (creates parent folders as needed).

**Strava is NOT stored as local files in this build.** Jim's workouts live in his
connected Strava connector and are read live (see "Strava" below) — do not write Strava
activity files.

## Day file format — preserve exactly

This is non-negotiable. The dashboard parses these files; corrupting the structure
breaks the dashboard. The shape:

```
---
title: Food Log 2026-06-04
date: 2026-06-04
category: health
type: food-log
calories_in: 1450
calories_out_manual: 0
net_calories_manual: 1450
protein_g: 85
carbs_g: 160
fat_g: 55
vo2_zone_min_manual: 0
---

## Summary
- Calories in: **1450** · manual exercise out: **0**
- Protein **85g** · Carbs **160g** · Fat **55g**
- Manual VO₂max zone minutes: **0**
- _(Strava-derived burn + VO₂ min are read live and not duplicated here.)_

## Food
- 07:30 — 2 eggs and toast _(2 eggs scrambled, 1 slice whole-wheat toast)_ — 280 cal · 18P / 24C / 14F
- 12:45 — turkey sandwich and chips — 720 cal · 35P / 78C / 28F

## Manual exercise
_(none yet)_

<!--DATA
{"food":[{"id":"f...","time":"07:30","name":"2 eggs and toast","cal":280,"p":18,"c":24,"f":14,"note":"2 eggs scrambled, 1 slice whole-wheat toast","source":"ai"},{"id":"f...","time":"12:45","name":"turkey sandwich and chips","cal":720,"p":35,"c":78,"f":28,"note":"","source":"library"}],"exercise":[]}
-->
```

Frontmatter numbers, the Summary section, and the human-readable Food/Exercise sections
are all **derived from** the JSON inside the `<!--DATA ... -->` block at the bottom. The
DATA block is the source of truth. Always rebuild the whole file from the JSON; never try
to surgically edit only the human-readable parts — they'll drift out of sync.

If a day has no food and no exercise, the Food and Manual exercise sections each read
`_(none yet)_`.

## Workflow: logging a food entry

1. **Resolve the date.** Default = today. Accept "yesterday", "Monday", explicit dates.
   Convert to `YYYY-MM-DD` in Jim's local timezone.
2. **Resolve the time.** Default = current local time formatted `HH:MM` (24-hour). If Jim
   says "at 7:30 am", use that.
3. **Read the day file.** If it doesn't exist, start with `{food: [], exercise: []}`.
4. **Check the library** (see "Library handling"). If there's an exact normalized-name
   match, reuse those macros — set `source: "library"`. A library hit is *more accurate*
   than a fresh estimate because it reflects Jim's prior corrections.
5. **If no library hit, estimate the macros yourself.** Use realistic restaurant/home
   values. Respect any quantity Jim gave ("2 eggs", "large", "8 oz"); otherwise assume one
   standard serving. Round protein/carbs/fat to whole grams. Capture the portion assumption
   in the `note` field (under 60 chars). Set `source: "ai"`.
6. **Append the entry** to the food array. Sort the array by `time`. Generate an `id` like
   `f${Date.now()}${random}` — the dashboard expects ids.
7. **Rebuild the file** from the updated arrays (see "Building the day file") and write it.
8. **Update the library** with this food's macros so the next time it's a library hit.

### Food entry shape

```json
{
  "id": "f1717536000123456",
  "time": "12:45",
  "name": "turkey sandwich and chips",
  "cal": 720,
  "p": 35,
  "c": 78,
  "f": 28,
  "note": "deli turkey, white bread, side of chips",
  "source": "ai"
}
```

Valid `source` values: `manual` (Jim supplied the macros), `library` (matched a known
food), `ai` (you estimated), `edited` (corrected after the fact).

## Workflow: logging an exercise entry

The `exercise` array is for activity Strava does **not** capture — weight training, yoga,
etc. Don't log runs/rides here; those come from Strava (read live). Same date/time
resolution as food, then:

1. Get from Jim: name (required), calories burned (required — if he doesn't know, estimate
   from duration + activity type), duration in minutes (optional), VO₂max zone minutes
   (optional, defaults to 0).
2. Append to the `exercise` array, sort by `time`, generate an `id` like `e${Date.now()}...`,
   rebuild the file, write.

### Exercise entry shape

```json
{ "id": "e1717536000999111", "time": "06:00", "name": "weight training", "cal": 280, "vo2": 0, "dur": 45 }
```

If Jim says "log my run", it's almost certainly already in Strava — check Strava first
(see below) and tell him it's there rather than duplicating it as a manual entry.

## Gut symptoms (digestion tracking)

Digestive episodes are logged to a single rolling file `BASE/symptom-log.md` (NOT per-day —
day files get rebuilt on every save and would clobber anything stored in them). Trigger
on things like "log diarrhea", "loose stool this morning", "constipated today",
"bloated/gassy after lunch", optionally with a Bristol number and a suspected trigger.

### Symptom file format — preserve exactly

```
---
title: Gut symptom log
status: living
last_updated: 2026-06-13
type: symptom-log
category: health
tags: ["health", "digestion", "symptom-log"]
---

# Gut symptom log

Rolling, append-only log of digestive episodes, kept alongside the food log so foods can be
correlated with symptoms over time. Maintained by the Health Tracker dashboard and by Claude in chat.

## Episodes (1)

| Date | Time | Type | Severity | Bristol | Note |
|------|------|------|----------|---------|------|
| 2026-06-13 | 09:30 | Diarrhea | mild | 5 |  |

<!--DATA
{"symptoms":[{"id":"s1781362256473618","date":"2026-06-13","time":"09:30","type":"diarrhea","severity":"mild","bristol":5,"note":""}]}
-->
```

As with day files, the `<!--DATA ... -->` block is the source of truth; the table is
derived from it. Episodes are sorted **newest first** in both the table and rebuild.

### Symptom entry shape

```json
{ "id": "s1781362256473618", "date": "2026-06-13", "time": "09:30", "type": "diarrhea", "severity": "mild", "bristol": 5, "note": "suspected dairy" }
```

- `type`: one of `diarrhea`, `loose` (loose stool), `constipation`, `gas` (gas/bloating),
  `other`. In the table, render with these labels: Diarrhea, Loose stool, Constipation,
  Gas / bloating, Other.
- `severity`: `mild`, `moderate`, or `severe`.
- `bristol`: Bristol Stool Scale 1–7 (1–2 hard · 3–4 normal · 5 soft · 6–7 diarrhea).
  Use `0` / omit if not applicable.
- `note`: free text — suspected trigger, cramps, etc.

### Logging a symptom

1. Resolve date (default today) and time (default now).
2. Read `symptom-log.md` (empty `{symptoms: []}` if missing).
3. Append the new episode, generate an `id` like `s${Date.now()}${random}`.
4. Rebuild the whole file from the `symptoms` array (frontmatter `last_updated` = today,
   `## Episodes (N)` count, table sorted newest-first, then the single-line DATA block) and
   write it.
5. Confirm briefly.

### Food ↔ symptom correlation

When Jim asks "what foods might be triggering my stomach / gut / diarrhea":

1. Read `symptom-log.md`; keep episodes of type `diarrhea` and `loose`. Need at least ~2
   to say anything useful; if fewer, tell him to keep logging.
2. Read the day files in `BASE/food-log/` and gather every food entry with its timestamp.
3. For each food, check whether a diarrhea/loose episode followed it within a window
   (default **6–48 hours** after eating). A food eaten before episodes at a rate higher
   than the overall base rate — and eaten ≥2 times, implicated in ≥2 episodes — is a
   "suspect".
4. Report suspects as **leads to watch, not proof**. State plainly: correlation isn't
   causation, and results assume consistent logging. (The dashboard has a "Find food
   correlations" button that does exactly this — point Jim there for the interactive view.)

## Strava (read live — not stored locally)

Jim has a Strava connector. Use it for burn / net-calorie reporting; do not write Strava
files. The Strava connector's tools (e.g. `list_activities`, then per-activity detail/streams)
are available at runtime — discover the connected Strava tools and call them.

When Jim asks about burn or net for a date:

1. Sum food → intake calories + macros from the day file.
2. Pull that date's Strava activities (calories burned + any VO₂/high-HR minutes).
3. Add manual exercise calories from the day file.
4. Add Jim's prorated resting burn (BMR math below).
5. Net = intake − total burn.

If Strava isn't connected yet, report intake + manual exercise + resting burn, and note
that Strava workouts aren't included until he connects Strava.

### Jim's stats (for BMR / resting calorie math)

These live in the dashboard's **Calorie-burn settings** (weight, height, age) and are the
source of truth. If you don't know them, ask Jim once and remember them for the session.

- BMR (Mifflin–St Jeor, male): `10 × weight_kg + 6.25 × height_cm − 5 × age + 5` kcal/day.
  (For female, the constant is `− 161` instead of `+ 5`. Confirm with Jim if unsure.)
- Prorated resting burn so far today: `BMR × (minutes_elapsed_today / 1440)`.

If Jim mentions a weight change in chat, remind him to update it in the dashboard's
Settings panel so the two stay in sync (you can't write the dashboard's stored settings).

## Building the day file

After mutating the `food` and `exercise` arrays, regenerate the whole file:

1. Sum totals: `calories_in`, `calories_out_manual` (Σ exercise.cal), `net_calories_manual`
   (calories_in − calories_out_manual), `protein_g`, `carbs_g`, `fat_g`,
   `vo2_zone_min_manual` (Σ exercise.vo2). Round calories to whole numbers; macros to one decimal.
2. Build the frontmatter, `## Summary`, `## Food`, `## Manual exercise`, then the DATA block.
3. **Food list lines:** `- HH:MM — name _(note if present)_ — N cal · NP / NC / NF`. Only
   include the italic note if `note` is non-empty.
4. **Exercise list lines:** `- HH:MM — name (N min) — N cal · N VO₂ min`. Duration in parens
   only if `dur > 0`.
5. **DATA block JSON** must be on a single line — `JSON.stringify({food, exercise})` style,
   no pretty-printing. Parse your own JSON back before writing to be sure it's valid.

## Library handling

`BASE/food-log/_library.md` — same DATA-block-at-the-bottom pattern.

**Normalize names before lookup or insert:** `name.toLowerCase().trim().replace(/\s+/g, " ")`.

### Library file format

```
---
title: "Food library"
status: living
last_updated: 2026-06-04
type: food-library
category: health
tags: ["health", "nutrition", "food-log"]
---

# Food library

Each row is a normalized food name → most recent macros from edits or AI estimates.

## Entries (N)

| Name | Cal | P | C | F | Source | Updated |
|------|-----|---|---|---|--------|---------|
| 2 eggs and toast | 280 | 18 | 24 | 14 | ai | 2026-06-04 |

<!--DATA
{"library":{"2 eggs and toast":{"cal":280,"p":18,"c":24,"f":14,"note":"...","source":"ai","original_name":"2 eggs and toast","updated":"2026-06-04"}}}
-->
```

Sort entries alphabetically by normalized name when rebuilding the table. On upsert, set
`updated` to today and keep `original_name` as whatever Jim originally typed (case preserved).

## Confirming back to Jim

After a successful write, confirm briefly — he just wants to know it landed:

> Logged: **turkey sandwich and chips** at 12:45 — 720 cal · 35P / 78C / 28F (AI estimate —
> say "change my lunch to N cal" to correct). Today so far: 1,450 cal in.

For a symptom: `Logged: diarrhea (mild · Bristol 5) at 09:30 on Jun 13.`

## First-time setup

If the Health Tracker folder is empty / files don't exist yet, or Jim asks to "set up my
Health Tracker":

1. Make sure his data folder exists (default `~/HealthTracker`) and is connected to Cowork.
   Create the `food-log/` subfolder if needed.
2. Create the dashboard artifact from the bundled HTML at
   `${CLAUDE_PLUGIN_ROOT}/skills/health-log/references/dashboard.html` — read that file and
   create a Cowork artifact named "Health Tracker" with its contents. The dashboard
   auto-discovers the data folder via the bundled `health-files` connector.
3. Tell him he can now log by chatting (this skill) or in the dashboard, and that Strava
   workouts appear when he asks you about burn/net in chat.

## Things that should NOT happen

- Don't write Strava activity files — Strava is read live from his connector.
- Don't break the DATA block JSON. Parse it back before writing.
- Don't silently overwrite a day's entire log. Append; only consolidate if Jim asks.
- Don't store gut symptoms in day files — they go in the rolling `symptom-log.md`.
- Don't try to write the dashboard's stored Settings (weight/height/age) — those are in the
  dashboard's local storage; ask Jim to change them in the Settings panel.
