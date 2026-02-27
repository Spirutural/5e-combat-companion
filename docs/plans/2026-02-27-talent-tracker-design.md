# The Talent Tracker — Design Document
**Date:** 2026-02-27
**Branch:** `talent-tracker` (off `main`)
**Base file:** `tracker.html` → `talent-tracker.html`

---

## Overview

A combat tracker tailored to The Talent, a psionic class by Matthew Colville (MCDM Productions). Built as a git branch of the 5e-combat-companion, using `tracker.html` as the base. The spellcasting section is removed and replaced with a full psionics system. All sections remain collapsible. Generic and customizable — no character is hardcoded.

---

## Section 1: Layout & Structure

Single-page scrollable, consistent with the base tracker. Changes from base:

- **Spellcasting section removed** entirely
- **Strain Tracker** added below HP bar (left column)
- **Active Powers** added below Strain Tracker
- **Effects section** replaces "Active Spells" — handles both auto-strain effects and manual effects
- **Formulas section** gains: Power Save DC and Power Attack Modifier
- Right column (stats, saves, skills, feats, lore) carries over unchanged

**Two modals added:**
- **Manage Powers** — full library picker, opened via button
- **Manifestation Resolver** — per-power, opened via Manifest button on each power card

All sections are collapsible via chevron toggle, same pattern as base tracker.

---

## Section 2: Strain Tracker

Sits directly below the HP bar. Three rows — Body, Mind, Soul.

**Each row contains:**
- Colored label (Body = `#FF4DA6` hot pink, Mind = `#4DD9FF` cyan, Soul = `#9B59FF` violet)
- 8 brain pip icons — filled pips glow in the row's color
- `+1` / `−1` buttons for manual adjustment

**Above the three rows:**
- Total Strain / Strain Maximum display (e.g. `7 / 16`)
- Pulses red when total approaches maximum (within 2)
- Strain Maximum and Manifestation Die are editable in the character header

**Pip logic:**
- 8 pips per row × 3 rows = 24 max, matching the level 20 strain maximum exactly
- Individual type can go 0–8; total across all three is compared to Strain Maximum
- Exceeding Strain Maximum = death (UI shows critical warning, does not auto-kill)

**Manifestation Die** displayed in header alongside level: d4 (levels 1–4), d6 (5–12), d8 (13–20). Editable manually.

---

## Section 3: Powers System

### Active Powers List (main page)
Compact cards shown below the Strain Tracker. Each card displays:
- Power name
- Order badge (e.g. "3rd")
- School label (e.g. "Resopathy")
- Manifestation time (Action / Bonus Action / Reaction)
- Concentration indicator (if applicable)
- **Manifest** button → opens Manifestation Resolver
- Toggle to mark as currently concentrating (feeds into score calculation)

### Manage Powers Modal
Opened via a "Manage Powers" button. Contains:
- Full library of all 103 powers organized by order (1st → 6th)
- Within each order, grouped by school
- Checkbox per power — checked powers appear in Active Powers list
- Search/filter bar by name or school
- School abbreviations: Chr / Mtm / Pyr / Res / Tlk / Tlp

### Powers Data
Hardcoded JS constant (not in `state`) — does not persist to localStorage. Fields per power:
```js
{
  id: "caress-of-fire",
  name: "Caress of Fire",
  order: 1,
  school: "Pyrokinesis",           // display name
  schoolKey: "pyr",                // abbreviation
  manifestTime: "action",          // action | bonus | reaction | special
  concentration: true,
  description: "..."
}
```

**Full powers roster (103 powers):**

#### Chronopathy (15)
| Name | Order |
|---|---|
| Glimpse | 1st |
| Time Thief | 1st |
| Again | 2nd |
| Intuition | 2nd |
| Precognition | 2nd |
| Read Object | 2nd |
| Forget | 3rd |
| Ravages of Time | 3rd |
| Restore the Past | 3rd |
| Memory Gap | 4th |
| Stasis Field | 4th |
| Reveal the Path | 5th |
| Witness Demise | 5th |
| Ally of Time | 6th |
| Rejuvenate | 6th |

#### Metamorphosis (21)
| Name | Order |
|---|---|
| Minor Acceleration | 1st |
| Sharpened Senses | 1st |
| Adapt | 2nd |
| Disappear | 2nd |
| Flay | 2nd |
| Fluid Motion | 2nd |
| Fortify | 2nd |
| Penetrating Sight | 2nd |
| Sixth Sense | 2nd |
| Amplify | 3rd |
| Beam Gaze | 3rd |
| Cure Ailment | 3rd |
| Embrace the Deep | 3rd |
| Iron | 3rd |
| Psionic Resilience | 4th |
| Shadow Form | 4th |
| Bones of Glass | 5th |
| The Real | 5th |
| Brain Overload | 6th |
| Paragon | 6th |
| Steel | 6th |

#### Pyrokinesis (15)
| Name | Order |
|---|---|
| Caress of Fire | 1st |
| Flame's Master | 1st |
| Incinerate | 1st |
| Ignite | 2nd |
| Kindling | 2nd |
| Scorch | 2nd |
| Imbue Flame | 3rd |
| Overheat | 3rd |
| Smoke Screen | 3rd |
| Fire Form | 4th |
| Melt | 4th |
| Detonate | 5th |
| Heat Shell | 5th |
| Crucible | 6th |
| Heat Transfer | 6th |

#### Resopathy (17)
| Name | Order |
|---|---|
| Apparition | 1st |
| Illuminator | 1st |
| Psionic Bolt | 1st |
| Rewrite | 1st |
| Jaunt | 2nd |
| Mindscape | 2nd |
| Weight | 2nd |
| Elsewhere | 3rd |
| Extinguish | 3rd |
| Guise | 3rd |
| Icon of Fear | 3rd |
| Momentary Lapse of Reason | 4th |
| Reminisce | 4th |
| Restructure | 5th |
| Target of Hate | 5th |
| Fold Space | 6th |
| Reflection | 6th |

#### Telekinesis (15)
| Name | Order |
|---|---|
| Concussive Slam | 1st |
| Invisible Force | 1st |
| Psionic Shift | 1st |
| Choke | 2nd |
| Kinetic Crush | 2nd |
| Repel | 2nd |
| Capture Energy | 3rd |
| Pulse | 3rd |
| Telekinetic Burst | 3rd |
| Fulcrum | 4th |
| Gravitational Collapse | 4th |
| Chariot of Thought | 5th |
| Force Orbs | 5th |
| Concussive Wave | 6th |
| Mass Choke | 6th |

#### Telepathy (20)
| Name | Order |
|---|---|
| Influence | 1st |
| Psychic Stab | 1st |
| Shared Thoughts | 1st |
| Awe | 2nd |
| Believe | 2nd |
| Bolster Ego | 2nd |
| Make Friends | 2nd |
| Read Thoughts | 2nd |
| Share Pain | 2nd |
| Veritas | 2nd |
| Clarity | 3rd |
| Dagger of the Mind | 3rd |
| Distant Voice | 3rd |
| Aura Projection | 4th |
| Harlequin | 4th |
| Broadcast | 5th |
| Fracture | 5th |
| Psychic Projection | 5th |
| Mindwipe | 6th |
| Souls Intertwined | 6th |

> **Note:** Full descriptions and manifestation times will be extracted from PDF during implementation phase.

---

## Section 4: Manifestation Resolver (Modal)

Triggered by the **Manifest** button on any active power card. Walks through the test step by step — player must confirm before strain is applied.

**Step 1 — Power Info**
- Name, order, school badge displayed
- Currently concentrating powers listed (for score calculation)

**Step 2 — Manifestation Score**
- Calculated: base order + 1 per extra power being concentrated on
- Example: "Order 3 + Apparition concentration = Score 4"
- Shown clearly before rolling

**Step 3 — Roll**
- "Roll for me" button → tracker generates result
- Manual input → player enters physical die result
- Die type matches current manifestation die (d4/d6/d8)

**Step 4 — Outcome**
Three possible results displayed with color coding:
- 🟢 Roll > Score: "Success — no strain"
- 🟡 Roll = Score: "Success — +1 strain"
- 🔴 Roll < Score: "Success — +[order] strain"

If strain would exceed maximum: special warning — player chooses to manifest and die, or cancel for 0 HP.

**Step 5 — Strain Allocation** (if strain gained)
- Player taps +Body / +Mind / +Soul to distribute
- Running total shown, must match amount gained before confirming
- Confirm button applies strain and closes modal

**1st-order powers** skip steps 2–5 entirely — no manifestation test, Manifest button just marks the power as used/active.

---

## Section 5: Effects Section (Auto-Strain Conditions)

Replaces "Active Spells." Two layers of effects:

### Auto-Strain Effects
Automatically appear/disappear as strain crosses thresholds. Cannot be manually dismissed.

| Type | Threshold | Effect | Color |
|---|---|---|---|
| Body | 1 | Disadv. on STR/DEX checks | `#FF4DA6` |
| Body | 3 | Speed halved | `#FF4DA6` |
| Body | 5 | Disadv. on STR/DEX saves | `#FF4DA6` |
| Body | 7 | HP max halved | `#FF4DA6` |
| Mind | 1 | Can't Dash/Disengage/Dodge | `#4DD9FF` |
| Mind | 3 | No skill proficiency | `#4DD9FF` |
| Mind | 5 | −5 penalty to AC | `#4DD9FF` |
| Mind | 7 | No save proficiency | `#4DD9FF` |
| Soul | 1 | Disadv. on WIS/CHA checks | `#9B59FF` |
| Soul | 3 | Disadv. on death saves | `#9B59FF` |
| Soul | 5 | Disadv. on WIS/CHA saves | `#9B59FF` |
| Soul | 7 | Half healing (supernatural) | `#9B59FF` |

Each effect appears as a colored badge with the type label. Cumulative — all thresholds at or below current strain are active simultaneously.

### Manual Effects
Free-text entries, dismissable, neutral color. Work identically to the base tracker's Active Spells entries.

---

## Section 6: State & Data Model

### New State Fields
```js
// In DEFAULTS (with migration guards after ~line 1014 in tracker.html)
strain:       { body: 0, mind: 0, soul: 0 },
strainMax:    16,
manifestDie:  "d6",      // "d4" | "d6" | "d8"
knownPowers:  [],        // array of power id strings
concentration: [],       // power ids currently being concentrated on
```

### Not in State
The full powers library (`POWERS_DATA`) is a hardcoded JS constant defined once at the top of the file. It never touches localStorage — smaller saves, easier to update.

### Formulas Section Additions
- **Power Save DC** = 8 + proficiency bonus + INT modifier
- **Power Attack Modifier** = proficiency bonus + INT modifier
- Both auto-calculated from existing stat fields

---

## Implementation Notes

- Branch name: `talent-tracker`
- Output file: `talent-tracker.html` (copy of `tracker.html`, modified in place)
- Power descriptions to be extracted from PDF during implementation (pages 34–53)
- Colors to reserve in main tracker: `#FF4DA6`, `#4DD9FF`, `#9B59FF` (do not use for standard conditions)
- Brain pip emoji/icon: 🧠 or custom SVG — decide during implementation
- Optional strain effects table (d8 cosmetic roll) — lower priority, add if time permits
