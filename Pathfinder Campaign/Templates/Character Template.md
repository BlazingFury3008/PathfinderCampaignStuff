<%*
const name = await tp.system.prompt("Enter character name");
tp.file.rename(name)

const player = await tp.system.prompt("Enter player's name");
const ancestry = await tp.system.prompt("Enter ancestry");
const charClass = await tp.system.prompt("Enter class");
const level = await tp.system.prompt("Enter character level", "1");
const background = await tp.system.prompt("Enter background");

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

const deity = await tp.system.prompt("Enter deity (if any)");
const origin = await tp.system.prompt("Enter origin (hometown, nation, or plane)");
const location = await tp.system.prompt("Enter current location");
const status = await tp.system.suggester(
  ["Active", "Inactive", "Deceased", "Missing", "Retired"],
  ["Active", "Inactive", "Deceased", "Missing", "Retired"],
  false,
  "Select character status"
);
%><% "---" %>
Name: <% name %>
Type: Player Character
Ancestry: <% ancestry %>
Class: <% charClass %>
Level: <% level %>
Background: <% background %>
Player: <% player %>
Alignment: <% alignment %>
Deity: <% deity %>
Origin: <% origin %>
Location: <% location %>
Status: <% status %>
tags: ["pc", "gm", "player-character", "pathfinder2e"]
<% "---" %>
# <% name %>
**Player:** <% player %>  
**Ancestry:** <% ancestry %>  
**Class:** <% charClass %> (Level <% level %>)  
**Background:** <% background %>  
**Alignment:** <% alignment %>  
**Deity:** <% deity || "None" %>  
**Origin:** <% origin %>  
**Location:** <% location %>  
**Status:** <% status %>  

---
## Overview
Brief summary of who this character is in the story.  
Include what the party and NPCs see vs. what only you (the GM) know.  
_Example:_ A wandering sellsword posing as a mercenary, secretly a fugitive from the Arcanum.

---
## History
Summarize key past events from the character’s backstory, focusing on details relevant to the campaign world.  
Include family, mentors, lost loves, past crimes, debts, or unresolved threads.  

---
## Party Dynamics
- **Relationships within the Party:**  
  - [[ ]] —  
  - [[ ]] —  
- **Interpersonal Conflicts or Bonds:**  
- **Leadership / Role in Group:**  

---
## Secrets & Hooks
- **Known Secrets (Player shared):**  
  -  
- **Hidden Secrets (GM-only):**  
  -  
- **Potential Plot Hooks:**  
  -  
  -  

---
## Goals
- **Short-Term (Player driven):**  
  -  
- **Long-Term (Narrative potential):**  
  -  
- **Personal Arc Resolution:**  
  -  

---
## Connections
- **Factions / Organizations:** [[ ]]  
- **Allies / Contacts:** [[ ]]  
- **Rivals / Enemies:** [[ ]]  
- **Important NPC Ties:** [[ ]]  

---
## Development Notes
Observations and adjustments for you as GM:  
- How they’re evolving through the campaign.  
- Themes or moral questions to highlight.  
- Possible turning points or temptations.  

---
## Appearances / Key Moments

---
## GM Notes
Freeform space for you to jot down story beats, changes, consequences, or foreshadowing ideas.
