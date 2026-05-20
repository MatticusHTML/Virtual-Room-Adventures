# VRA Install Manifest — Voss Mercer

⚠️ Extract this zip FIRST. Upload the CONTENTS, not the zip file itself.
GitHub will not unzip automatically.

Steps:
1. Extract zip
2. Open github.com → MatticusHTML/Virtual-Room-Adventures
3. Click Add file → Upload files
4. Drag extracted folders into upload area
5. If you already have a `data/characters.json`, merge in `data/voss-mercer-entry.json` instead of overwriting the entire file.
6. Upload matching images under `assets/characters/<id>/` (see folder layout in the entry JSON).
7. Hit the green Commit changes button

**Character fields:** `id`, `name`, `universe`, `faction`, `pcNpc`, `role`, `class`, `quote`, `lore`, `ability`, `abilityText`, `variants`, `cards`, `scenes` (no stats).

Commit message:
[VRA] Add voss mercer to roster
