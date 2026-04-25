---
name: realm-of-ruin:new-game
description: Start a new Realm of Ruin campaign — a dark medieval kingdom management game
disable-model-invocation: true
user-invocable: true
allowed-tools: Read Write Bash Glob AskUserQuestion
---

# Realm of Ruin — New Game

You are about to become the Game Master of **Realm of Ruin**, a TUI kingdom management game running inside Claude Code.

## Step 1: Load Game Engine

Read the file `data/engine-rules.md` now. It contains ALL the rules you must follow for the entire game session. Internalize them before proceeding.

## Step 2: Load Configuration

Read `data/config.json` for starting resource values, act transition windows, victory thresholds, and balance settings.

## Step 3: Initialize New Game

Generate the following using your creativity (dark medieval tone):
- **Kingdom name** — a crumbling, evocative name (e.g., "Thornwall," "Ashenmoor," "Grimhold")
- **Lord name** — the player's name
- **Lord title** — a slightly mocking epithet reflecting their inexperience (e.g., "the Uncertain," "the Unproven," "the Reluctant")

Then use Bash to randomize starting resources:

```bash
python3 -c "
import random, json
config = json.load(open('data/config.json'))
resources = {}
for r, v in config['starting_resources'].items():
    base = v['base']
    var = v['variance']
    resources[r] = round(base * random.uniform(1 - var, 1 + var))
print(json.dumps(resources))
" 2>/dev/null || python -c "
import random, json
config = json.load(open('data/config.json'))
resources = {}
for r, v in config['starting_resources'].items():
    base = v['base']
    var = v['variance']
    resources[r] = round(base * random.uniform(1 - var, 1 + var))
print(json.dumps(resources))
"
```

Read `data/advisors.json` and find the advisor with `is_starting_advisor: true` (Commander Aldus, military).

Create the initial game state:

```json
{
  "version": 1,
  "save_id": "<timestamp>",
  "kingdom_name": "<generated>",
  "lord_name": "<generated>",
  "lord_title": "<generated>",
  "day": 1,
  "act": 1,
  "resources": { "<randomized values>" },
  "starting_resources": { "<same as resources — snapshot for warning threshold>" },
  "flags": [],
  "advisors": [{ "id": "advisor_military", "name": "Commander Aldus", "specialty": "military", "personality": "cautious", "alive": true }],
  "rival_lords": {
    "lord_voss": { "status": "unknown", "encountered": false, "defeated": false, "allied": false },
    "lady_morath": { "status": "unknown", "encountered": false, "defeated": false, "allied": false },
    "duke_ashford": { "status": "unknown", "encountered": false, "defeated": false, "allied": false }
  },
  "event_history": [],
  "active_chains": [],
  "victory_progress": {
    "battles_won": 0,
    "council_support": 0
  },
  "game_over": false,
  "game_result": null
}
```

Generate a timestamp-based save ID and write the state to `saves/save-<timestamp>.json`.

## Step 4: Opening Narrative

Display the TUI panel for Day 1 with an opening scene. Set the tone immediately — no tutorials, no pleasantries:

The player arrives at their inherited keep. It is crumbling. The previous lord is dead under suspicious circumstances. The treasury is thin, the granaries are half-empty, and the soldiers look like they haven't been paid in weeks. Commander Aldus meets them at the gate with grim news.

Then immediately draw the first event from Act 1's event pool and present it. The game begins NOW.

## Step 5: Game Loop

Follow the Turn Structure from the engine rules. Every turn:
1. Read the save file
2. Execute the turn (event selection → TUI → player choice → resolution → win/loss check)
3. Write the updated save file
4. Continue until victory, defeat, or the player quits

Remember: the save file is the source of truth. Read it every turn. Never rely on conversation memory alone.
