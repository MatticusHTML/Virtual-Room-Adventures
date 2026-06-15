# VRA Workflow Overview

**Virtual Room Adventures (VRA)** is a game-style character archive — a dark-mode "codex" site where tabletop/RPG campaign characters are catalogued with tiles, profile portraits, cards, and scenes. It is deployed as a static site on **GitHub Pages** at:

**https://matticushtml.github.io/Virtual-Room-Adventures/**

This document explains how the project is operated day-to-day: who does what, how content flows from idea to live site, and what rules constrain the system. It is written for an external advisor (e.g. Claude) who needs full context without reading every file in the repo.

---

## 1. What VRA Is (Product Summary)

- A **character roster browser** with universe filtering, search, and a detail panel per character.
- Visual identity: **dark fantasy-tech codex** — "an elf with an iPhone." Black + blue palette, **Azeret Mono** font, dark mode only (no light mode, no toggle).
- Each character has:
  - **One roster tile** (square, 1024×1024) — always the default variant only.
  - **Profile portrait(s)** per variant (tall, 1024×1536).
  - Optional **cards** (character card art, shown in overlay).
  - Optional **scenes** (mood-board thumbnails + lightbox).
- Data fields: name, universe, faction, class, PC/NPC, ability, quote, lore, variants.
- **"Voyage"** is the name of the whole site experience — it is never a filter option. **Universe** is the top-level filter inside the archive.

---

## 2. Architecture (Technical Summary)

| Layer | What it is |
|---|---|
| **Hosting** | GitHub Pages, served from repo root on `main` |
| **Entry point** | `index.html` at repo root — **all CSS and JS are inlined** (no separate `style.css` / `script.js`) |
| **Data** | `data/characters.json` — single source of truth, JSON array of character objects |
| **Assets** | `assets/characters/[character-id]/` — tiles, portraits, cards, scenes per character |
| **Audio** | `assets/audio/` — menu music, boot line click sound |
| **Working inbox** | `incoming/` at repo root — drop zone for loose images; gitignored so it never deploys |
| **Operator protocol** | `docs/VRA_CURSOR_PROTOCOL.md` — full step-by-step rules for the Cursor AI agent |
| **Agent guardrails** | `.cursorrules` — auto-loaded summary that points the agent at the protocol |

**Path convention:** All site paths are root-relative (`assets/...`, `data/...`) so they work identically on localhost and GitHub Pages.

**Character ID = folder name:** If JSON says `"id": "nicholas"`, assets live under `assets/characters/nicholas/`.

---

## 3. Roles: Operator vs. Cursor Agent

This is a **human-in-the-loop, menu-driven workflow**. The operator (Matthew) and a Cursor AI agent collaborate, but responsibilities are strictly divided.

### Operator (human) does:
- Chooses actions from menus (`activate` → pick branch → pick sub-option).
- **Creates all images externally** (tiles, profile pics, cards, scenes) — the agent never generates images.
- Drops loose image files into `incoming/`.
- Reviews diffs in Cursor.
- **Commits and pushes via GitHub Desktop** — not command-line git.

### Cursor agent does:
- Reads `docs/VRA_CURSOR_PROTOCOL.md` at session start.
- Presents menus and **waits** at every decision point — never skips ahead or freelances.
- Edits `index.html`, `data/characters.json`, and asset paths directly in the repo.
- Runs the **Incoming Intake** routine: scan `incoming/`, smart-match filenames, confirm with operator, move + rename to final paths, update JSON.
- On deploy: lists changed files, proposes a commit message, hands off to GitHub Desktop.
- **Never runs git** (`git add`, `commit`, `push` are explicitly out of scope).

### Wake command
The operator types **`activate`** (any casing) to boot the protocol. Until then, the agent does nothing — no edits, no assumptions. **`MENU`** returns to the top-level folder menu at any time.

---

## 4. Top-Level Menu (Front Door)

When activated, the agent shows:

```
VRA // OPERATOR MENU

  1. Characters   - add / update roster
  2. Themes       - switch / build styling
  3. Reference    - style rules + file map
  4. Deploy       - review changes & hand off to GitHub Desktop
```

The operator picks a number. Each branch has its own submenu. The agent never proceeds without an explicit choice.

---

## 5. Branch 1: Characters (Main Workflow)

### Submenu
```
  A. Add New Character
  B. Update Existing Character
  C. Quick Intake (from card + transcript)
```

### 5A — Add New Character (full pipeline)

A guided, step-by-step pipeline. The agent stops and waits at every step.

**Step 1 — Collect data**
The operator provides all fields at once:
- Name, Universe, Faction (or INDEPENDENT), Class, PC or NPC
- Ability name + description, Quote, Lore / Archive Note

**Step 2 — Confirm**
Agent repeats everything back. Operator says `CONFIRM` or `EDIT [field]`.

**Step 3 — Images [ DO EXTERNALLY ]**
Operator generates images outside Cursor and drops them in `incoming/`:
- Default tile → 1024×1024 PNG
- Default profile → 1024×1536 PNG
- One profile per variant that has a card

Loose filenames are fine (e.g. `nicholas-tile.png`). Operator types `READY` when done.

**Step 4 — Wire it in**
Agent:
1. Confirms/creates folder structure under `assets/characters/[id]/`
2. Appends new character object to `data/characters.json`
3. Validates JSON parses and asset paths point at real files
4. Reports what changed — does **not** commit

### 5B — Update Existing Character

Agent reads `characters.json`, shows current data + asset inventory, then submenu:

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

Text edits → agent edits JSON directly. Image options → operator drops file, types `READY`, agent verifies and updates paths. Agent touches **only** what was selected.

### 5C — Quick Intake

For when the operator has a character card image and/or campaign transcript instead of typing fields manually.

1. Operator drops card and/or pastes transcript, types `READY`.
2. Agent extracts fields silently (never guesses missing data — asks instead).
3. Agent offers 4 real spoken quote lines from the material; operator picks or writes custom.
4. Confirm → then continues from Add New Character Step 3 (images).

---

## 6. Incoming Intake (`incoming/`)

The **only** drop zone for images. Operator never creates final folders or filenames manually.

### Routine (on `READY` or `INCOMING`)

1. **Scan** — list everything in `incoming/`
2. **Smart-match** — propose destination per file from filename cues:
   - Character name → `id`
   - "tile" → tile asset
   - "profile" / "pic" / "portrait" → profile pic
   - "card" → `cards/`
   - "scene" → `scenes/`
   - Variant word (e.g. "injured") → that variant, else `default`
3. **Always-ask fallback** — unparseable filenames (e.g. `IMG_4821.png`) are never guessed; agent asks directly
4. **File on CONFIRM** — create folders, **move** (not copy) + rename to kebab-case, update JSON paths
5. **Report** — log where each file went; inbox should end empty (skipped files stay)

### Rules
- Move, never copy — successful files leave the inbox
- Never overwrite existing assets without confirming
- `incoming/` is gitignored — loose drops never deploy

### Naming format (final paths)
```
[character-id]_[variant-id]_[asset-type].png
e.g. nicholas_default_tile.png · nicholas_neck-injury_profile-pic.png
```

On disk, files often live at paths like `assets/characters/nicholas/default/tile.png` (folder structure, not flat kebab-case filenames — the protocol describes both the folder layout and the rename convention used during intake).

---

## 7. Character Data Schema

All roster data in `data/characters.json` (JSON array):

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
  "lore": "A practical survivor from the Fallout campaign...",
  "ability": "Adapt & Endure",
  "abilityText": "Thrives in harsh conditions, learns fast...",
  "variants": [
    {
      "id": "default",
      "title": "Default",
      "tile": "assets/characters/nicholas/default/tile.png",
      "portrait": "assets/characters/nicholas/default/portrait.png"
    }
  ],
  "cards": [],
  "scenes": []
}
```

**Important quirks in live data:**
- Both `pcNpc` and legacy `role` exist — keep them in sync.
- Portrait filename is inconsistent: some characters use `portrait.png`, others `profile-pic.png`. JSON path must match the actual file on disk.
- No `traits` or stats — don't add them.
- Every character **must** have `pcNpc`.

**UI fallbacks:** missing tile/portrait → initials placeholder; no cards → hide View Cards; no scenes → button stays, shows "no scenes yet".

---

## 8. Branch 2: Themes

Currently one locked theme: **Black / Blue Codex**. New themes can be built alongside (Halloween, winter, etc.) but the active theme only changes on explicit request.

Theme tokens live in the `:root { ... }` block inside `index.html` (because CSS is inlined). Future refactor idea: `:root[data-theme="..."]` blocks for one-line switching.

---

## 9. Branch 3: Reference (Locked Rules)

These are non-negotiable product rules:

| Rule | Detail |
|---|---|
| Dark mode only | No light mode, no toggle |
| Voyage ≠ filter | Universe is the top filter; "All" = everything |
| Tile content | PC/NPC tag (top-right) + character name (bottom) only |
| One tile per character | Default variant only; profile pic is per-variant |
| View Cards | Hidden when `cards[]` is empty |
| Popular Scenes | Always visible; shows "no scenes yet" when empty |
| Portrait area | Min 380px tall in detail panel |
| Font | Azeret Mono |

### Image display (CSS object-fit)
| Context | Fit |
|---|---|
| Roster tile | cover |
| Detail portrait | cover |
| View Cards overlay | contain |
| Scenes thumbnail | cover |
| Scene lightbox | contain |

### Locked identity sheets
Some characters have non-negotiable visual rules documented in the protocol (e.g. Nicholas: Vault-Tec logo + "81" on jumpsuit in every asset).

---

## 10. Branch 4: Deploy (GitHub Desktop Handoff)

Deploy is a **deliberate handoff**, not an automated push.

When operator types `DEPLOY`, the agent:
1. Lists every file changed this session
2. Proposes a commit message
3. Tells operator to commit + push in **GitHub Desktop**

**Commit message format:**
```
[VRA] Add [name] to roster
[VRA] Update [name] - [what changed]
[VRA] Add theme - [theme name]
[VRA] Fix [specific issue]
```

**Rules:** One logical change per commit. Agent does not verify push or check git status afterward.

---

## 11. Quick Commands Reference

| Command | Action |
|---|---|
| `activate` | Wake protocol, show Folder Menu |
| `MENU` | Return to Folder Menu |
| `READY` | Materials/images are in repo — continue |
| `CONFIRM` | Accept data as shown |
| `EDIT [field]` | Correct one field |
| `QUOTES` | Re-show quote options |
| `DEPLOY` | Jump to Deploy branch |

---

## 12. File / Folder Map

```
Virtual-Room-Adventures/          ← repo root = GitHub Pages serves this
├── index.html                    ← entry point; CSS + JS + theme tokens inlined
├── data/
│   └── characters.json           ← roster source of truth
├── assets/
│   ├── characters/
│   │   └── [character-id]/
│   │       ├── default/          ← tile.png, portrait.png or profile-pic.png
│   │       ├── [variant-id]/     ← optional extra variants
│   │       ├── cards/
│   │       └── scenes/
│   └── audio/
├── docs/
│   ├── VRA_CURSOR_PROTOCOL.md    ← full operator protocol (v2.2)
│   └── VRA_WORKFLOW_OVERVIEW.md  ← this file
├── incoming/                     ← image drop zone (gitignored)
├── .cursorrules                  ← agent guardrails
└── .gitignore
```

---

## 13. Evolution: Old Workflow → Current Workflow

VRA originally ran in a **chat-only** environment (Codex) where the assistant could not touch files. The old flow required:
- Pasting code blocks for the user to merge manually
- Zipping assets and dragging into GitHub's web UI
- Isolated JSON entry files merged by hand

**Current workflow (Cursor era, protocol v2.2):**
- Agent edits files directly in the open repo
- Images drop into `incoming/` instead of manual folder creation
- Deploy handoff goes to **GitHub Desktop**, not command-line git
- Image generation is always external — agent only files and wires paths

---

## 14. Current Roster (as of writing)

Characters in `data/characters.json` include Nicholas, Overseer Gwen, Austin, Voss Mercer, Sovrax, and others across universes (primarily Fallout and Marvel-DC Merged Universe). Nicholas has two variants (`default`, `neck-injury`). Most characters have cards; scenes arrays are largely empty pending future assets.

---

## 15. Design Philosophy Behind the Workflow

1. **Menu-driven, never freestyle** — reduces agent hallucination and scope creep; operator always knows what step they're on.
2. **Strict separation of concerns** — human creates art, agent handles file ops and JSON; human owns git.
3. **Single source of truth** — `characters.json` drives everything; folder names match character IDs.
4. **Confirm before destructive actions** — data confirm, intake confirm, overwrite confirm.
5. **Working space isolated from deploy** — `incoming/` gitignored so WIP images never ship.
6. **Protocol as law** — `VRA_CURSOR_PROTOCOL.md` overrides agent assumptions; `.cursorrules` reinforces it every session.

---

## 16. Open Questions / Areas Where Advice May Be Useful

When sharing this doc with an advisor, these are common topics worth discussing:

- **Protocol ergonomics** — Is the menu + wait-at-every-step model too slow, or the right guardrail for a solo operator + AI?
- **Incoming intake** — Filename smart-matching vs. a structured drop template (e.g. required naming convention upfront).
- **Schema consistency** — `portrait.png` vs `profile-pic.png` drift; whether to normalize retroactively.
- **Monolithic `index.html`** — When/if to split CSS/JS; impact on theme switching and maintainability.
- **Quick Intake quality** — Extracting character fields from cards + transcripts; quote selection UX.
- **Scale** — What breaks first at 50+ characters? 100+? Search, load time, JSON size, asset organization.
- **Variant model** — One tile per character but multiple variants with their own portraits — is this intuitive for visitors?
- **Deploy handoff** — GitHub Desktop-only vs. optional CLI; whether the agent should ever stage files.
- **Testing / validation** — Automated JSON schema validation, broken-path checks before deploy.
- **Theme system** — `data-theme` refactor timing; seasonal themes without protocol churn.

---

## 17. Key Source Files (for deeper dives)

| File | Purpose |
|---|---|
| `docs/VRA_CURSOR_PROTOCOL.md` | Full operator protocol v2.2 — authoritative step-by-step |
| `.cursorrules` | Short guardrails loaded every Cursor session |
| `data/characters.json` | Live roster data |
| `index.html` | Entire site UI, styles, and behavior |
| `incoming/README.txt` | Quick reference for the drop zone |

---

*Generated for external review. For authoritative operational steps, always defer to `docs/VRA_CURSOR_PROTOCOL.md`.*
