# 🤸 CalisthenicsMemory — Claude Skill

A Claude skill for reading, editing, and regenerating
[CalisthenicsMemory](https://f-droid.org/packages/io.github.gonbei774.calisthenicsmemory/)
backup files. Upload your backup JSON, ask Claude to update your routine,
and get a ready-to-import file back — no manual JSON editing required.

---
## Official Distribution

This app is officially distributed only through F-Droid, IzzyOnDroid, and Codeberg Releases.
We cannot guarantee the safety of APKs downloaded from any other source.

<p align="center">
  <a href="https://apt.izzysoft.de/fdroid/index/apk/io.github.gonbei774.calisthenicsmemory"><img src="https://gitlab.com/IzzyOnDroid/repo/-/raw/master/assets/IzzyOnDroidButtonGreyBorder_nofont.png" height="80" alt="Get it on IzzyOnDroid"></a>
</p>

<p align="center">
  <a href="https://f-droid.org/packages/io.github.gonbei774.calisthenicsmemory/"><img src="https://fdroid.org/badge/get-it-on.png" height="119" alt="Get it on F-Droid"></a>
</p>

<p align="center">
  <a href="https://codeberg.org/Gonbei774/CalisthenicsMemory/releases"><img src="https://get-it-on.codeberg.org/get-it-on-white-on-black.png" height="80" alt="Get it on Codeberg"></a>
</p>
---


## What This Skill Does

- Reads and parses `calisthenics_memory_backup_*.json` exports
- Adds or updates exercises, programs, and targets
- Rebuilds day-based workout programs (Upper Body / Core / Lower Body / Skill days)
- Preserves all historical performance records untouched
- Outputs a clean JSON file ready to import back into the app

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill itself — Claude reads this before working on your backup |
| `README.md` | This file |

---

## How to Use

### 1. Export your backup
In the CalisthenicsMemory app → **Settings → Export Backup** → save the JSON file.

### 2. Start a Claude conversation
Upload both files to Claude:
- Your backup: `calisthenics_memory_backup_YYYY-MM-DD_HH-MM-SS.json`
- The skill: `SKILL.md`

Then say something like:
> "Use this skill. Add Pike Push-Up to my Upper Body Day program."

or

> "Use this skill. Increase my Pull-up target to 12 reps."

### 3. Import the result
Claude outputs an updated backup JSON. In the app → **Settings → Import Backup** → select the file.

---

## Supported Operations

| Request | Example |
|---------|---------|
| Add a new exercise | "Add Archer Push-Up to Upper Body Day" |
| Update a target | "Change Pistol Squat target to 8 reps per leg" |
| Add a new program | "Create a Mobility Day for Sunday" |
| Restructure a program | "Move Leg Raises before Plank in Core Day" |
| Change sets/rest | "Set all strength work to 4 sets with 90s rest" |
| Full rebuild | "Rebuild all programs from this weekly plan: …" |

---

## Backup JSON Schema (v8)

```
{
  version, exportDate, app,
  groups           → exercise library categories
  exercises        → exercise definitions (name, type, targets)
  records          → historical performance log  ← NEVER modified
  programs         → named workout programs
  programExercises → ordered exercise list per program
}
```

Full schema documentation with field rules, type constraints, and
code snippets is in [`SKILL.md`](./SKILL.md).

---

## My Current Workout Plan

> Intermediate level — 3 sets, 60–75 s rest

| Day | Program | Focus |
|-----|---------|-------|
| Monday | Skill Day | Handstand skill only |
| Tuesday | Upper Body Day | Pull-ups / Push-ups / Dips |
| Wednesday | Skill Day | Handstand skill only |
| Thursday | Core Day | Plank / Boat Hold / Superman / Leg Raises |
| Friday | Skill Day | Handstand skill only |
| Saturday | Lower Body Day | Squats / Bulgarian Split / Pistol Squats |
| Sunday | REST | — |

Every training day starts with a **Warm-Up** block and a **Handstand Skill**
block before the day-specific strength work.

---

## Key Rules (for Claude)

- **Never edit `records`** — they are the historical performance log
- **IDs** — always `max(existing) + 1`, never reuse or skip
- **`group` field** on exercises must exactly match a group `name`
- **`type`** determines `targetValue` units: `Dynamic` = reps, `Isometric` = seconds
- **`sortOrder`** in `programExercises` is 0-based and must be sequential
- **Cool Down** (Child's Pose) appended last on all strength days

---

## Interval Seconds Reference

| Block | Rest |
|-------|------|
| Warm-Up | 25 s |
| Handstand Skill (rep exercises) | 60 s |
| Handstand Skill (hold exercises) | 75 s |
| Strength Work | 75 s |
| Cool Down | 30 s |

---

## Requirements

- Any coding agent support skills
- CalisthenicsMemory app (Android) — schema version 8
