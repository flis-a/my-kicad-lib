# KiCad Component Library

Personal, reusable KiCad **symbol / footprint / 3D-model** library, shared across all my PCB
projects and configured with a **single environment variable**.

## Structure

```
<repo root>/            ← point KICAD_USER_LIB here
├─ symbols/             *.kicad_sym                 (schematic symbols)
├─ footprints/          <Lib>.pretty/*.kicad_mod    (footprints)
├─ 3dmodels/            <Lib>.3dshapes/*.step|*.wrl  (3D bodies)
├─ sym-lib-table        ready-to-use symbol library table
├─ fp-lib-table         ready-to-use footprint library table
├─ .gitignore
└─ README.md
```

Four libraries, kept split by source so imports stay tidy: **Andrew-Custom** (my own parts),
**EasyEDA**, **Samacsys**, **Snapeda** — 4 symbol libs, 38 footprints, 36 3D models.

## Why this layout is portable

Everything resolves from **one variable, `KICAD_USER_LIB`**, pointed at the repo root:

- Footprints reference 3D models as `${KICAD_USER_LIB}/3dmodels/<Lib>.3dshapes/<model>`
- The library tables reference symbols/footprints as `${KICAD_USER_LIB}/symbols|footprints/...`

Clone the repo anywhere, set that one path, and every symbol → footprint → 3D link resolves —
no absolute paths, nothing machine-specific. (The source files used KiCad's built-in
`KICAD_3RD_PARTY`, which is reserved for Plugin & Content Manager installs; this repo uses a
dedicated `KICAD_USER_LIB` so it stays self-contained and can't collide with PCM.)

## One-time setup in KiCad 10

1. Clone it, e.g. `git clone <url> ~/KiCad/kicad-library`
2. **Preferences → Configure Paths…** → add one variable:
   `KICAD_USER_LIB` = `/home/<you>/KiCad/kicad-library`   *(the repo root)*
3. Add the four libraries to your **global** tables — nicknames must stay exactly
   `Andrew-Custom`, `EasyEDA`, `Samacsys`, `Snapeda` (symbols link to footprints by nickname):
   - **Easiest:** copy the four `(lib …)` lines from the provided `sym-lib-table` /
     `fp-lib-table` into your global tables
     (`~/.config/kicad/10.0/sym-lib-table` and `fp-lib-table`).
   - **Or via GUI:** *Preferences → Manage Symbol / Footprint Libraries → Global* → add each
     `symbols/*.kicad_sym` and `footprints/*.pretty`.
4. Restart KiCad — symbols now show their footprints and 3D bodies.

> Setting `KICAD_USER_LIB` is the entire per-machine configuration.

## Updating: `git pull` = libraries update

KiCad reads these libraries live from the folder, so updating is just a pull:

```
cd ~/KiCad/kicad-library && git pull
```

New/changed parts appear after a library reload (or restart) — no re-import, and every project
using the global tables gets them at once. To freeze a specific project to a library version,
add this repo as a git **submodule** in that project and point its `KICAD_USER_LIB` at the
submodule.

## Adding new parts (keeps the structure consistent)

Impart saves per-source `.pretty` / `.3dshapes` folders and writes 3D paths with
`${KICAD_3RD_PARTY}`. To fold an import in, drop each piece into the matching subfolder and
rewrite that one variable:

1. `symbols/`    ← the new/updated `.kicad_sym`
2. `footprints/` ← the new `.pretty`
3. `3dmodels/`   ← the new `.3dshapes`

Then fix the 3D paths in the new footprints (one command on Linux) and commit:

```
sed -i 's#${KICAD_3RD_PARTY}/#${KICAD_USER_LIB}/3dmodels/#g' footprints/*/*.kicad_mod
git add -A && git commit -m "Add <part>" && git push
```

## KiCad 9 → 10 notes

- **File-format version ≠ app version.** The `(version YYYYMMDD)` line in each file is the
  file-format version, not the KiCad version — yours are `20241209` (Andrew-Custom, KiCad-9 era)
  and `20211014` (EasyEDA/Samacsys/Snapeda, from the vendor generators). **KiCad 10 reads them
  all** and silently upgrades a file's format the first time you edit and save it in 10.
- **Upgrade is one-way.** Once KiCad 10 rewrites a file, KiCad 9 may not open it — a non-issue
  since you're going 10-only.
- **Config is per-version.** KiCad 10 stores settings separately from 9
  (`~/.config/kicad/10.0/` vs `9.0/`), so set `KICAD_USER_LIB` and the tables in KiCad 10.
  KiCad 10 offers to import your 9.0 settings on first launch, which carries them over.
- **Footprints & 3D models need no migration.** `.kicad_mod`, `.step`, `.wrl` are read natively.
  The legacy `.dcm` description files were intentionally dropped — KiCad 7+ embeds descriptions
  in `.kicad_sym`.
