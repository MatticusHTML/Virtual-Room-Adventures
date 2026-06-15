# VRA // Virtual Room Adventures — Operator Workflow

> **Purpose of this document:** Explain how the VRA project is built and operated in Cursor, so another AI (or collaborator) can review the workflow and give advice. This is a human-readable companion to the stricter agent protocol at `docs/VRA_CURSOR_PROTOCOL.md`.
>
> **Live site:** https://matticushtml.github.io/Virtual-Room-Adventures/  
> **Repo:** GitHub Pages from repo root on `main` — `index.html` at top level.

---

## 1. What VRA Is

**Virtual Room Adventures (VRA)** is a game-style **character archive / codex** deployed on GitHub Pages. It is not a full game engine — it is a dark, cinematic roster browser:

- **Landing gate** → boot terminal animation → **Character Select** archive
- Filter by **Universe** (not "Voyage" — Voyage is the whole site name)
- Click tiles → detail panel with ability, quote, lore, variants, cards, scenes
- **One tile per character** on the grid (always the default variant’s tile image)
- Menu music + optional low-performance mode

**Visual identity (locked):** Dark mode only. Black + blue codex palette. Font: **Azeret Mono**. No light mode toggle.

---

## 2. Roles: Operator vs Cursor Agent

| Who | Does |
|-----|------|
| **Operator (Matthew)** | Makes creative decisions, generates all images externally, drops files in `incoming/`, commits and pushes via **GitHub Desktop** |
| **Cursor agent** | Edits files directly (`index.html`, `data/characters.json`, asset paths), runs the guided menu workflow, files images from `incoming/`, validates JSON |

**Hard boundaries for the agent:**

- **Does NOT** run `git add`, `commit`, or `push` — deploy is a handoff to GitHub Desktop
- **Does NOT** generate images (tiles, portraits, cards, scenes) — only placement and wiring
- **Does NOT** freelance — follows menu branches and waits for operator choices (`CONFIRM`, `READY`, etc.)
- **Does NOT** change the global theme unless explicitly asked (Themes branch)

---

## 3. Booting a Session

The operator types **`activate`** (any casing) or **`MENU`** to return to the front door.

```
VRA // OPERATOR MENU

  1. Characters   - add / update roster
  2. Themes       - switch / build styling
  3. Reference    - style rules + file map
  4. Deploy       - review changes & hand off to GitHub Desktop
```

Each branch has its own submenu. The agent stops at every decision point.

**Quick commands:**

| Command | Meaning |
|---------|---------|
| `activate` / `MENU` | Show folder menu |
| `CONFIRM` | Accept data or filing proposal |
| `EDIT [field]` | Fix one field before proceeding |
| `READY` | Images are in `incoming/` — run intake |
| `DEPLOY` | Summarize changed files + suggest commit message |

---

## 4. Adding a Character (Main Workflow)

Path: **Menu → 1 Characters → A Add New Character**

### Step 1 — Collect data (operator fills in)

```
Name:
Universe:
Faction (if any):     <- INDEPENDENT if none
Class:
PC or NPC:
Ability name:
Ability description:
Quote:
Lore / Archive Note:
```

Optional to discuss upfront: extra **variants**, **cards**, **scenes**.

There is **no separate “Tags” field** in the live JSON — tags often map to `faction` or get folded into lore.

### Step 2 — Confirm

Agent repeats everything back. Operator replies **`CONFIRM`** or **`EDIT [field]`**.

Auto-generated **`id`**: kebab-case from name (e.g. `Sovrax` → `sovrax`).

### Step 3 — Images [ DO EXTERNALLY ]

Operator creates PNGs and drops **loose files** into:

```
Virtual-Room-Adventures/incoming/
```

**Standard assets:**

| Asset | Size | Purpose |
|-------|------|---------|
| Default tile | 1024×1024 | Roster grid (square) |
| Default profile | 1024×1536 | Detail panel portrait |
| Extra variant profile | 1024×1536 | One per alt outfit |
| Cards | varies | View Cards overlay |
| Scenes | varies | Popular Scenes (optional) |

Loose filenames are fine (`sovrax-tile.png`, `Sovrax - Portrait.png`, etc.).

When done, operator types **`READY`**.

### Step 4 — Incoming intake (agent)

1. **Scan** `incoming/` and list files
2. **Smart-match** each filename to character / variant / asset type
3. **Ask** if any file is ambiguous — never guess
4. Operator **`CONFIRM`** filing plan (or `FIX` / `SKIP`)
5. Agent **moves** (not copies) files to final paths, **renames** to convention, **empties** inbox (except skipped files)
6. Agent **appends** entry to `data/characters.json`, validates JSON + file existence

### Step 5 — Wire report

```
[ WIRED ]
  + assets/characters/[id]/  (tile, profile(s), cards verified)
  + data/characters.json     (entry appended — roster now N)

Review diff in Cursor. Deploy when ready.
```

---

## 5. Operator Conventions (Learned in Practice)

These are project rules the operator has reinforced beyond the base protocol:

### 5.1 One tile per character — always default

- The roster grid shows **exactly one tile** per character (`variants[0]` in code).
- **Alternative outfits (variants) change portrait (+ card) only** — not the grid tile.
- In JSON, alt variants can point `tile` at the same path as default:

```json
{
  "id": "chitin",
  "title": "Chitin",
  "tile": "assets/characters/sovrax/default/tile.png",
  "portrait": "assets/characters/sovrax/chitin/profile-pic.png"
}
```

### 5.2 Portrait filename standard for new characters

Older entries mix `portrait.png` and `profile-pic.png`. **New characters** should use **`profile-pic.png`** on disk; JSON key remains `"portrait"`.

### 5.3 Image compression

Operator compresses PNGs externally before drop-off. Large uncompressed tiles/portraits were a major load-time factor on archive entry.

### 5.4 Character-specific visual overrides ≠ Themes

The global **Black / Blue Codex** theme stays locked.

**Per-character overrides** are scoped CSS/JS in `index.html` — not a new `:root` theme.

**Example: Sovrax (`id: sovrax`)**

- Tile hover/select: crimson glow instead of blue
- Selected tile: animated **blood drip** layer (CSS + JS)
- Detail panel: red accents via `[data-character="sovrax"]`
- Disabled in **LITE** mode and `prefers-reduced-motion`

Future special characters would follow the same pattern — scoped overrides, not Theme branch changes.

---

## 6. Updating an Existing Character

Path: **Menu → 1 → B Update Existing Character**

Agent loads current JSON + asset inventory, then submenu:

```
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
```

Text edits → agent patches JSON. Image edits → operator drops in `incoming/`, `READY`, intake routine.

---

## 7. Quick Intake (Optional)

Path: **Menu → 1 → C Quick Intake**

For character card images and/or campaign transcripts. Agent extracts fields, offers quote picker, then joins the normal add pipeline at Step 3 (images).

---

## 8. Deploy Workflow

Path: **Menu → 4 Deploy** or operator types **`DEPLOY`**

Agent lists files changed in session, proposes commit message. Operator uses **GitHub Desktop**:

1. Review diff  
2. Commit to `main`  
3. Push origin  
4. GitHub Pages rebuilds (~1 minute)

**Commit message format:**

```
[VRA] Add [name] to roster
[VRA] Update [name] - [what changed]
[VRA] Add theme - [theme name]
[VRA] Fix [specific issue]
```

One logical change per commit when possible.

---

## 9. File / Folder Map

```
Virtual-Room-Adventures/          <- GitHub Pages root
├── index.html                    <- ALL CSS + JS inlined here (no separate .css/.js)
├── data/
│   └── characters.json           <- Single source of truth (JSON array)
├── assets/
│   ├── characters/
│   │   └── [character-id]/
│   │       ├── default/          <- tile.png, profile-pic.png (or portrait.png legacy)
│   │       ├── [variant-id]/     <- profile only for alt outfits
│   │       ├── cards/
│   │       └── scenes/           <- optional
│   └── audio/
│       ├── menu-music.mp3
│       └── boot-line.mp3
├── incoming/                     <- DROP ZONE ONLY (gitignored loose files)
├── docs/
│   ├── VRA_CURSOR_PROTOCOL.md    <- Agent source of truth (strict)
│   └── VRA_WORKFLOW.md           <- This file (human + external AI review)
└── .cursorrules                  <- Auto-loads protocol every session
```

**Path rules:**

- All site paths are **root-relative** (`assets/...`, `data/...`)
- Character `id` in JSON **must match** folder name under `assets/characters/`
- `incoming/` is gitignored — drops never deploy

---

## 10. Character JSON Schema (Live Shape)

```json
{
  "id": "sovrax",
  "name": "Sovrax",
  "universe": "Larion",
  "faction": "League of Adventurers",
  "pcNpc": "PC",
  "role": "PC",
  "class": "Bard / Rogue / Blood Mage",
  "quote": "The world is a stage, and I always take the best seat in the house.",
  "lore": "A theatrical blood-mage adventurer...",
  "ability": "Deep Siphon",
  "abilityText": "Sovrax drains the vitality and essence...",
  "variants": [
    {
      "id": "default",
      "title": "Default",
      "tile": "assets/characters/sovrax/default/tile.png",
      "portrait": "assets/characters/sovrax/default/profile-pic.png"
    },
    {
      "id": "chitin",
      "title": "Chitin",
      "tile": "assets/characters/sovrax/default/tile.png",
      "portrait": "assets/characters/sovrax/chitin/profile-pic.png"
    }
  ],
  "cards": [
    "assets/characters/sovrax/cards/default-card.png",
    "assets/characters/sovrax/cards/chitin-card.png"
  ],
  "scenes": []
}
```

**Notes:**

- **`universe`** = campaign world filter; **`faction`** = in-world allegiance
- Both **`pcNpc`** and legacy **`role`** should match
- No stats / traits array in live data
- Empty `cards[]` → hide View Cards button
- Empty `scenes[]` → Popular Scenes still visible, shows “no scenes yet”

---

## 11. UI / Display Rules (Locked)

- Tiles show **only**: PC/NPC tag (top-right) + character name (bottom)
- Universe filter lives **inside** archive, not landing page
- Detail portrait area: **min 380px** tall
- Cards in modal: `object-fit: contain` (never crop card art)
- Tiles / portraits: `object-fit: cover`

---

## 12. Site Features (Current `index.html`)

### 12.1 Boot flow

1. Landing gate with ambient effects + “Click To Begin”
2. Terminal boot sequence (~2.6s intentional delay + transition)
3. Character select archive + menu music

### 12.2 Audio controls (archive only)

After entering character select:

- Play / pause menu music
- Volume slider (persisted in `localStorage`)

### 12.3 LITE — low performance mode

Button in top-right control cluster (with audio), **only visible on character select** (not landing gate).

- Toggles `body.perf-lite`
- Disables heavy ambient animations, smoke, drips, etc.
- Preference saved: `localStorage` key `vra-perf-lite`
- Can still apply on landing if previously enabled (effects off, button hidden until archive)

### 12.4 Performance notes (recent work)

- Ambient effects lightweight pass: fewer blur layers, no `backdrop-filter` on panel
- Large PNGs and `menu-music.mp3` preload remain main load factors
- Operator compresses images; music kept as-is by choice

---

## 13. Themes Branch (Mostly Future)

Path: **Menu → 2 Themes**

Global theme is **Black / Blue Codex** — locked unless operator explicitly requests a switch.

Themes would swap CSS custom properties in `index.html` `:root` block. **Character-specific overrides (like Sovrax blood effects) are not themes** — they are per-`id` CSS/JS hooks.

---

## 14. Example End-to-End: Sovrax (May 2026)

1. Operator provided character sheet + images in chat
2. Data collected → **CONFIRM**
3. Files dropped in `incoming/` (5 PNGs)
4. Agent filed:
   - `default/tile.png`, `default/profile-pic.png`
   - `chitin/profile-pic.png` (no separate chitin tile)
   - `cards/default-card.png`, `cards/chitin-card.png`
5. Appended to `characters.json` → roster 5
6. Later: Sovrax-only crimson tile glow + blood drip + red detail panel accents in `index.html`

---

## 15. Current Roster (as of doc write)

| ID | Name | Universe | PC/NPC |
|----|------|----------|--------|
| nicholas | Nicholas | Fallout | PC |
| overseer-gwen | Overseer Gwen | Fallout | NPC |
| austin | Austin | Fallout | NPC |
| voss-mercer | Voss Mercer | Marvel-DC Merged Universe | PC |
| sovrax | Sovrax | Larion | PC |

---

## 16. Questions You Might Ask Another AI About

Useful angles for advice when sharing this doc:

1. **Scaling character-specific effects** — pattern for N characters without bloating `index.html`
2. **Variant system** — JSON shape vs UI (single tile, multi-portrait) — cleaner data model?
3. **Performance** — balance cinematic boot + effects vs GitHub Pages static hosting
4. **Asset pipeline** — compression targets, WebP, lazy-loading portraits on archive entry
5. **Protocol ergonomics** — menu-driven agent vs lighter workflow for repeat operators
6. **Incoming intake** — filename matching reliability, validation scripts
7. **Separation of concerns** — when to extract inlined CSS/JS without breaking Pages deploy

---

## 17. Related Files

| File | Role |
|------|------|
| `docs/VRA_CURSOR_PROTOCOL.md` | Strict agent protocol (v2.2) — overrides assumptions |
| `.cursorrules` | Cursor auto-load summary + hard rules |
| `data/characters.json` | Roster data |
| `index.html` | Entire frontend (style, behavior, audio, effects) |
| `incoming/README.txt` | Drop zone reminder for operator |

---

*Document version: 1.0 — written for external review, May 2026*
