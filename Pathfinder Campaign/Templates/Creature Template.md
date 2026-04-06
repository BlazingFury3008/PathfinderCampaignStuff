<%*
// Build list of Infoblock files for linking
const infoblockFolder = app.vault.getAbstractFileByPath("Setting Notes/Beastiary/Infoblock");
let infoblockNames = [];

if (infoblockFolder && infoblockFolder.children) {
  infoblockNames = infoblockFolder.children
    .filter(f => f.extension === "md")
    .map(f => f.basename);
}

// Pick base Infoblock (e.g. 'Creature A')
const baseInfoblock = await tp.system.suggester(
  infoblockNames,
  infoblockNames,
  false,
  "Select base Infoblock (the species / main entry)"
);

const infoblockLink = `[[${baseInfoblock}]]`;

// Ask for variant name (subtitle)
let name = await tp.system.prompt("Enter statblock subtitle (e.g. 'Variant 1')");

// Build final filename
let finalName;

if (!name || name.trim() === "") {
  finalName = baseInfoblock;
} else {
  finalName = `${baseInfoblock} - ${name.trim()}`;
}

// Rename file
await tp.file.rename(finalName);

// Move into Statblock folder
await tp.file.move(`Setting Notes/Beastiary/Statblock/${finalName}`);

// Level prompt (free text)
const level = await tp.system.prompt("Enter creature level");

// --- Pull metadata from the Infoblock frontmatter ---
const infoblockPath = `Setting Notes/Beastiary/Infoblock/${baseInfoblock}.md`;
const infoblockFile = app.vault.getAbstractFileByPath(infoblockPath);

let ctype = "";
let rarity = "";
let alignment = "";
let source = "";

if (infoblockFile) {
  const cache = app.metadataCache.getFileCache(infoblockFile);
  if (cache && cache.frontmatter) {
    ctype = cache.frontmatter.type ?? "";
    rarity = cache.frontmatter.rarity ?? "";
    alignment = cache.frontmatter.alignment ?? "";
    source = cache.frontmatter.source ?? "";
  }
}

// For display / frontmatter: use the full statblock name as title,
// and keep subtype/variant name separately if you want it
const statblockTitle = finalName;
const subtitle = name && name.trim() !== "" ? name.trim() : "";
%><% "---" %>
creature-name: "<% baseInfoblock %>"
statblock-name: "<% statblockTitle %>"
subtitle: "<% subtitle %>"
level: "<% level %>"
type: "<% ctype %>"
rarity: "<% rarity %>"
alignment: "<% alignment %>"
source: "<% source %>"
infoblock: <% infoblockLink %>
tags: ["creature", "pathfinder2e", "statblock"]
---
# <% statblockTitle %>
**Level:** <% level %>  
**Creature Type:** <% ctype %>  
**Rarity:** <% rarity %>  
**Alignment:** <% alignment %>  
**Source:** <% source %>  
**Infoblock:** <% infoblockLink %>  

---
## Description
Short variant-specific description (how this differs from the base Infoblock).

---
## Statistics
- **Perception:**  
- **Languages:**  
- **Skills:**  
- **Str** | **Dex** | **Con** | **Int** | **Wis** | **Cha**  
- **AC:**  
- **HP:**  
- **Immunities / Resistances / Weaknesses:**  

---
## Attacks
- **Melee:**  
- **Ranged:**  
- **Special Abilities:**  

---
## Tactics
How this variant fights, reacts, and uses its abilities.

---
## Loot
Treasure, reagents, special materials.

---
## Encounters
- [[ ]]
