---
name: 3d-print-info-stamp
description: >-
  Standard workflow for OpenSCAD parts that deboss a three-line info stamp on a
  print-bed face: JSON config per product folder, generate stamp_generated.scad,
  and export STLs. Use when adding or changing print stamps, print-stamp-config.json,
  export scripts, or CI for 3D prints.
---

# 3D print info stamp (OpenSCAD)

## Purpose

Keep **legible, slice-aligned** debossed metadata on chosen parts (usually the **bottom** that touches the build plate) while avoiding hand-editing OpenSCAD for every build.

- **Line 1:** `brand_name`
- **Line 2:** `product_name`
- **Line 3:** `product_version` + (`preview` ? ` Prototype` : ``) — version only when not preview.

Text is **centered** on the bed face (part axis); export writes **`info_stamp_line1`–`info_stamp_line3`** into `stamp_generated.scad`.

Example (preview): `Verminology` / `Liquid Bait Station` / `v2.4 Prototype`

Release (`preview` false): line 3 is **`v2.4`** only (no “Prototype”).

**Naming:** In **`print-stamp-config.json`**, the boolean **`preview`** controls only the **stamp text** (appends ` Prototype` on line 3). OpenSCAD should use a **different** variable for mesh quality — e.g. **`mesh_preview`** in `common.scad` (low `$fn` when true). **Export scripts** always pass **`-D mesh_preview=false`** and **`-D crosssection_view=false`** so STL builds are full-detail and never cross-sectioned; do not put those in JSON (avoids clashing with the word “preview” in config).

## When to apply

- User adds or edits **`print-stamp-config.json`** at the **product** folder (see layout below).
- User asks how to **flag** which exported STLs get a stamp.
- Setting up **CI** or local **export** for stamped OpenSCAD builds.

## Repo layout (per product)

Put **one** `print-stamp-config.json`, **`stamp_generated.scad`**, and **`scripts/`** in the **product** directory (e.g. `liquid-bait-station/`). Put major-version OpenSCAD sources in a subfolder (e.g. `v2-clam-shell/`).

| Artifact | Role |
|----------|------|
| `print-stamp-config.json` | Brand, product, version, **`preview`**, `exports[]` (input `.scad` under a version folder; output `.stl` often under **`latest-build/`**) |
| `stamp_generated.scad` | **Generated** assignments: `info_stamp_line1`–`3`, `{stem}_stamp_bottom` — defaults can be committed so F5 works without export |
| `scripts/Export-OpenScadStl.ps1` | Write `stamp_generated.scad` → run OpenSCAD per `exports` |

Version subfolders hold **`.scad`** sources. Write exported STLs to a dedicated folder such as **`latest-build/`**. Add **`latest-build/*.stl`** to `.gitignore` only if you want CI or local exports to stay out of git; otherwise track them like any other artifact. OpenSCAD in a subfolder should **`include <../stamp_generated.scad>`** (path relative to the **including** `.scad` file’s directory).

See [reference.md](reference.md) for the JSON schema and stem naming.

## Flagging which parts get a stamp

In `exports`, each entry describes one STL (paths relative to the product folder):

```json
{
  "scad": "v2-clam-shell/reservoir.scad",
  "stl": "latest-build/reservoir.stl",
  "stamp": { "face": "bottom" }
}
```

Use `stamp: null` (or omit `stamp`) for parts with **no** stamp.

The export script maps the **basename** of `scad` → stem `reservoir` → OpenSCAD variable **`reservoir_stamp_bottom`**. The part’s `.scad` must call the shared deboss module with that flag and flat OD, e.g. `part_bottom_info_stamp_deboss(station_stamp_bottom, station_od);` inside a `difference()` for the body that includes the stamped face (stamp may be offset along +Y toward mid-radius — see `res_bottom_mark_radial_shift_fraction` in `common.scad`).

## Build order (agents and humans)

1. Edit **`print-stamp-config.json`**: set **`preview`**, version strings, and which exports have `stamp.face == "bottom"`.
2. Run **`Export-OpenScadStl.ps1`** (or **`export-open-scad-stl.sh`** on Linux/macOS CI):
   - Compose the three stamp lines from the template rules.
   - Regenerate **`stamp_generated.scad`** in the product folder (do not hand-edit after export).
   - Invoke OpenSCAD for each export with **fixed** defines: **`mesh_preview=false`**, **`crosssection_view=false`** (implemented in the export scripts, not in JSON).

## OpenSCAD integration checklist

- **`include <../stamp_generated.scad>`** (from a version subfolder) after print-depth / typography constants, before modules that reference `info_stamp_*` or `*_stamp_bottom`.
- Shared module **`part_bottom_info_stamp_deboss(enable, part_od)`** performs the deboss (shared depth, up to three `text()` lines; position uses `part_od` for radial offset from disc center).
- Per-part: subtract only when `enable` is true and at least one line is non-empty.

## CI / GitHub Actions

- On Linux runners, install **OpenSCAD** and **jq**, set **`working-directory`** to the **product** folder, run **`./scripts/export-open-scad-stl.sh`** (same behavior as `Export-OpenScadStl.ps1`).
- Point artifact paths at **`latest-build/*.stl`** (or whatever `exports[].stl` directory you use).

## Source control hygiene

- **`stamp_generated.scad`** can stay checked in with **sensible defaults** so OpenSCAD preview works without running export; export overwrites it before STL builds.

## Windows PowerShell 5.1

- Use **`Export-OpenScadStl.ps1`** with Windows PowerShell; the scripts avoid `utf8NoBOM` and `ProgramFiles(x86)` patterns that break on 5.1. PowerShell 7+ is also fine.

## Extension: other faces or products

- Add new `face` values in config and branch in generated variables (e.g. `{stem}_stamp_top`); extend the shared OpenSCAD module or add a second module.
- Reuse one `print-stamp-config.json` per product folder; add another product tree with the same pattern.
