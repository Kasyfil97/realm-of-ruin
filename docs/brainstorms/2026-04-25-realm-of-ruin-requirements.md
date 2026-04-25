---
date: 2026-04-25
topic: realm-of-ruin-game
---

# Realm of Ruin: Kingdom Management Game

## Problem Frame

Players want a replayable, narrative-driven kingdom management game that runs entirely inside Claude Code — no API keys, no external dependencies. Claude acts as the game master: generating events, narrating consequences, and tracking state. The player is a minor lord who inherits a broken kingdom and must survive 100 days to either die in obscurity or rise to become emperor. Every playthrough is different because Claude improvises within structured templates.

## Game Flow

```
┌─────────────────────────────────────────────────────┐
│                    NEW GAME                         │
│  Generate kingdom name, lord name, starting state   │
│  Roll initial conditions (slightly randomized)      │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │     DAILY TURN LOOP     │◄──────────────┐
          │                         │               │
          │  1. Display status panel │               │
          │  2. Draw event from pool │               │
          │  3. Present dilemma      │               │
          │  4. Player chooses       │               │
          │  5. Resolve consequences │               │
          │  6. Update resources     │               │
          │  7. Check win/loss       │               │
          │  8. Save state to JSON   │               │
          └─────┬──────┬──────┬─────┘               │
                │      │      │                     │
         ┌──────▼┐  ┌──▼───┐  └─────────────────────┘
         │ LOSS  │  │ WIN  │     (next day)
         │       │  │      │
         │ Any   │  │ Claim│
         │ stat  │  │ the  │
         │ = 0   │  │throne│
         └───────┘  └──────┘
```

## Requirements

**Core Game Loop**

- R1. Each turn represents 1 day in-game. A full campaign runs ~100 days (variable based on act pacing).
- R2. Every day draws 1 event from a weighted pool. Events are the heart of the game — never just good or bad, always a dilemma with trade-offs.
- R3. After the player chooses, Claude resolves consequences narratively and updates resources. Resource changes follow ranges defined in event templates but Claude adjusts within bounds based on context.
- R4. Game state is saved to a JSON file after every turn. Players can quit and resume anytime.

**Resources (6 Stats)**

- R5. The player manages 6 resources: Gold, Food, Military, Loyalty, Population, Reputation.
- R6. If any resource reaches zero, the game ends with a thematic death: Gold 0 = bankruptcy/seizure, Food 0 = famine revolt, Military 0 = overrun, Loyalty 0 = coup, Population 0 = abandoned, Reputation 0 = exiled.
- R7. Resources have soft warning thresholds. When a resource drops below 20% of its starting value, Claude weaves urgency into the narrative (e.g., Food low = "Your granaries echo with emptiness"; Loyalty low = "Guards avoid your gaze"). No UI warning banners — urgency is conveyed through tone and narrative only.
- R8. Starting resources are slightly randomized each playthrough to create different opening pressures.

**Three-Act Structure**

- R9. **Act 1 — The Inheritance (target: Days 1-30):** Player inherits a struggling keep. Threats are local: bandits, bad harvests, unhappy nobles. Teaches basics through consequences, not tutorials.
- R10. **Act 2 — The Rivals (target: Days 31-65):** Neighboring lords notice your growth. Diplomacy, espionage, and war become options. Alliances, marriages, and conquest are available. Internal politics heat up — advisors have agendas.
- R11. **Act 3 — The Throne (target: Days 66-100):** A power vacuum opens in the capital. Victory requires accumulating enough influence (military conquest, political alliances, popular support) to claim the throne. Three rival lords and a corrupt council are obstacles, not sequential bosses. Player chooses which to confront and how.
- R12. Act transitions use soft windows with triggers. Day ranges are targets, not hard boundaries. A trigger event fires within a transition window (e.g., Act 2 trigger fires between day 25-35). If no trigger has fired by the end of the window, one is forced. This keeps pacing flexible but bounded.

**Rival Lords**

- R36. Three named rival lords exist in the game world, each defined in JSON with: name, faction, military strength, diplomatic stance, personality, and event triggers.
- R37. Rival lords are introduced progressively: mentioned in rumors during Act 1, directly encountered in Act 2, and become primary obstacles in Act 3.
- R38. Each rival lord can be dealt with through multiple paths: military defeat, diplomatic alliance, assassination/espionage, or marriage pact. The approach affects game flags and resource costs.

**Event System**

- R13. Events are defined as JSON templates with: event ID, title, act tags, weight, preconditions (resource thresholds, act, flags), option templates with resource impact ranges, and narrative hooks/tags for Claude to improvise from.
- R14. Claude uses guided improvisation: JSON defines event structure and resource impact ranges (e.g., "gold_change": [-80, -50]). Claude picks a value within that range based on game state and prior decisions. Claude may not apply effects outside the defined range or invent new resource impacts not in the template.
- R15. Events can set flags in game state (e.g., "allied_with_lord_voss", "spymaster_is_traitor") that unlock or block future events. This creates branching storylines across playthroughs.
- R16. Some events are chained — a choice on day 5 can trigger a consequence event on day 12. Chains are tracked explicitly in game state via an `active_chains` array (not inferred from event history). Each chain entry records the triggering event, day scheduled, and consequence event ID.

**Combat — Risk-Commit System**

- R17. Before battle, the player commits resources: Gold for mercenaries, Food for siege supplies, Loyalty for morale rallying. More commitment = better odds but higher cost if defeated.
- R18. Battle outcomes are influenced by committed resources + Military stat + narrative context. Claude resolves the battle and applies consequences (resource gains/losses, territory, reputation shifts).
- R19. Retreat is always an option but carries its own costs (Reputation loss, Military desertion, emboldened enemies).

**Advisors — Recruitable Specialists**

- R20. Player starts with 1 advisor. Additional advisors (up to 4 total) are recruited through advisor-type events tagged in the event pool. Maximum 1 per specialty.
- R21. Each advisor has a specialty (military, trade, espionage, diplomacy) and a personality trait that colors their advice. Advisor templates are defined in JSON with name, specialty, personality, and bias tendencies.
- R22. Advisors can die, betray, or leave based on events and player decisions. Losing an advisor is a real setback — their specialty advice disappears from future events.
- R23. During events, relevant advisors offer their perspective as part of the narrative (e.g., the military advisor warns against diplomacy). Their advice is not always correct — personality bias can lead to bad counsel.

**TUI Display — Hybrid Panel**

- R24. The display uses a box-drawing hybrid panel layout: resource dashboard (top), narrative event (middle), choices (bottom).
- R25. The panel shows: game title, current day/100, current act, kingdom name, lord name, all 6 resources with values, event title and narrative, and numbered choice options.
- R26. After resolving a choice, show a brief outcome narrative with resource change indicators (e.g., Gold -80, Military +15) before advancing to the next day.

**State & Persistence**

- R27. All game state is stored in a single JSON file per save: resources, day number, act, flags, advisor roster, event history (last 10 events for context), and active chains.
- R28. Save files are stored in a `saves/` directory within the project. File naming: `save-{timestamp}.json`.
- R29. The game supports multiple save files. `/realm-of-ruin:load-game` lists available saves and lets the player choose.

**Skill/Plugin Integration**

- R30. The game is packaged as a Claude Code skill/plugin with two entry points: `/realm-of-ruin:new-game` (start fresh) and `/realm-of-ruin:load-game` (resume from save).
- R31. `/realm-of-ruin:new-game` generates a randomized starting state (kingdom name, lord name, slightly varied starting resources) and begins at Day 1.
- R32. The skill prompt contains the full game engine rules: how to read event templates, resolve choices, update state, render the TUI panel, and save state. Claude follows these rules as the game master.

**Victory Conditions**

- R33. Victory requires accumulating enough influence across three paths: Military Dominance, Political Alliance, and Popular Support. The player doesn't need all three — a strong enough position in one or two paths can suffice.
- R34. Influence is tracked through game flags and resource thresholds, not a separate influence stat. For example: Military > 80 + defeated 2 rival lords = Military Dominance path viable.
- R35. The final throne claim is an event that fires when influence conditions are met. It's the last dilemma — even at the end, there's a meaningful choice about what kind of emperor you become.

## Success Criteria

- A complete game can be played from Day 1 to victory or defeat in a single Claude Code session or across multiple sessions via save/load.
- Every playthrough feels meaningfully different due to randomized starting conditions, weighted event pools, and Claude's guided improvisation.
- The TUI panel renders cleanly in Claude Code's terminal output.
- Game state persists reliably across sessions via JSON save files.
- No external API calls or dependencies — runs entirely on Claude Code Pro subscription.

## Scope Boundaries

- No graphical UI — TUI box-drawing only, rendered as Claude's text output.
- No multiplayer or online features.
- No procedurally generated maps — the world is implied through narrative, not visualized spatially.
- No tutorial system — the player learns through consequences.
- No undo/rollback — decisions are permanent within a playthrough.
- No external dependencies or npm packages — pure Claude Code skill with JSON data.

## Key Decisions

- **6 resources (Standard):** Gold, Food, Military, Loyalty, Population, Reputation. Enough for real dilemmas without overwhelming the TUI.
- **Risk-commit combat:** Players gamble resources before battle. Ties warfare directly into resource management tension.
- **Recruitable specialist advisors:** Start with 1, find more through events. Each has specialty + personality. Creates unique playthroughs.
- **Hybrid panel TUI:** Box-drawing layout with status/narrative/choices sections. Balances information density with readability.
- **Any resource at zero = death:** Clean, brutal, fits the no-hand-holding philosophy. Every resource always matters.
- **Faction influence victory race:** Victory is about accumulating influence, not defeating sequential bosses. More emergent and strategic.
- **Soft windows with triggers for act transitions:** Day ranges are targets. Trigger events fire within a window; forced if overdue. Flexible but bounded pacing.
- **Skill + JSON data files:** Event templates, advisor data, and rival lord definitions live in JSON. Moddable and separates content from engine.
- **Guided improvisation:** JSON defines structure and ranges. Claude picks values within bounds. Cannot invent effects outside templates.

## Dependencies / Assumptions

- Assumes Claude Code skill system supports the `/realm-of-ruin:new-game` and `/realm-of-ruin:load-game` invocation pattern.
- Assumes Claude can reliably read/write JSON files via the skill's tool access.
- Assumes Claude Code terminal supports Unicode box-drawing characters for TUI rendering.
- Game balance will need iterative tuning of event weights, resource impact ranges, and starting values after initial implementation.

## Outstanding Questions

### Resolve Before Planning

(none — all product decisions resolved)

### Deferred to Planning

- [Affects R13][Needs research] How many events should the initial event pool contain per act? Need to balance variety against development effort. Recommend starting with ~15 per act (45 total) as a minimum viable pool.
- [Affects R8][Technical] What should the base starting resource values and randomization ranges be? Needs playtesting. Suggested starting point: Gold 500, Food 600, Military 40, Loyalty 60, Population 1000, Reputation 30, with +/-15% variance.
- [Affects R32][Technical] What is the optimal skill prompt size and structure to fit the full game engine rules while leaving room for Claude's context window to hold game state?
- [Affects R36][Technical] What are the three rival lords' names, factions, and specific attributes? Should be designed alongside Act 2-3 event pools.
- [Affects R33-34][Needs research] What specific flag combinations and resource thresholds define each victory path? Needs to be designed alongside the Act 3 event pool.
- [Affects R13][Technical] What is the event template JSON schema? Define required and optional fields for events, options, and impacts.

## Next Steps

-> `/ce:plan` for structured implementation planning
