<%*
const name = await tp.system.prompt("Item name");
await tp.file.rename(name);
await tp.file.move(`Setting Notes/Items/${name}`);

// Level
const level = await tp.system.prompt("Item level");

// Category
const category = await tp.system.suggester(
  [
    "Weapon",
    "Armor",
    "Shield",
    "Consumable",
    "Worn Item",
    "Held Item",
    "Wand",
    "Staff",
    "Scroll",
    "Rune",
    "Alchemical Item",
    "Magical",
    "Ammunition",
    "Other"
  ],
  [
    "Weapon",
    "Armor",
    "Shield",
    "Consumable",
    "Worn Item",
    "Held Item",
    "Wand",
    "Staff",
    "Scroll",
    "Rune",
    "Alchemical Item",
    "Magical",
    "Ammunition",
    "Other"
  ],
  false,
  "Item category"
);

// Rarity
const rarity = await tp.system.suggester(
  ["Common", "Uncommon", "Rare", "Unique"],
  ["Common", "Uncommon", "Rare", "Unique"],
  false,
  "Rarity"
);

// Traits
const traitsPrompt = await tp.system.prompt("Enter traits (comma-separated, e.g. 'Magical, Fire, Invested')");
let traits = traitsPrompt
  ? traitsPrompt.split(",").map(t => t.trim()).filter(Boolean)
  : [];

// Price / Bulk
const price = await tp.system.prompt("Price (e.g. '50 gp')");
const bulk = await tp.system.prompt("Bulk (e.g. 'L', '1')");

// Usage / Hands
const usage = await tp.system.prompt("Usage (e.g. 'held in 1 hand', 'worn cloak')");
const hands = await tp.system.prompt("Hands (if applicable, otherwise '-')");

// Source
const source = await tp.system.suggester(
  ["Core Rulebook", "GMG", "AP", "LO", "SoM", "Homebrew", "Other"],
  ["Core Rulebook", "GMG", "AP", "LO", "SoM", "Homebrew", "Other"],
  false,
  "Source"
);
%><% "---" %>
item: "<% name %>"
level: <% level %>
category: "<% category %>"
rarity: "<% rarity %>"
price: "<% price %>"
bulk: "<% bulk %>"
usage: "<% usage %>"
hands: "<% hands %>"
traits: [<% traits.map(t => `"${t}"`).join(", ") %>]
source: "<% source %>"
tags: ["item", "pf2e"]
---
# <% name %>
**Level:** <% level %>  
**Category:** <% category %>  
**Rarity:** <% rarity %>  
**Price:** <% price %>  
**Bulk:** <% bulk %>  
**Usage:** <% usage %>  
**Hands:** <% hands %>  
**Traits:** <% traits.join(", ") %>  
**Source:** <% source %>

---
## Activate
Action symbol, requirements, frequency, and effect.

---
## Description
Physical description and in-world context.

---
## Mechanics
Rules text, bonuses, conditions, saving throws.

---
## Crafting
- **Requirements:**  
- **Crafting DC:**  
- **Formula:**  

---

## Variants / Heightened Versions

---

## Lore & Hooks
