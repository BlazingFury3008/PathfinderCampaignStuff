---
Type: Dashboard
Tags: [dashboard, dataview, pathfinder2e]
---
# 🧩 Campaign Progression

## 🏛️ Arches, Chapters & Sessions
```dataviewjs
// === Load Sessions ===
let sessions = dv.pages('"Session Notes"')
  .where(p => (p.tags && p.tags.includes("session")) || (p.Tags && p.Tags.includes("session")))
  .array();

// === Extract Arch + Chapter from folder path ===
sessions = sessions.map(p => {
    const parts = p.file.path.split("/");
    p.Arch = parts[1] ?? "";
    p.Chapter = parts[2] ?? "";
    return p;
});

// === Setup container ===
const container = this.container;

// === Search + Filters UI ===
const controls = container.createEl("div");
controls.style.display = "flex";
controls.style.flexWrap = "wrap";
controls.style.gap = "0.5em";
controls.style.marginBottom = "0.75em";

const search = controls.createEl("input", { type: "text", placeholder: "Search sessions..." });
search.style.padding = "8px";
search.style.flex = "1 1 200px";

// --- Helper for dropdowns ---
function makeSelect(label, options) {
  const wrap = controls.createEl("div");
  wrap.createEl("span", { text: label + ": " });
  const sel = wrap.createEl("select");
  ["All", ...options].forEach(o => sel.createEl("option", { text: o, value: o }));
  return sel;
}

// === Collect unique values ===
const allArches = Array.from(new Set(sessions.map(p => p.Arch))).sort();
const allChapters = Array.from(new Set(sessions.map(p => p.Chapter))).sort();
const allTypes = Array.from(new Set(sessions.map(p => p.Type).filter(Boolean))).sort();

const archSel = makeSelect("Arch", allArches);
const chapSel = makeSelect("Chapter", allChapters);
const typeSel = makeSelect("Type", allTypes);

const levelInput = controls.createEl("input", { type: "number", placeholder: "Min Level" });
levelInput.style.padding = "4px";
levelInput.style.width = "90px";

// === Table container ===
const tableWrapper = container.createEl("div");

// === Sorting state ===
let sortKey = "Session";
let sortAsc = true;

// === RENDER FUNCTION ===
function render() {
  tableWrapper.innerHTML = "";

  const q = (search.value || "").toLowerCase();
  const arch = archSel.value;
  const chap = chapSel.value;
  const type = typeSel.value;
  const minLvl = levelInput.value ? Number(levelInput.value) : null;

  // === Filtering ===
  let filtered = sessions.filter(p =>
    (!q || (`${p.Title} ${p.Location} ${p.file.name} ${p.file.content}`.toLowerCase().includes(q))) &&
    (arch === "All" || p.Arch === arch) &&
    (chap === "All" || p.Chapter === chap) &&
    (type === "All" || p.Type === type) &&
    (!minLvl || (Number(p.PartyLevel) >= minLvl))
  );

  // === Sorting ===
  filtered = filtered.sort((a, b) => {
    const A = (a[sortKey] ?? "").toString().toLowerCase();
    const B = (b[sortKey] ?? "").toString().toLowerCase();

    if (A < B) return sortAsc ? -1 : 1;
    if (A > B) return sortAsc ? 1 : -1;
    return 0;
  });

  // === Build Table ===
  const table = tableWrapper.createEl("table", { cls: "dataview table-view-table" });
  const thead = table.createEl("thead");
  const headerRow = thead.createEl("tr");

  const headers = [
    { label: "Arch", key: "Arch" },
    { label: "Chapter", key: "Chapter" },
    { label: "Session", key: "Session" },
    { label: "Title", key: "Title" },
    { label: "Location", key: "Location" },
    { label: "Type", key: "Type" },
    { label: "Level", key: "PartyLevel" }
  ];

  // === Header buttons for sorting ===
  headers.forEach(h => {
    const th = headerRow.createEl("th");
    const btn = th.createEl("button", { text: h.label });
    btn.style.all = "unset";
    btn.style.cursor = "pointer";
    btn.style.fontWeight = "bold";

    btn.onclick = () => {
      if (sortKey === h.key) sortAsc = !sortAsc;
      else { sortKey = h.key; sortAsc = true; }
      render();
    };

    if (sortKey === h.key) btn.textContent += sortAsc ? " ▲" : " ▼";
  });

  const tbody = table.createEl("tbody");

  // === Table rows ===
  filtered.forEach(p => {
    const row = tbody.createEl("tr");

    row.createEl("td", { text: p.Arch });
    row.createEl("td", { text: p.Chapter });
    row.createEl("td", { text: p.Session });

    // Make Title clickable
    const titleTd = row.createEl("td");
    const link = titleTd.createEl("a", { text: p.Title, href: p.file.path });
    link.addClass("internal-link");

    row.createEl("td", { text: p.Location ?? "" });
    row.createEl("td", { text: p.Type ?? "" });
    row.createEl("td", { text: p.PartyLevel ?? "" });
  });

  // === Empty state ===
  if (!filtered.length) {
    const row = tbody.createEl("tr");
    const td = row.createEl("td", { text: "No sessions found." });
    td.colSpan = headers.length;
    td.style.textAlign = "center";
    td.style.opacity = "0.6";
  }
}

// === Event Listeners ===
[search, archSel, chapSel, typeSel, levelInput].forEach(e => e.oninput = render);

// === Initial Render ===
render();

````




---
## 📚 Chapters Overview
```dataviewjs
const chapters = dv.pages('"Session Notes"')
  .where(p => p.Tags && p.Tags.includes("chapter"))
  .sort(p => [p.Arch ?? 0, p.Chapter ?? 0]);

dv.table(["Arch", "Chapter", "Status"], chapters.map(p => [p.Arch, p.Chapter, p.Status]));
```

---
## 🧍 NPC Directory

```dataviewjs
// ---- Filters (edit these) ----
const FILTER = {
  tag: "npc",
  location: "",     // e.g. "Eldermount" (blank = all)
  affiliation: "",  // e.g. "House Malakai" (blank = all)
};

// ---- Sorting (edit these) ----
const SORT = {
  name: true,
  location: true,
  affiliation: false,
};

// ---------- helpers ----------
const norm = (v) => (v ?? "").toString().trim();

function hasTag(p, tag) {
  const tags = p.tags ?? p.Tags;
  if (!tags) return false;
  if (Array.isArray(tags)) return tags.includes(tag);
  return norm(tags).includes(tag);
}

// Works with strings or dv link objects
function labelOf(v) {
  if (!v) return "";
  if (typeof v === "string") return v.replace(/^\[\[|\]\]$/g, "");
  if (v.display) return v.display;
  if (v.fileName) return v.fileName;
  if (v.path) return v.path.split("/").pop().replace(/\.md$/i, "");
  return "";
}

function isLink(v) {
  return v && typeof v === "object" && (v.path || v.fileName || v.display);
}

// ---------- pull NPCs ----------
let npcs = dv.pages('"Setting Notes/NPC"')
  .where(p => hasTag(p, FILTER.tag))
  .where(p => !FILTER.location || labelOf(p.Location) === FILTER.location)
  .where(p => !FILTER.affiliation || labelOf(p.Affiliation) === FILTER.affiliation);

// ---------- sorts (least important first) ----------
if (SORT.affiliation) npcs = npcs.sort(p => labelOf(p.Affiliation), "asc");
if (SORT.location) npcs = npcs.sort(p => labelOf(p.Location), "asc");
if (SORT.name) npcs = npcs.sort(p => norm(p.file.name), "asc");

// ---------- lookup map for plain-text affiliation/location -> link ----------
const linkIndex = new Map(
  [
    ...dv.pages('"Setting Notes/Factions"').map(f => [f.file.name, f.file.link]),
    ...dv.pages('"Setting Notes/Locations"').map(l => [l.file.name, l.file.link]),
  ]
);

// ---------- render ----------
dv.table(
  ["Name", "Location", "Affiliation"],
  npcs.map(p => {
    // Location: if already a link, render it; else try to link by filename
    const loc = p.Location;
    const locLabel = labelOf(loc);
    const locOut = isLink(loc) ? loc : (linkIndex.get(locLabel) ?? locLabel);

    // Affiliation: if already a link, render it; else try to link by filename
    const aff = p.Affiliation;
    const affLabel = labelOf(aff);
    const affOut = isLink(aff) ? aff : (linkIndex.get(affLabel) ?? affLabel);

    return [
      p.file.link,
      locOut ?? "",
      affOut ?? ""
    ];
  })
);

```
---
## 🐉 Creature Index

```dataviewjs
// === Load creature data ===
const creatures = dv.pages('"Setting Notes/Beastiary/Infoblock"')
  .where(p => p.tags && p.tags.includes("creature"))
  .sort(p => p.Level ?? 0, "asc");

// === UI setup ===
const container = this.container;
const filterBox = container.createEl("div");
filterBox.style.display = "flex";
filterBox.style.flexWrap = "wrap";
filterBox.style.gap = "0.5em";
filterBox.style.marginBottom = "1em";

// --- Helper for dropdowns ---
function makeSelect(label, options) {
  const wrap = filterBox.createEl("div");
  wrap.createEl("span", { text: label + ": " });
  const sel = wrap.createEl("select");
  ["All", ...options].forEach(o => sel.createEl("option", { text: o, value: o }));
  return sel;
}

// --- Controls ---
const raritySel = makeSelect("Rarity", ["Common","Uncommon","Rare","Unique"]);
const typeSel = makeSelect("Type", ["Aberration","Animal","Beast","Construct","Dragon","Elemental","Fey","Fiend","Humanoid","Undead","Other"]);
const alignSel = makeSelect("Alignment", [
  "Lawful Good","Neutral Good","Chaotic Good",
  "Lawful Neutral","Neutral","Chaotic Neutral",
  "Lawful Evil","Neutral Evil","Chaotic Evil"
]);

const search = filterBox.createEl("input", { type: "text", placeholder: "Search creatures..." });
search.style.padding = "4px";
search.style.flex = "1 1 200px";

// --- Container for the single table ---
const tableWrapper = container.createEl("div");

// === Render function (manual HTML table) ===
function render() {
  tableWrapper.innerHTML = ""; // clear previous table

  const q = search.value.toLowerCase();
  const rarity = raritySel.value;
  const type = typeSel.value;
  const align = alignSel.value;

  const filtered = creatures.filter(p =>
    (!q || p.file.name.toLowerCase().includes(q)) &&
    (rarity === "All" || p.Rarity === rarity) &&
    (type === "All" || p["Creature Type"] === type) &&
    (align === "All" || p.Alignment === align)
  );

  const table = tableWrapper.createEl("table", { cls: "dataview table-view-table" });
  const thead = table.createEl("thead");
  const headerRow = thead.createEl("tr");
  ["Creature", "Level", "Type", "Rarity", "Alignment", "Source"].forEach(h => {
    headerRow.createEl("th", { text: h });
  });

  const tbody = table.createEl("tbody");

  for (const p of filtered) {
    const row = tbody.createEl("tr");

    // --- Creature Name (create proper HTML link) ---
    const nameCell = row.createEl("td");
    const link = nameCell.createEl("a", { 
      text: p.file.name, 
      href: p.file.path 
    });
    link.addClass("internal-link"); // makes it behave like a normal Obsidian link

    // --- Other columns ---
    row.createEl("td", { text: p.Level ?? "" });
    row.createEl("td", { text: p.Type ?? "" });
    row.createEl("td", { text: p.Rarity ?? "" });
    row.createEl("td", { text: p.Alignment ?? "" });
    row.createEl("td", { text: p.Source ?? "" });
  }

  if (filtered.length === 0) {
    const row = tbody.createEl("tr");
    const td = row.createEl("td", { text: "No creatures found." });
    td.colSpan = 6;
    td.style.textAlign = "center";
    td.style.opacity = "0.6";
  }
}

// --- Event listeners ---
[raritySel, typeSel, alignSel, search].forEach(el => el.oninput = render);

// --- Initial render ---
render();
```
## 🐉 Statblock Index

```dataviewjs
// === Load creature data ===
const creatures = dv.pages('"Setting Notes/Beastiary/Statblock"')
  .where(p => p.tags && p.tags.includes("creature"))
  .sort(p => p.Level ?? 0, "asc");

// === UI setup ===
const container = this.container;
const filterBox = container.createEl("div");
filterBox.style.display = "flex";
filterBox.style.flexWrap = "wrap";
filterBox.style.gap = "0.5em";
filterBox.style.marginBottom = "1em";

// --- Helper for dropdowns ---
function makeSelect(label, options) {
  const wrap = filterBox.createEl("div");
  wrap.createEl("span", { text: label + ": " });
  const sel = wrap.createEl("select");
  ["All", ...options].forEach(o => sel.createEl("option", { text: o, value: o }));
  return sel;
}

// --- Controls ---
const raritySel = makeSelect("Rarity", ["Common","Uncommon","Rare","Unique"]);
const typeSel = makeSelect("Type", ["Aberration","Animal","Beast","Construct","Dragon","Elemental","Fey","Fiend","Humanoid","Undead","Other"]);
const alignSel = makeSelect("Alignment", [
  "Lawful Good","Neutral Good","Chaotic Good",
  "Lawful Neutral","Neutral","Chaotic Neutral",
  "Lawful Evil","Neutral Evil","Chaotic Evil"
]);

const search = filterBox.createEl("input", { type: "text", placeholder: "Search creatures..." });
search.style.padding = "4px";
search.style.flex = "1 1 200px";

// --- Container for the single table ---
const tableWrapper = container.createEl("div");

// === Render function (manual HTML table) ===
function render() {
  tableWrapper.innerHTML = ""; // clear previous table

  const q = search.value.toLowerCase();
  const rarity = raritySel.value;
  const type = typeSel.value;
  const align = alignSel.value;

  const filtered = creatures.filter(p =>
    (!q || p.file.name.toLowerCase().includes(q)) &&
    (rarity === "All" || p.Rarity === rarity) &&
    (type === "All" || p["Creature Type"] === type) &&
    (align === "All" || p.Alignment === align)
  );

  const table = tableWrapper.createEl("table", { cls: "dataview table-view-table" });
  const thead = table.createEl("thead");
  const headerRow = thead.createEl("tr");
  ["Creature", "Level", "Type", "Rarity", "Alignment", "Source"].forEach(h => {
    headerRow.createEl("th", { text: h });
  });

  const tbody = table.createEl("tbody");

  for (const p of filtered) {
    const row = tbody.createEl("tr");

    // --- Creature Name (create proper HTML link) ---
    const nameCell = row.createEl("td");
    const link = nameCell.createEl("a", { 
      text: p.file.name, 
      href: p.file.path 
    });
    link.addClass("internal-link"); // makes it behave like a normal Obsidian link

    // --- Other columns ---
    row.createEl("td", { text: p.Level ?? "" });
    row.createEl("td", { text: p.Type ?? "" });
    row.createEl("td", { text: p.Rarity ?? "" });
    row.createEl("td", { text: p.Alignment ?? "" });
    row.createEl("td", { text: p.Source ?? "" });
  }

  if (filtered.length === 0) {
    const row = tbody.createEl("tr");
    const td = row.createEl("td", { text: "No creatures found." });
    td.colSpan = 6;
    td.style.textAlign = "center";
    td.style.opacity = "0.6";
  }
}

// --- Event listeners ---
[raritySel, typeSel, alignSel, search].forEach(el => el.oninput = render);

// --- Initial render ---
render();
```
---
## 🏙️ Location Index
```dataviewjs
const locations = dv.pages('"Setting Notes/Locations"')
  .where(p => p.tags)
  .sort(p => p.Region ?? "");

dv.table(
  ["Location", "Region", "Type"],
  locations.map(p => [p.file.link, p.Region ?? "", p.Type ?? ""])
);
```
## 🏙️ POI Index
```dataviewjs
// ---- Filters (edit these) ----
const FILTER = {
  tag: "poi",            // required tag to include
  region: "",            // e.g. "Skadia" (blank = all)
  location: "",          // e.g. "Eldermount" (blank = all)
  poiType: "",           // e.g. "Fortress" (blank = all)
  faction: "",           // e.g. "House Malakai" (blank = all)
  hideHidden: false,      // exclude Status: Hidden
};

// ---- Sorting (edit these) ----
const SORT = {
  region: true,
  location: true,
  poiType: true,
  name: true,
};

const norm = (v) => (v ?? "").toString().trim();
const normLower = (v) => norm(v).toLowerCase();

let pois = dv.pages('"Setting Notes/Point of Interest"')
  .where(p => {
    const tags = p.tags;
    const hasTag = Array.isArray(tags)
      ? tags.includes(FILTER.tag)
      : norm(tags).includes(FILTER.tag);

    if (!hasTag) return false;
    if (FILTER.hideHidden && normLower(p.Status) === "hidden") return false;

    if (FILTER.location && !norm(p.Location).contains(FILTER.location)) return false;
    if (FILTER.poiType && norm(p.POIType) !== FILTER.poiType) return false;
    if (FILTER.faction && !norm(p.Faction).contains(FILTER.faction)) return false;

    return true;
  });

// Apply sorts (Dataview's sort is stable; apply least-important first)
if (SORT.name) pois = pois.sort(p => norm(p.file.name), "asc");
if (SORT.poiType) pois = pois.sort(p => norm(p.POIType), "asc");
if (SORT.location) pois = pois.sort(p => norm(p.Location), "asc");


// Faction link lookup
const factionByName = new Map(
  dv.pages('"Setting Notes/Factions"').map(f => [f.file.name, f.file.link])
);

dv.table(
  ["POI", "Location", "POI Type", "Faction"],
  pois.map(p => {
    const factionName = norm(p.Faction);
    const factionLink = factionByName.get(factionName);
    return [
      p.file.link,
      norm(p.Location),
      norm(p.POIType),
      factionLink ?? factionName
    ];
  })
);
```

---
## 🗃️ Quick Filters
#### 📜 In-Progress Sessions
```dataviewjs
const inProgress = dv.pages('"Session Notes"').where(p => p.Status === "In Progress");
dv.list(inProgress.file.link);
```
#### ✅ Completed Sessions
```dataviewjs
const done = dv.pages('"Session Notes"').where(p => p.Status === "Completed");
dv.list(done.file.link);
```
#### ⚙️ Active NPCs
```dataviewjs
const activeNPCs = dv.pages('"Setting Notes/NPC"').where(p => p.Status === "Active");
dv.list(activeNPCs.file.link);
```
---
## 📈 Party Tracking

```dataviewjs
const parties = dv.pages('"Session Notes"')
  .where(p => p.Tags && p.Tags.includes("session"))
  .sort(p => [p.Arch ?? 0, p.Chapter ?? 0, p.Session ?? 0]);
dv.table(
  ["Arch", "Chapter", "Session", "Party Level", "Location", "Status"],
  parties.map(p => [p.Arch, p.Chapter, p.Session, p.PartyLevel, p.Location, p.Status])
);
```
---