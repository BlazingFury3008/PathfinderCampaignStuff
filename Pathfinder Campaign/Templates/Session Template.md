<%*
const sessionsRoot  = "Session Notes";          // Arc folders live here
const locationsRoot = "Setting Notes/Locations";

// ---------- helpers ----------
function byName(a, b) {
  return a.localeCompare(b, undefined, { sensitivity: "base" });
}

function getFolder(path) {
  const f = app.vault.getAbstractFileByPath(path);
  return (f && f.children) ? f : null;
}

async function ensureFolder(path) {
  const existing = app.vault.getAbstractFileByPath(path);
  if (!existing) await app.vault.createFolder(path);
}

function listImmediateSubfolders(path) {
  const folder = getFolder(path);
  if (!folder) return [];
  return folder.children
    .filter(c => c.children) // folders only
    .map(f => f.name)
    .sort(byName);
}

async function pickOrCreateFolder(parentPath, label) {
  await ensureFolder(parentPath);

  const existing = listImmediateSubfolders(parentPath);
  const options = [...existing, "Other"];

  const picked = await tp.system.suggester(options, options, false, `Select ${label}`);
  if (!picked) throw new Error(`No ${label} selected`);

  if (picked !== "Other") {
    return { name: picked, path: `${parentPath}/${picked}` };
  }

  const name = await tp.system.prompt(`Enter ${label} title`);
  if (!name || !name.trim()) throw new Error(`${label} title required`);

  const folderPath = `${parentPath}/${name.trim()}`;
  await ensureFolder(folderPath);

  return { name: name.trim(), path: folderPath };
}

// ---------- Arc / Chapter ----------
const arc = await pickOrCreateFolder(sessionsRoot, "Arc");
const chapter = await pickOrCreateFolder(arc.path, "Chapter");

// ---------- Session questions ----------
const session = await tp.system.prompt("Enter session number");
const title = await tp.system.prompt("Enter session title");
const partyLevel = await tp.system.prompt("Enter party level");

// ---------- Locations ----------
const allLocations = app.vault.getAllLoadedFiles()
  .filter(f => f.extension === "md" && f.path.startsWith(locationsRoot + "/"))
  .map(f => f.path.replace(locationsRoot + "/", ""))
  .sort(byName);

if (!allLocations.length) {
  await tp.system.prompt(`No location files found in '${locationsRoot}'. Press Enter.`);
  return;
}

const location = await tp.system.suggester(allLocations, allLocations, false, "Select Location");

// ---------- Session type ----------
const type = await tp.system.suggester(
  ["Main Story", "Side Quest", "Downtime", "Exploration", "Social", "Combat Focus", "Other"],
  ["Main Story", "Side Quest", "Downtime", "Exploration", "Social", "Combat Focus", "Other"],
  false,
  "Select session type"
);

// ---------- Rename + Move ----------
try { await tp.file.rename(`{session}. {title}`); } catch (_) {}

try {
  await tp.file.move(`${chapter.path}/${title}`);
} catch (_) {}

// ---------- Output ----------
tR += `---\n`;
tR += `Session: ${session}\n`;
tR += `Title: ${title}\n`;
tR += `Arc: ${arc.name}\n`;
tR += `Chapter: ${chapter.name}\n`;
tR += `Location: ${location}\n`;
tR += `PartyLevel: ${partyLevel}\n`;
tR += `Status: Planned\n`;
tR += `Type: ${type}\n`;
tR += `tags: ["session", "pathfinder2e", "ammiryon"]\n`;
tR += `---\n`;
tR += `# Session ${session} – ${title}\n`;
tR += `**Arc:** ${arc.name}  \n`;
tR += `**Chapter:** ${chapter.name}  \n`;
tR += `**Location:** [[${location}]]  \n`;
tR += `**Party Level:** ${partyLevel}  \n`;
tR += `**Session Type:** ${type}  \n\n`;
tR += `---\n`;
tR += `## Recap\n`;
tR += `Recap Text\n`;
tR += `## Planned Events\n`;
tR += `Any Planned Events\n`;
tR += `## Expected NPCs\n`;
tR += `- NPCs expected to show up during the session\n`;
tR += `## Planned Encounters\n`;
tR += `- Any Statblocks planned specifically for the session\n`;
tR += `## GM Notes:  \n`;
tR += `Short guidance on pacing, tone, and optional skill checks.  \n`;
tR += `_Example:_ Encourage exploration and roleplay. Allow downtime before the inciting event.\n\n`;
%>
