<%*
const name = await tp.system.prompt("Enter Event Name");
await tp.file.rename(name);

const year = await tp.system.prompt("Enter Event Year (e.g. 1067)");

const span = await tp.system.suggester(
  ["Global", "Continental", "Local"],
  ["Global", "Continental", "Local"],
  false,
  "Select the event span"
);

const locationFiles = app.vault.getFiles()
  .filter(f =>
    (
      f.path.startsWith("Setting Notes/Locations/") ||
      f.path.startsWith("Setting Notes/Point of Interest/")
    ) &&
    f.extension === "md"
  )
  .map(f =>
    f.path
      .replace(/^Setting Notes\/Locations\//, "")
      .replace(/^Setting Notes\/Point of Interest\//, "")
      .replace(/\.md$/, "")
  );

const location = await tp.system.suggester(
  locationFiles.length > 0 ? locationFiles : ["No locations found"],
  locationFiles.length > 0 ? locationFiles : [""],
  false,
  "Select the primary location of the event"
);
%><% "---" %>
Event Name: <% name %>
Year: <% year %>
Span: <% span %>
Location: <% location %>
tags: ["event", "history", "setting", "pathfinder2e"]
<% "---" %>
# <% name %>
**Year:** <% year || "Unknown Year" %>  
**Span:** <% span || "(Global, Continental, Local)" %>  
**Location:** [[<% location || "" %>]]  
**Key Figures:**  
- [[ ]]  
- [[ ]]  
- [[ ]]

---
## Overview
<% name %> was a historical event that took place in <% year || "an unknown year" %><% location ? " at " + location : "" %>.  
Write a short summary of the event, its significance, and how it is remembered.

---
## Causes
What led to this event?
-  
-  
-  

---
## Timeline/Description
Describe what happened during the event.  
Include its scale, atmosphere, immediate impact, and what people experienced.

---
## Major Participants
- [[ ]]
- [[ ]]
- [[ ]]

---
## Outcome / Consequences
Describe the result of the event and its lasting effects on the world, region, or people involved.

---
## Historical Importance
Explain why this event matters in the broader timeline of the setting.

---
## Related Locations
- [[<% location || "" %>]]
- [[ ]]
- [[ ]]

---
## Related Sessions
- [[ ]]
- [[ ]]