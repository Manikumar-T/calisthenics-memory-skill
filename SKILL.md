---
name: calisthenics-memory-backup
description: >
  Use this skill whenever the user uploads a CalisthenicsMemory backup JSON
  file and wants to read it, update exercises, restructure programs, or
  generate a new backup ready for import. Triggers: file named
  `calisthenics_memory_backup_*.json`, or any request like "update my
  calisthenics app", "add exercise to my routine", "restructure my programs".
compatibility: claude.ai, Claude Desktop, Cowork
---
 
# CalisthenicsMemory Backup Skill
 
## App Overview
 
**CalisthenicsMemory** is a mobile tracking app. It exports/imports data as
a single JSON file. The user restores progress by importing a backup through
the app's settings.
 
---
 
## JSON Schema (version 8)
 
```json
{
  "version":          8,
  "exportDate":       "ISO-8601 string",
  "app":              "CalisthenicsMemory",
  "groups":           [ ...GroupObject ],
  "exercises":        [ ...ExerciseObject ],
  "records":          [ ...RecordObject ],
  "programs":         [ ...ProgramObject ],
  "programExercises": [ ...ProgramExerciseObject ]
}
```
 
### GroupObject
```json
{
  "id":           1,
  "name":         "Warm-Up",
  "displayOrder": 0          // optional; controls sidebar order
}
```
Groups are categories shown in the exercise library sidebar.  
Known groups: `Warm-Up`, `Hand Stands Skill`, `Upper Body`, `Core`,
`Lower Body`, `Cool Down`.
 
---
 
### ExerciseObject
```json
{
  "id":           8,
  "name":         "Regular Push Up",
  "type":         "Dynamic",      // "Dynamic" | "Isometric"
  "group":        "Upper Body",   // must match a group name exactly
  "sortOrder":    5,              // internal; keep at 5 for new entries
  "displayOrder": 1,              // optional; order within the group
  "laterality":   "Bilateral",   // "Bilateral" | "Unilateral"
  "targetSets":   3,
  "targetValue":  20             // reps (Dynamic) or seconds (Isometric)
}
```
 
**Type rules:**
- `Dynamic`   → targetValue = rep count
- `Isometric` → targetValue = hold seconds
 
**Laterality rules:**
- `Bilateral`  → single value logged per set
- `Unilateral` → separate left/right values logged per set
 
---
 
### RecordObject
```json
{
  "id":         282,
  "exerciseId": 8,
  "valueRight": 8,          // reps or seconds completed
  "valueLeft":  null,       // null for Bilateral; number for Unilateral
  "setNumber":  1,
  "date":       "2026-03-26",
  "time":       "06:19",
  "comment":    "Workout Mode"
}
```
**NEVER modify records.** They are the user's historical performance log.
Always copy them verbatim from the original backup.
 
---
 
### ProgramObject
```json
{
  "id":   1,
  "name": "Skill Day (Mon / Wed / Fri)"
}
```
 
---
 
### ProgramExerciseObject
```json
{
  "id":              1,
  "programId":       1,      // FK → programs.id
  "exerciseId":      4,      // FK → exercises.id
  "sortOrder":       0,      // 0-based; controls exercise sequence in program
  "sets":            3,
  "targetValue":     30,     // overrides exercise default for this program
  "intervalSeconds": 60      // rest timer shown after each set
}
```
 
---
 
## ID Assignment Rules
 
When adding new objects, always use `max(existing ids) + 1` per table:
 
| Table              | Find max with                                      |
|--------------------|----------------------------------------------------|
| `groups`           | `max(g["id"] for g in data["groups"])`             |
| `exercises`        | `max(e["id"] for e in data["exercises"])`          |
| `programs`         | `max(p["id"] for p in data["programs"])`           |
| `programExercises` | `max(pe["id"] for pe in data["programExercises"])` |
 
Never reuse or gap IDs — the app uses them as primary keys.
 
---
 
## Reading the Backup
 
```python
import json
 
with open('/mnt/user-data/uploads/calisthenics_memory_backup_*.json') as f:
    data = json.load(f)
 
# Quick summary
ex_map  = {e["id"]: e["name"] for e in data["exercises"]}
prog_map = {p["id"]: p["name"] for p in data["programs"]}
 
for prog in data["programs"]:
    pes = sorted(
        [pe for pe in data["programExercises"] if pe["programId"] == prog["id"]],
        key=lambda x: x["sortOrder"]
    )
    print(f"\n[{prog['name']}]")
    for pe in pes:
        print(f"  {pe['sortOrder']:2d}. {ex_map[pe['exerciseId']]:40s}"
              f"sets={pe['sets']} target={pe['targetValue']} rest={pe['intervalSeconds']}s")
```
 
---
 
## Standard Workout Structure (Mani's current plan)
 
Every training day follows this sequence in its program:
 
```
1. Warm-Up       (Jumping Jacks → Leg Swings → Leg Rotates L/R →
                  Knee Rotates → Hip Rotation → Arm Circles → Wrist Roll)
2. Handstand Skill  (Shoulder Shrugs → Chest-to-Wall Handstand →
                     Balance Drill 1-Foot → Wall Walking)
3. Day-specific strength block  (see table below)
4. Cool Down     (Child's Pose) — only on strength days
```
 
| Day       | Program Name                  | Strength Block                                          |
|-----------|-------------------------------|---------------------------------------------------------|
| Monday    | Skill Day (Mon / Wed / Fri)   | — (skill only)                                          |
| Tuesday   | Upper Body Day (Tuesday)      | Pull-ups × 10, Push-ups × 20, Dips × 15               |
| Wednesday | Skill Day (Mon / Wed / Fri)   | — (skill only)                                          |
| Thursday  | Core Day (Thursday)           | Full Plank 60s, Boat Hold 30s, Superman 30s, Leg Raises × 10 |
| Friday    | Skill Day (Mon / Wed / Fri)   | — (skill only)                                          |
| Saturday  | Lower Body Day (Saturday)     | Squats × 25, Bulgarian Split × 10/leg, Pistol × 5/leg |
| Sunday    | REST                          | —                                                       |
 
**Level:** Intermediate → **3 sets**, **60–75 s rest** between sets.
 
---
 
## Interval Seconds Reference
 
| Block         | intervalSeconds |
|---------------|-----------------|
| Warm-Up       | 25              |
| Handstand Skill (reps) | 60   |
| Handstand Skill (holds) | 75  |
| Strength Work | 75              |
| Cool Down     | 30              |
 
---
 
## Adding a New Exercise
 
```python
new_ex_id = max(e["id"] for e in data["exercises"]) + 1
 
data["exercises"].append({
    "id":          new_ex_id,
    "name":        "Pike Push-Up",
    "type":        "Dynamic",
    "group":       "Upper Body",
    "sortOrder":   5,
    "laterality":  "Bilateral",
    "targetSets":  3,
    "targetValue": 10
})
```
 
Then add it to a program:
 
```python
new_pe_id  = max(pe["id"] for pe in data["programExercises"]) + 1
target_pid = 2  # Upper Body Day
 
existing   = [pe for pe in data["programExercises"] if pe["programId"] == target_pid]
next_sort  = max(pe["sortOrder"] for pe in existing) + 1
 
data["programExercises"].append({
    "id":              new_pe_id,
    "programId":       target_pid,
    "exerciseId":      new_ex_id,
    "sortOrder":       next_sort,
    "sets":            3,
    "targetValue":     10,
    "intervalSeconds": 75
})
```
 
---
 
## Generating the Output File
 
```python
from datetime import datetime
 
output = {
    "version":          8,
    "exportDate":       datetime.now().isoformat(),
    "app":              "CalisthenicsMemory",
    "groups":           data["groups"],
    "exercises":        data["exercises"],
    "records":          data["records"],          # NEVER modify
    "programs":         data["programs"],
    "programExercises": data["programExercises"],
}
 
out_path = "/mnt/user-data/outputs/calisthenics_memory_backup_updated.json"
with open(out_path, "w") as f:
    json.dump(output, f, indent=2)
```
 
The output file is ready to import directly into the CalisthenicsMemory app
via **Settings → Import Backup**.
 
---
 
## Common Mistakes to Avoid
 
| Mistake | Correct approach |
|---------|-----------------|
| Editing `records` | Never touch — preserve verbatim |
| Reusing an existing ID | Always `max + 1` |
| `group` value doesn't match any group name | Add the group first |
| `sortOrder` gaps in a program | Renumber 0, 1, 2… sequentially |
| Setting `targetValue` in seconds for a Dynamic exercise | Use rep count for Dynamic |
| Forgetting Cool Down on strength days | Append Child's Pose as last program exercise |