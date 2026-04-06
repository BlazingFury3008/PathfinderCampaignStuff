<%*
const poiName = await tp.system.prompt("Enter Point of Interest name");

// === Base folder path ===
const basePath = "Setting Notes/Point of Interest";

// === Collect region folders (ignore .md files) ===
const allFolders = app.vault.getAllLoadedFiles()
  .map(f => f.path)
  .filter(p => p.startsWith(basePath + "/") && !p.endsWith(".md"))
  .map(p => p.replace(basePath + "/", ""));

// === Choose region folder ===
let chosenFolder = "";
if (allFolders.length > 0) {
  chosenFolder = await tp.system.suggester(
    allFolders,
    allFolders,
    false,
    "Select folder (region) for this Point of Interest"
  );
} else {
  // create base folder if missing
  await app.vault.createFolder(basePath);
  new Notice(`⚠️ No subfolders found in '${basePath}'. Created the base folder.`);
  return;
}

// === Ensure destination folder exists ===
const targetFolder = `${basePath}/${chosenFolder}`;
if (!app.vault.getAbstractFileByPath(targetFolder)) {
  await app.vault.createFolder(targetFolder);
}

// === Move and rename the file ===
await tp.file.move(`${targetFolder}/${poiName}.md`);
await tp.file.rename(poiName);

// === Prompt for metadata ===
const type = await tp.system.suggester(
  ["Landmark", "Building", "Ruins", "Temple", "Cavern", "District", "Fortress", "Natural Feature", "Other"],
  ["Landmark", "Building", "Ruins", "Temple", "Cavern", "District", "Fortress", "Natural Feature", "Other"],
  false,
  "Select Point of Interest type"
);

// === Location options ===
const locationFiles = app.vault.getFiles()
  .filter(f => f.path.startsWith("Setting Notes/Locations/") && f.extension === "md")
  .map(f => f.path.replace(/^Setting Notes\/Locations\//, "").replace(/\.md$/, ""));

const location = await tp.system.suggester(
  locationFiles.length ? [...locationFiles, "Other"] : ["No locations found"],
  locationFiles.length ? [...locationFiles, "Other"] : [""],
  false,
  "Select associated Location (from Locations folder)"
);

// === Faction options ===
const factionFiles = app.vault.getFiles()
  .filter(f => f.path.startsWith("Setting Notes/Factions/") && f.extension === "md")
  .map(f => f.path.replace(/^Setting Notes\/Factions\//, "").replace(/\.md$/, ""));

const faction = await tp.system.suggester(
  factionFiles.length ? ["None", ...factionFiles] : ["None"],
  factionFiles.length ? ["None", ...factionFiles] : ["None"],
  false,
  "Select associated Faction (or choose 'None')"
);

// === Status selector ===
const status = await tp.system.suggester(
  ["Active", "Abandoned", "Hidden", "Destroyed", "Unknown"],
  ["Active", "Abandoned", "Hidden", "Destroyed", "Unknown"],
  false,
  "Select current status"
);
%><% "---" %>
Name: <% poiName %>
Type: Point of Interest
POIType: <% type %>
Location: <% location %>
Faction: <% faction %>
Region: <% chosenFolder %>
Status: <% status %>
tags: ["poi", "setting", "pathfinder2e"]
<% "---" %>
# <% poiName %>
**Type:** <% type %>  
**Location:** <% location && location !== "No locations found" ? `[[Setting Notes/Locations/${location}|${location.split("/").pop()}]]` : "None" %>  
**Faction:** <% faction === "None" ? "None" : `[[Setting Notes/Factions/${faction}|${faction.split("/").pop()}]]` %>  
**Status:** <% status %>  

---
## Overview
A short summary of this point of interest, including its purpose and significance.

---
## Description
In-game narration details — architecture, mood, sounds, and atmosphere.

---
## History / Lore
Background or notable historical events tied to this location.

---
## Current Role
How the place functions today, and who controls or visits it.

---
## Key Figures
- [[ ]] — Guardian / Keeper  
- [[ ]] — Researcher / Explorer  
- [[ ]] — Notable Visitor  

---
## Secrets & Hooks
-  
-  

---
## Related Factions
- <% faction === "None" ? "" : `[[Setting Notes/Factions/${faction}|${faction.split("/").pop()}]]` %>  
- [[ ]]  

---
## Encounters
Potential encounters, challenges, or traps at this POI.

---
## Notes
Freeform space for discoveries, rumors, or connections.
