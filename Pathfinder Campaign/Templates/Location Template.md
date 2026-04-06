<%*
const name = await tp.system.prompt("Enter Location name");
tp.file.rename(name)

const region = await tp.system.prompt("Enter the region or continent this location is in");

const type = await tp.system.suggester(
  ["City", "Town", "Village", "Ruins", "Dungeon", "Fortress", "Temple", "Wilderness", "Plane", "Other"],
  ["City", "Town", "Village", "Ruins", "Dungeon", "Fortress", "Temple", "Wilderness", "Plane", "Other"],
  false,
  "Select the type of location"
);
%><% "---" %>
Location Name: <% name %>
Type: <% type %>
Region: <% region %>
tags: ["location", "setting", "pathfinder2e"]
<% "---" %>
# <% name %>
**Region:** <% region || "Unknown Region" %>  
**Type:** <% type || "(City, Dungeon, Ruins, etc.)" %>  
**Key NPCs:**  
- [[ ]]  
- [[ ]]  
- [[ ]]

---
## Overview
<% name %> is a location within <% region || "a known region" %>.  
Write a short description of the area’s purpose, significance, and reputation here.

---
## Description
Details for in-game narration — architecture, mood, people, sounds, and culture.  
Mention what the players experience upon arriving.

---
### Districts
#### District 1
##### Description
##### Locations of Interest
- 

---
## Lore / History
Document key background events, ancient origins, or political changes tied to this location.

---
## Hooks
-  
-  

---
## Related Sessions
- [[ ]]
- [[ ]]
