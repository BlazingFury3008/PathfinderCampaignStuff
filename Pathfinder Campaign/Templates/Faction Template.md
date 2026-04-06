<%*
const name = await tp.system.prompt("Enter faction name");
tp.file.rename(name)

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
  "Select faction alignment"
);

// --- Search for all Markdown files in /Setting Notes/Locations (including subfolders) ---
const locationFiles = app.vault.getFiles()
  .filter(f => (f.path.startsWith("Setting Notes/Locations/") || f.path.startsWith("Setting Notes/Point of Interest")) && f.extension === "md")
  .map(f => f.path.replace(/^Setting Notes\/Locations\//, "").replace(/\.md$/, "")); 
// removes 'Setting Notes/Locations/' prefix and '.md' suffix for clean names

// Suggester for headquarters selection
const headquarters = await tp.system.suggester(
  locationFiles.length > 0 ? locationFiles : ["No locations found"],
  locationFiles.length > 0 ? locationFiles : [""],
  false,
  "Select faction headquarters (from Locations folder and subfolders)"
);

const influence = await tp.system.suggester(
  ["Local", "Regional", "Global"],
  ["Local", "Regional", "Global"],
  false,
  "Select faction influence scope"
);

const status = await tp.system.suggester(
  ["Active", "Inactive", "Disbanded", "Unknown"],
  ["Active", "Inactive", "Disbanded", "Unknown"],
  false,
  "Select faction status"
);
%><% "---" %>
Name: <% name %>
Type: Faction
Alignment: <% alignment %>
Headquarters: <% headquarters.split("/").pop() %>
Influence: <% influence %>
Status: <% status %>
tags: ["faction", "pathfinder2e"]
<% "---" %>
# <% name %>

**Alignment:** <% alignment %>  
**Headquarters:** [[<% headquarters.split("/").pop() %>]]  
**Influence:** <% influence %>  
**Status:** <% status %>  

---
## Overview
A short summary of what this faction is, their reputation, and how they’re generally perceived.  
_Example:_ The Covenant of Eternal Ascendance is a secretive order devoted to unlocking divine transformation through forbidden magic.

---
## Goals & Motivations
-  
-  
-  

---
## Structure & Hierarchy
Describe how the group is organized — ranks, leadership, and command style.  
_Example:_ Led by an Archmagister supported by a council of four “Ascendants,” each controlling a different aspect of the order.

---
## Ideology & Beliefs
Summarize what they stand for, their philosophy, or their code of conduct.  
_Example:_ They believe mortality is a curse imposed by the gods, and only by reclaiming divine power can true freedom be achieved.

---
## Methods & Operations
How do they act on their goals? What are their common tactics, symbols, or areas of expertise?  
_Example:_ Espionage, relic recovery, ritual experimentation, political infiltration.

---
## Key Members
- [[ ]] — Leader or founder  
- [[ ]] — Second-in-command or specialist  
- [[ ]] — Notable member or contact  

---
## Relationships
- **Allies:** [[ ]]  
- **Rivals / Enemies:** [[ ]]  
- **Associated Locations:** [[ ]]  

---
## Resources & Assets
List ships, temples, guildhalls, magical relics, armies, or other holdings.  
_Example:_ Hidden libraries beneath Eldermount; network of sympathetic mages across Lakon.

---
## Role in the Campaign
Explain how this faction connects to your current story arc or characters.  
_Example:_ Provides information and funding to the party but conceals its true motives.

---
## Symbols & Colors
- **Symbol:**  
- **Colors:**  
- **Motto / Saying:**  

---
## Appearances

---
## Notes
Freeform section for miscellaneous lore, rumors, or quotes.  
_Example:_ “They seek the keys that bind the heavens — and they’ve already found two.”
