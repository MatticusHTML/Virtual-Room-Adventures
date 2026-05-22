# VRA // Virtual Room Adventures
# CURSOR OPERATOR PROTOCOL v2.2
# Changelog:
#   v2.2 - Deploy reworked for GitHub Desktop handoff. Agent no longer runs git; operator
#          commits + pushes in the GitHub Desktop app. Aligned to the real repo (incoming/
#          drop-off, inlined CSS/JS in index.html, real roster, portrait/profile-pic naming).
#   v2.1 - Formatting simplified for Cursor. Removed box-drawing menus and em dashes
#          (they render badly in Cursor unlike Codex). Plain markdown + fenced text only.
#          Added "activate" wake command. Expanded GitHub-Pages file structure.
#   v2.0 - Rebuilt for Cursor. Direct file edits replace the zip-drag workflow.
#          Folder-menu front door. Codex Sync intake folded in. Theme system added.

> **HOW TO USE THIS DOCUMENT**
> This is the front door to the VRA project in Cursor.
> The agent has the repo open and can read/write files and edit JSON directly. It does NOT run
> git - the operator commits and pushes using the GitHub Desktop app (see Section 7).
> Read this file fully before touching any file. It overrides prior habits from chat-only sessions.
> When in doubt, show the menu and wait. Never freelance.
>
> **WAKE COMMAND:** The operator types **`activate`** (any casing - `activate`, `ACTIVATE`, `Activate`)
> to boot the protocol. On wake, the agent loads this file and immediately displays the Folder Menu
> (Section 1). Nothing happens before `activate` - no edits, no file moves, no assumptions.

---

## SECTION 0 - HOW THIS WORKS IN CURSOR (READ FIRST)

This project used to run in a chat window where the assistant couldn't touch files - so the
old protocol made you drag zips into GitHub by hand. **That era is over.** In Cursor:

| Capability | Old (Codex/chat) | Now (Cursor) |
|---|---|---|
| Edit `index.html` (CSS + JS are inlined here) | Paste blocks, user commits | Agent edits the file directly |
| Update `data/characters.json` | Deliver isolated entry, user merges | Agent reads, appends, validates, saves in place |
| Add image assets | Zip + drag into github.com | User drops loose files in `incoming/`; agent files & renames them |
| Commit & push to live | Manual upload on github.com | Agent edits files; **operator commits + pushes in GitHub Desktop** |
| Generate tiles / profile pics | Codex image gen | **[ DO EXTERNALLY ]** - user makes them, agent wires them in |

**The one thing the agent still does NOT do:** generate images. Tiles, profile pics, cards,
and scenes are all created by the operator outside Cursor and dropped into the repo. The agent's
job for images is **placement and wiring only** - correct folder, correct filename, correct JSON path.

---

## SECTION 1 - FOLDER MENU (THE FRONT DOOR)

When the operator types **`activate`** (any casing), or `MENU` at any time, display this and wait:

```
VRA // OPERATOR MENU
Virtual Room Adventures

  1. Characters   - add / update roster
  2. Themes       - switch / build styling
  3. Reference    - style rules + file map
  4. Deploy       - review changes & hand off to GitHub Desktop

Live: matticushtml.github.io/Virtual-Room-Adventures
What would you like to do?
```

Wait for a number. Do not proceed until the operator picks one. Each branch has its own submenu below.

---

## SECTION 2 - BRANCH 1: CHARACTERS

The main workflow. When the operator picks `1`, show:

```
VRA // CHARACTERS

  A. Add New Character
  B. Update Existing Character
  C. Quick Intake (from card + transcript)

What would you like to do?
```

---

### 2A - ADD NEW CHARACTER

A guided pipeline. Stop and wait at every step. Never skip ahead.

**[ STEP 1 - COLLECT DATA ]**
Ask for all of the following at once. Generate nothing yet.
```
Provide the following for your new character:

Name:
Universe:
Faction (if any):    <- use INDEPENDENT if none
Class:
PC or NPC:
Ability name:
Ability description:
Quote:
Lore / Archive Note:
```

**[ STEP 2 - CONFIRM DATA ]**
Repeat it all back. Wait for `CONFIRM` or `EDIT [field]`.

**[ STEP 3 - IMAGES ] [ DO EXTERNALLY ]**
```
[ DO EXTERNALLY ] - Cursor does not generate images.

Generate these yourself and drop them in the incoming/ folder at the repo root:
  • Default tile      → 1024×1024 PNG (square, roster grid)
  • Default profile   → 1024×1536 PNG (tall, detail panel)
  • One profile per variant that has a card

Don't worry about folders or exact names - just drop the files in incoming/.
Naming them loosely after the character helps (e.g. "nicholas-tile.png",
"nicholas-injured-profile.png") but isn't required.

Type READY when the files are in incoming/.
```
On READY, the agent runs the **Incoming Intake** routine (Section 6): scans `incoming/`,
smart-matches each file, asks you to confirm, then moves + renames them to their final
paths. It does not continue until the inbox is empty and every needed image is filed.

**[ STEP 4 - WIRE IT IN ]**
The agent now does the work directly - no zip, no manual merge:
1. Confirms/creates the folder structure under `assets/characters/[id]/`
2. Reads `data/characters.json`, appends the new character object (Section 5 schema)
3. Validates the JSON parses cleanly and the asset paths point at files that exist
4. Reports what changed:
```
[ WIRED ]
  + assets/characters/[id]/  (tile, profile pic(s) verified on disk)
  + data/characters.json     (entry appended - roster now N characters)

Review the diff in Cursor. Ready to deploy? → Branch 4, or type DEPLOY.
```
The agent does NOT commit or push here, and never runs git. Deploy is its own deliberate handoff
step (Branch 4) where the agent summarizes changes for you to commit in GitHub Desktop.

---

### 2B - UPDATE EXISTING CHARACTER

**[ STEP 1 - SELECT ]** Ask which character. The agent reads `data/characters.json` and prints
that character's current data + asset inventory (which variants, how many cards, how many scenes).

**[ STEP 2 - SUBMENU ]**
```
UPDATE - [CHARACTER NAME]

  A. Replace Tile            [ EXTERNAL ]
  B. Replace Profile Pic(s)  [ EXTERNAL ]
  C. Add New Variant         [ EXTERNAL ]
  D. Edit Quote
  E. Edit Lore / Archive Note
  F. Edit Ability
  G. Edit Universe / Faction / Class
  H. Add Cards               [ EXTERNAL ]
  I. Add Scenes              [ EXTERNAL ]
  J. Remove Cards
  K. Remove Scenes

What would you like to update?
```

Touch ONLY what's selected. Text edits (D–G, J, K) → the agent edits `characters.json` directly,
shows the before/after, done. Image options (A, B, C, H, I) → marked `[ EXTERNAL ]`: agent tells
the operator the exact path + filename to drop the new image at, waits for `READY`, verifies the
file exists, then updates any JSON paths needed.

After any update:
```
Done. Anything else for [Name]? → submenu again, or DEPLOY when ready.
```

---

### 2C - QUICK INTAKE (from card + transcript)

For when you have a character card image and/or a campaign transcript and want the fields pulled
automatically instead of typing them.

**[ STEP 1 ]** `Drop the character card and/or paste the transcript. Type READY when submitted.`

**[ STEP 2 - EXTRACT ]** Silently read everything and fill the full field list (Section 5).
Extraction guidance:
- **Name** - card header or primary dialogue tag
- **Universe** - campaign setting / card subtitle
- **Faction** - card tags or story allegiance; INDEPENDENT if none
- **Class** - card role label, or combat role shown in the transcript
- **PC or NPC** - inferred from who the operator controls
- **Ability** - named power from the card or used in the story
- **Lore** - card summary, or a 2–3 sentence narrative summary
- **Never guess.** If a field isn't in the materials, flag it and ask.

**[ STEP 3 - QUOTE PICK ]** Offer 4 real spoken lines (not narration), exact wording, varied in tone:
```
Lines recorded from [NAME]:
  [1] "..."   [2] "..."   [3] "..."   [4] "..."
Enter 1–4, or type a custom quote.
```

**[ STEP 4 ]** Confirm the filled form → then hand off to **2A Step 3** (images) and continue the
normal add pipeline from there.

---

## SECTION 3 - BRANCH 2: THEMES

VRA ships with one locked theme today. This branch exists so you can switch themes for a push or
build new seasonal ones (Halloween, December Winter, etc.) later **without** touching this protocol.

```
VRA // THEMES

  ACTIVE:  Black / Blue Codex  (locked)

  1. View active theme tokens
  2. Switch active theme
  3. Create a new theme
```

> **For now, leave the active theme as Black / Blue Codex.** It is the project's identity.
> The switch and create options exist for the future - don't change the live theme unless the
> operator explicitly asks.

### Theme tokens (the current, locked theme)
```css
--black-0: #02040a;  --black-1: #050a13;
--blue-0:  #59d7ff;  --blue-1:  #2297ff;  --blue-2: #1a6fcc;
--panel:   rgba(3,9,18,.78);
--line:    rgba(91,215,255,.22);
--text:    #eaf7ff;  --muted: #8fa8bd;  --dim: #536c80;
--shadow:  rgba(0,0,0,.68);
```
Vibe: dark fantasy-tech codex - "an elf with an iPhone." Dark mode only, no toggle, ever.
NOT: sporty, esports, bright, generic sci-fi, plain wiki, corporate.

### Option 2 - Switch active theme
A theme is just a named set of the tokens above. Because CSS is inlined in `index.html`, the
theme tokens live in the `:root { ... }` block inside `index.html` (not a separate stylesheet).
To switch, the agent swaps the active token values there. To make future switching one-line,
the first theme job is a good time to refactor each theme into its own `:root[data-theme="..."]`
block. Confirm the swap, show the diff, then it's a normal deploy. **Never auto-switch** - only on
explicit request.

### Option 3 - Create a new theme (e.g. Halloween, December Winter)
Guided, and it does NOT disturb the active theme:
```
New theme name:           (e.g. "Halloween")
Keep the dark base?       (recommended yes - VRA is dark-only)
Accent shift:             (e.g. orange/violet smoke for Halloween;
                           ice-cyan/silver for December Winter)
Any seasonal accents?     (e.g. subtle snow, ember motes - optional)
```
The agent then writes a new theme token set alongside the existing one (commented, named, inactive),
and tells you how to activate it later via Option 2. The current theme stays the live default.

---

## SECTION 4 - BRANCH 3: REFERENCE

```
VRA // REFERENCE

  1. Style & Locked Rules
  2. File / Folder Map
  3. Image Display Rules
  4. Locked Identity Sheets (per character)
```

### 4.1 - Locked rules (no exceptions)
- **Dark mode only.** No light mode, no toggle.
- **"Voyage" is the whole site - never a filter option.** Universe is the top filter; "All" = everything.
- Universe filtering happens **inside** the archive, never on the landing page.
- Tiles show exactly two things: **PC/NPC tag** (top-right) and **character name** (bottom). Nothing else.
- **One tile per character** (default image only). Profile pic is per-variant.
- `VIEW CARDS` hidden when `cards[]` empty. `POPULAR SCENES` **always** visible (shows "no scenes yet" when empty).
- Detail-panel portrait area: **min 380px tall.**
- Font: **Azeret Mono**, fallback `"Consolas", "Courier New", monospace`.

### 4.2 - File / folder map (GitHub Pages friendly)

GitHub Pages serves from the repo **root** on the `main` branch, so `index.html` must sit at the
top level. Everything the site loads uses root-relative paths under `assets/` and `data/`, which
keeps links working identically on localhost and on the live `github.io` URL. The `incoming/` and
other underscore/dot items are working files - they live in the repo but are kept out of what ships
via `.gitignore`.

```
Virtual-Room-Adventures/          <- repo root = what GitHub Pages serves
│
├── index.html                    <- entry point. CSS + JS are INLINED here (no separate
│                                    style.css / script.js exist). Theme tokens live in here too.
│
├── data/
│   ├── characters.json           <- single source of truth for the roster (a JSON array)
│   └── voss-mercer-entry.json    <- leftover isolated entry from the old Codex add flow (safe to delete)
│
├── assets/                       <- everything the live site loads
│   ├── characters/
│   │   ├── nicholas/   default/ (tile.png, portrait.png), neck-injury/ (tile.png, portrait.png), cards/
│   │   ├── overseer-gwen/ default/ (tile.png, portrait.png), cards/
│   │   ├── austin/     default/ (tile.png, profile-pic.png), cards/
│   │   └── voss-mercer/ default/ (tile.png, profile-pic.png), cards/
│   │       (each character folder: default/ + any variant/ + cards/ + scenes/)
│   └── audio/
│       ├── menu-music.mp3
│       └── boot-line.mp3
│
├── docs/
│   ├── VRA_CURSOR_PROTOCOL.md     <- THIS FILE - keep it here, versioned with the repo
│   └── VRA_Character_Details_-_v1.3.md  <- old Codex intake doc (reference/archive)
│
├── incoming/                     <- DROP-OFF (working space, NOT served). Has a README.txt.
│                                    loose images land here → agent files them → empties
│
├── INSTALL_MANIFEST.md           <- leftover from the old zip workflow (safe to delete)
│
├── .cursorrules                  <- auto-loads this protocol every Cursor session
└── .gitignore                    <- ignores incoming/ drops + OS junk
```

**Reality notes (current repo state):**
- **No `style.css` / `script.js`** - they're inlined inside `index.html`. Theme color tokens
  therefore live in `index.html`, not a separate stylesheet (matters for the Themes branch).
- **Drop-off is `incoming/`** (already exists, with a README.txt) - this is the repo's real
  inbox. The protocol uses `incoming/` everywhere; there is no `_inbox/`.
- **Portrait filenames are mixed:** nicholas + overseer-gwen use `portrait.png`; austin +
  voss-mercer use `profile-pic.png`. Match whatever each character's JSON path says.
- **Leftovers from the old Codex/zip era** (safe to delete when you feel like tidying):
  `data/voss-mercer-entry.json`, `INSTALL_MANIFEST.md`.
- **Scenes:** Nicholas and Austin have empty `scenes` arrays in `characters.json` (no scene images
  filed yet). Overseer Gwen and Voss Mercer also have none. When scene images exist, drop them in
  `incoming/` and wire paths; until then, leave `scenes` as `[]`.

**Why this serves cleanly on GitHub Pages:**
- `index.html` at root → Pages finds it with zero config.
- All site paths are root-relative (`assets/...`, `data/...`) → links work the same locally and live.
- `incoming/` is gitignored → loose drops never deploy.
- Character `id` in JSON == its folder name under `assets/characters/` → one obvious place per character.

**Suggested `.gitignore`:**
```
incoming/*
!incoming/.gitkeep
.DS_Store
desktop.ini
Thumbs.db
```
> `incoming/*` ignores everything dropped in, while `!incoming/.gitkeep` keeps the empty folder
> tracked so it survives a fresh clone. (Your repo already has `incoming/README.txt`, which also
> keeps the folder alive - so the `.gitkeep` is optional here, but harmless.)

### 4.3 - Image display rules
| Context | CSS | Why |
|---|---|---|
| Roster tile | `object-fit: cover` | fills square cleanly |
| Detail portrait | `object-fit: cover` | controlled vertical framing |
| View Cards overlay | `object-fit: contain` | never crop card art |
| Scenes thumbnail | `object-fit: cover` | clean mood-board grid |
| Scene lightbox | `object-fit: contain` | show full image |

### 4.4 - Naming format (all lowercase, kebab-case)
```
[character-id]_[variant-id]_[asset-type].png
e.g. nicholas_default_tile.png · nicholas_neck-injury_profile-pic.png
```

### 4.5 - Locked identity sheets
Some characters have non-negotiable visual rules. Keep them here. Example on file:
**Nicholas** - Fallout / Vault 81. Tall lean athletic; sharp face; piercing blue eyes; dark swept hair;
black rectangular glasses. Blue/gold Vault 81 jumpsuit. **🔒 Vault-Tec logo on one side AND "81"
on the opposite side - in every asset, no exceptions.** Variants: `default`, `neck-injury`.

---

## SECTION 5 - CHARACTER DATA SCHEMA

All roster data lives in `data/characters.json` (a JSON array, one object per character). This is
the REAL shape currently live in the repo:
```json
{
  "id": "nicholas",
  "name": "Nicholas",
  "universe": "Fallout",
  "faction": "Vault 81",
  "pcNpc": "PC",
  "role": "PC",
  "class": "Vault Dweller",
  "quote": "I survived the Vault. The wasteland can get in line.",
  "lore": "A practical survivor from the Fallout campaign, built for bad odds.",
  "ability": "Adapt & Endure",
  "abilityText": "Thrives in harsh conditions, learns fast, survives the impossible.",
  "variants": [
    { "id": "default", "title": "Default",
      "tile": "assets/characters/nicholas/default/tile.png",
      "portrait": "assets/characters/nicholas/default/portrait.png" }
  ],
  "cards": [],
  "scenes": []
}
```
Notes from the live data, so you match what's already there:
- **`universe`** is the campaign world (e.g. "Fallout", "Marvel-DC Merged Universe"). **`faction`**
  is the in-world allegiance (e.g. "Vault 81", "Independent"). Do not confuse the two.
- Both **`pcNpc`** and a legacy **`role`** field exist on every current entry, usually with the same
  value. Keep `pcNpc` accurate; mirror it into `role` so nothing relying on the old field breaks.
- **Portrait filename is inconsistent in the live repo:** Nicholas and Overseer Gwen use
  `portrait.png`; Austin and Voss Mercer use `profile-pic.png`. Whatever the JSON path says is what
  must exist on disk. When adding new characters, pick one (the schema/protocol standard is
  `profile-pic.png`) and make the JSON path match the actual file. Don't rename existing ones
  without updating their JSON paths too.
- No `traits` and no stats in the live data - don't add them.

Graceful fallbacks: no tile/portrait → initials placeholder · no cards → hide View Cards ·
no scenes → button stays, shows "no scenes yet" · 0 variants → no selector · 1+ → show selector.
**Every character must have a `pcNpc` field** (missing one falls back to an old role label - bug).

---

## SECTION 6 - INCOMING INTAKE (`incoming/`)

The `incoming/` folder at the repo root is the single drop-off point for every image. You never
create folders or final filenames yourself - you drop loose files in `incoming/` and the agent files
them. This routine runs whenever the operator types `READY` during an image step, or `INCOMING` directly.

### The routine
1. **Scan** `incoming/` and list everything in it:
   ```
   [ INCOMING - 3 files ]
     • nicholas-tile.png
     • nicholas-injured-profile.png
     • IMG_4821.png
   ```
2. **Smart-match** each file from its name. The agent proposes a destination for each:
   ```
   Proposed filing:
     nicholas-tile.png            → nicholas / default / tile
     nicholas-injured-profile.png → nicholas / neck-injury / profile-pic
     IMG_4821.png                 → ??? (couldn't read this one - see below)

   CONFIRM to file these  |  FIX [filename] to correct  |  SKIP [filename] to leave in incoming/
   ```
   Matching cues: character name in the filename → `id`; "tile" → tile; "profile"/"pic"/"portrait"
   → profile-pic; "card" → cards/; "scene" → scenes/; a variant word the character already has
   (e.g. "injured") → that variant, else `default`.
3. **Always-ask fallback** - any file the agent can't confidently parse (like `IMG_4821.png`) is
   never guessed. The agent asks straight out:
   ```
   IMG_4821.png - which character? which variant? tile, profile, card, or scene?
   ```
4. **File on CONFIRM** - for each confirmed file the agent:
   - creates the destination folder if missing
   - **moves** the file (inbox ends empty) and renames it to kebab-case Section 4.4 format
   - updates the matching path in `data/characters.json` if relevant
5. **Log + report** - after filing:
   ```
   [ FILED ]
     nicholas-tile.png  →  assets/characters/nicholas/default/tile.png
     nicholas-injured-profile.png  →  assets/characters/nicholas/neck-injury/profile-pic.png
   [ INCOMING ] empty ✓   (1 skipped: IMG_4821.png)
   ```

### Rules
- **Only `incoming/` is a drop zone.** Files anywhere else in the repo are left alone.
- **Move, never copy** - a successful file leaves the inbox. Skipped/unparsed files stay so nothing
  is silently lost.
- **Never overwrite** an existing asset without asking - if the destination already exists, confirm
  replace vs. rename first.
- `incoming/` is working space, not site content. Add it to `.gitignore` (or keep it empty on commit)
  so loose drops never deploy.

---

## SECTION 7 - BRANCH 4: DEPLOY (HANDOFF TO GITHUB DESKTOP)

**The operator commits and pushes using the GitHub Desktop app - NOT the command line.**
The agent does NOT run git. There is no command-line git in this workflow, and the agent should
never try to run `git add`, `git commit`, or `git push`, never report git as "unavailable" or
"broken" (it isn't - it's just handled by GitHub Desktop), and never treat a missing command-line
git as a problem.

Deploy is a deliberate handoff step. When the operator types `DEPLOY` (or finishes an edit and is
ready to ship), the agent does this and only this:

```
VRA // DEPLOY - READY FOR GITHUB DESKTOP

  Changed since last commit:
  [agent lists the files it edited/created/deleted this session]

  Suggested commit message:
  [agent proposes one - see format below]

  Next step (in GitHub Desktop):
  1. Open GitHub Desktop - it will show these changes
  2. Review the diff
  3. Enter the commit message and click Commit to main
  4. Click Push origin
  GitHub Pages rebuilds in ~1 minute, then the live URL updates.
```

The agent's job at deploy = **summarize what changed + propose the commit message + hand off.**
Nothing more. It does not need to verify the push or check git status afterward; the operator
confirms in GitHub Desktop.

Rules:
- **One logical change per commit.** Don't mix a character add with a theme change - if the session
  touched two unrelated things, tell the operator so they can make two commits in GitHub Desktop.
- Always list exactly which files changed so the operator knows what they're committing.
- Commit message format (the agent proposes; the operator pastes it into GitHub Desktop):
  ```
  [VRA] Add [name] to roster
  [VRA] Update [name] - [what changed]
  [VRA] Add theme - [theme name]
  [VRA] Fix [specific issue]
  ```

---

## QUICK COMMANDS
| Command | Action |
|---|---|
| `activate` / `ACTIVATE` | Wake the protocol - load this file and show the Folder Menu |
| `MENU` | Return to the folder menu |
| `READY` | Materials/images are in the repo - continue |
| `CONFIRM` | Accept the data as shown |
| `EDIT [field]` | Correct one field |
| `QUOTES` | Re-show quote options |
| `DEPLOY` | Jump to Branch 4 |

---

```
END OF PROTOCOL - VRA_CURSOR_PROTOCOL_v2.2
LIVE  : https://matticushtml.github.io/Virtual-Room-Adventures/
REPO  : MatticusHTML/Virtual-Room-Adventures
```

## CHANGELOG
| Version | Change |
|---|---|
| v1.0–v1.9 | Chat/Codex era. Menu system, PC/NPC, Safe Add vs Full Repackage, stats removed. |
| v2.0 | Rebuilt for Cursor. Direct file edits + real git replace the zip-drag workflow. Folder-menu front door. Codex Sync intake folded in as Quick Intake. Theme system added (current theme locked, new themes addable). Image steps marked [ DO EXTERNALLY ]. Added `incoming/` drop-off folder - loose files in, agent smart-matches + confirms + moves to final paths, inbox ends empty. |
| v2.1 | Formatting simplified for Cursor: box-drawing menus, em dashes, arrows, and ellipses replaced with plain markdown + ASCII (Codex rendered them fine; Cursor does not). Added `activate` wake command. Expanded GitHub-Pages-friendly file structure with `.gitignore` guidance. |
| v2.2 | Deploy reworked for GitHub Desktop handoff - agent edits files but never runs git; operator commits + pushes in GitHub Desktop. Capability table, menu, and `.cursorrules` updated so the agent stops reaching for command-line git and stops flagging it as broken. |
