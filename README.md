# 5e Combat Companion

A single-file D&D 5e combat tracker for experienced players. No install, no server, no account — just open `tracker.html` in any browser.

## Features

**Character & HP**
- HP tracking with color-reactive bar (green → yellow → red)
- Temp HP, editable current/max HP
- HP Effects: Aid, Heroes' Feast, and Max HP Reduction toggles with correct 5e math
- **Hit Dice tracker** — collapsible heart-pip display below HP bar, configurable die type (d6–d12)

**Stats**
- Ability scores with auto-calculated modifiers
- AC, Initiative, Proficiency Bonus
- Walk / Swim / Climb / Fly speeds (expandable)
- Passive Perception
- All stats inline-editable via pencil mode or click-to-edit

**Saves & Skills**
- Saving throws with 3-state proficiency pips (none / proficient / expertise)
- Full skills section with auto-calculated modifiers, 3-state pips
- Custom skills support
- Click any skill value to set a flat bonus (racial, item, etc.) — pip turns red when active

**Spellcasting**
- Spell Save DC and Spell Attack (auto-calculated, manual override available)
- Spellcasting Ability selector (INT / WIS / CHA)
- Spell slots for all 9 levels with collapsible panel — DC/Atk/pips all hide together
- Active spells tracker with Concentration handling, independently collapsible

**Status**
- Conditions tracker
- Immunities & Resistances
- Optional Senses panel (Truesight, Blindsight, Tremorsense)
- Conditions reference dropdown (header) — all 15 SRD conditions with mechanical effects

**Inventory**
- Unified item list — normal items and magic items in one panel
- Magic items have Equipped and Attuned toggles; Attuned section shows pip display
- Adjustable attunement slot cap
- Quantity +/− buttons on all items
- Click item name to expand description; double-click to rename
- Collapsible item descriptions with double-click to edit
- Magic items support an optional **charge tracker** — amber pip row, configurable max, fully manual
- Currency tracker: PP / GP / SP / CP

**Feats**
- Collapsible feats list with per-feat expandable descriptions
- Double-click feat name to rename; double-click description to edit inline

**Combat**
- Custom Attacks (name, to-hit, damage, notes)
- Dice roller with modifier and history

**Tools**
- **Materials** button — opens floating stat block viewer and dropdown to manage loaded files
- Drag-and-drop images or PDFs directly into the viewer; supports multiple files
- Export / Import character as JSON
- Formula reference dropdown (Attack Roll, Spell Attack, Spell Save DC)
- Quick-start **Help guide** (? button in header)
- All data persists in browser localStorage

## Usage

Download `tracker.html` and open it in your browser. Everything saves automatically. To load a stat block, click **Materials ▾** and either drag a file onto the viewer or use **+ Add file…**.

## Design Philosophy

Built for high-level, high-complexity play where strict level-cap assumptions and locked character sheets get in the way. Every field is editable, every section is optional, and nothing requires an external database or account.

## Class Variants

Class-specific variants (Druid with Wild Shape, etc.) are maintained on separate branches.

## Upcoming Features

**Planned**
- Prepared spells — manual spell list folded into the Spell Slots panel, organized by level
- Combat section revamp — Actions, Bonus Actions, and Reactions in labelled columns with one-click dice rolls wired to the existing roller
- Lore Book — floating notes panel with session-by-session import/export, separate from character data
- Class resource blocks — Bardic Inspiration, Channel Divinity, Action Surge, Ki Points, etc. (deferred; tied to class variant system)
