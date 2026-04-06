<%*
const name = await tp.system.prompt("Enter creature name");

// Rename file
await tp.file.rename(name);

// Move into Infoblock folder (optional but recommended)
await tp.file.move(`Setting Notes/Beastiary/Infoblock/${name}`);

// Dropdown-style suggester for creature type
const ctype = await tp.system.suggester(
  ["Aberration", "Animal", "Beast", "Construct", "Dragon", "Elemental", "Fey", "Fiend", "Humanoid", "Undead", "Other"],
  ["Aberration", "Animal", "Beast", "Construct", "Dragon", "Elemental", "Fey", "Fiend", "Humanoid", "Undead", "Other"],
  false,
  "Select creature type"
);

// Dropdown-style suggester for rarity
const rarity = await tp.system.suggester(
  ["Common", "Uncommon", "Rare", "Unique"],
  ["Common", "Uncommon", "Rare", "Unique"],
  false,
  "Select rarity"
);

// Dropdown-style suggester for alignment
const alignment = await tp.system.suggester(
  [
    "Lawful Good", "Neutral Good", "Chaotic Good",
    "Lawful Neutral", "Neutral", "Chaotic Neutral",
    "Lawful Evil", "Neutral Evil", "Chaotic Evil"
  ],
  [
    "Lawful Good", "Neutral Good", "Chaotic Good",
    "Lawful Neutral", "Neutral", "Chaotic Neutral",
    "Lawful Evil", "Neutral Evil", "Chaotic Evil"
  ],
  false,
  "Select alignment"
);

// Dropdown for source options
const source = await tp.system.suggester(
  ["Bestiary 1", "Bestiary 2", "Bestiary 3", "Homebrew", "Other"],
  ["Bestiary 1", "Bestiary 2", "Bestiary 3", "Homebrew", "Other"],
  false,
  "Select source"
);
%><% "---" %>
creature-name: "<% name %>"
type: "<% ctype %>"
rarity: "<% rarity %>"
alignment: "<% alignment %>"
source: "<% source %>"
tags: ["creature", "pathfinder2e", "infoblock"]
---
# <% name %>
**Creature Type:** <% ctype %>  
**Rarity:** <% rarity %>  
**Alignment:** <% alignment %>  
**Source:** <% source %>  

---
## Overview
General concept, species-level description, role in the setting, etc.

---
## Appearance
What it looks like across its common forms/variants.

---
## Behavior & Ecology
Habits, habitat, diet, life cycle, social structure.

---
## Lore & Hooks
Legends, rumors, plot hooks, factions that use or hunt them.

---
## Variants / Statblocks

```dataview
LIST
FROM "Setting Notes/Beastiary/Statblock"
WHERE contains(string(infoblock), this.file.name)
SORT number(level) ASC
```

