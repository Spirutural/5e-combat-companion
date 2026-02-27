# Changelog

## v1.2.0 — 2026-02-26

### Feats
- New **Feats** section in the right panel between Inventory and Skills
- Add feats by name; expand to view or edit a description
- Double-click the description or feat name to edit inline
- Section is independently collapsible; state persists in localStorage

### Conditions Reference
- **Conditions ▾** dropdown added to the header
- Lists all 15 SRD conditions (Blinded through Unconscious) with rules-as-written mechanical effects
- Scrollable, closes on click-outside

### Inventory
- Item descriptions now support **double-click to edit** — ✏ Edit button removed

---

## v1.1.0 — 2026-02-26

### Inventory Consolidation
- Merged **Inventory**, **Magic Items**, and **Attuned Items** into a single unified **Inventory** panel
- Items now carry a type (`normal` / `magic`) — magic items show Equipped and Attuned toggles inline
- Attuned sub-section with pip display sits below the item list; attunement slot cap still adjustable
- Full data migration from old localStorage shape — existing saves upgrade automatically

### Spellcasting
- Spell Save DC, Spell Attack, and Ability selector now **collapse with the Spell Slots panel** — no more values floating above the fold when slots are hidden

### Status Panel
- **Active Spells sub-section is now independently collapsible** — collapse state persists in localStorage

### Materials Viewer
- Replaced the standalone **Materials** button and the **Stat Blocks** tab bar with a single **Materials ▾** dropdown
- Clicking Materials always opens the floating viewer (ready for drag-and-drop)
- Dropdown lists all loaded stat blocks; clicking one activates it and brings the panel forward
- Loading a new file via **+ Add file…** opens the viewer automatically
- Viewer panel can still be closed independently via its own controls

### Help Guide
- Added **?** button to the header that opens a quick-start guide modal covering all major sections

## v1.0.0 — 2026-02-26

First full-featured release.

### HP & Status
- HP Effects section: Aid, Heroes' Feast, and Max HP Reduction — each correctly adjusts current and/or max HP per 5e rules, with editable values and ON/OFF toggles
- Active Spells and Conditions merged into a unified collapsible **Status** panel

### Stats & Skills
- Saving throws now have **3-state proficiency pips** (none / proficient / expertise)
- Full **Skills section** with auto-calculated modifiers, 3-state pips, and support for custom skills
- Level & Class label moved to the **header subtitle** (click to edit)
- **Passive Perception** moved into the Stats block
- **Speed** now expandable to show Walk / Swim / Climb / Fly

### Spellcasting
- Spell Ability, Spell Save DC, and Spell Attack moved to the top of the **Spell Slots** panel with their own pencil-edit mode
- Spellcasting section removed from the left panel

### Inventory
- **Quantity +/− buttons** on all inventory and magic items
- **Item descriptions** — collapsible per item, with read-only view and pencil edit
- **Currency tracker** (PP / GP / SP / CP) at the bottom of Inventory
- **Attunement slot cap** now has both − and + buttons

### Layout & Polish
- Right column widened to 360px default
- Stat panel can no longer be dragged off the top of the screen
- Spell DC/Attack labels now show `(auto)` instead of a cryptic icon
