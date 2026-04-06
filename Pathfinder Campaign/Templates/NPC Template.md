<%*
const byName = (a, b) => a.localeCompare(b, undefined, { sensitivity: "base" });

async function listMarkdownBasenames(folderPath) {
  const folder = app.vault.getAbstractFileByPath(folderPath);
  if (!folder || folder.children === undefined) return [];

  const results = [];
  const walk = (f) => {
    if (f.children) for (const child of f.children) walk(child);
    else if (f.extension === "md") results.push(f.basename);
  };
  walk(folder);

  return results.sort(byName);
}

async function pickFromList(title, items, allowEmpty = false) {
  if (!items?.length) {
    return await tp.system.prompt(`${title} (no options found—type value manually)`);
  }
  const display = allowEmpty ? ["(None)", ...items] : items;
  const values  = allowEmpty ? ["", ...items] : items;
  return await tp.system.suggester(display, values, false, title);
}

const wl = (name) => name ? `[[${name}]]` : "";

// --- Name / rename ---
const name = await tp.system.prompt("Enter NPC name");
await tp.file.rename(name);

// --- Location dropdown -> wikilink ---
const locationOptions = await listMarkdownBasenames("Setting Notes/Locations");
const locationName = await pickFromList("Select NPC location", locationOptions, false);
const locationLink = locationName;

// --- Affiliation: choose type then dropdown -> wikilink ---
const affiliationType = await tp.system.suggester(
  ["Faction", "Location", "Independent"],
  ["Faction", "Location", "Independent"],
  false,
  "Affiliation type"
);

let affiliationLink = "";

if (affiliationType === "Faction") {
  const factionOptions = await listMarkdownBasenames("Setting Notes/Factions");
  const factionName = await pickFromList("Select Faction affiliation", factionOptions, false);
  affiliationLink = factionName;
} else if (affiliationType === "Location") {
  const affLocName = await pickFromList("Select Location affiliation", locationOptions, false);
  affiliationLink = affLocName;
}

// --- Ancestry / Class free text ---
const ancestryFinal = (await tp.system.prompt("Enter ancestry"))?.trim() || "";
const classFinal = (await tp.system.prompt("Enter class (optional)"))?.trim() || "";

// --- Status dropdown ---
const status = await tp.system.suggester(
  ["Active", "Inactive", "Deceased", "Missing", "Unknown"],
  ["Active", "Inactive", "Deceased", "Missing", "Unknown"],
  false,
  "Select NPC status"
);

// --- Optional role text for body only ---
const roleText = (await tp.system.prompt("Enter NPC role (for write-up only; optional)"))?.trim() || "";
%><%"---"%>
Name: <% name %>
Type: NPC
Ancestry: <% ancestryFinal %>
<%* if (classFinal) { %>Class: <% classFinal %>
<%* } %>Affiliation: <% affiliationLink || "Independant" %>
Location: <% locationLink || "Unknown" %>
Status: <% status %>
tags: ["npc", "pathfinder2e"]
---
# <% name %>
<%* if (roleText) { %>**Role:** <% roleText %>  
<%* } %>**Affiliation:** <% wl(affiliationLink) || "Independent" %>  
**Location:** <% wl(locationLink) || "Unknown" %>  
**Ancestry:** <% ancestryFinal || "Unknown" %><%* if (classFinal) { %>  
**Class:** <% classFinal %><%* } %>  
**Status:** <% status %>  

---
## Appearance
Brief description of attire, physical traits, and notable features.

---
## Personality
- **Demeanor:**  
- **Motivations:**  
- **Flaws / Secrets:**  
- **Voice / Quirk:**  

---
## Background
Short history or connection to the city/region.

---
## Relationships
- **Allies:** [[ ]]  
- **Rivals:** [[ ]]  
- **Related Factions/Groups:** [[ ]]  

---
## Plot Role
- Current relevance in the story.  
- Hooks or scenes involving them.

---
## Appearances
List sessions or scenes where this NPC appears.  
- [[ ]]

---
## Notes
Freeform details, quotes, or session takeaways.
