# print-stamp-config.json reference

## Top-level fields

| Field | Type | Description |
|-------|------|-------------|
| `brand_name` | string | Line 1 (e.g. `Verminology`). |
| `product_name` | string | Line 2 (e.g. `Liquid Bait Station`). |
| `product_version` | string | Line 3 base (e.g. `v2.4`). |
| `preview` | boolean | If true, line 3 becomes `product_version` + ` Prototype`. **JSON only** — unrelated to OpenSCAD `mesh_preview`. |
| `exports` | array | STL build targets. |

Export scripts (not JSON) always pass **`-D mesh_preview=false`** and **`-D crosssection_view=false`** for STL output. In OpenSCAD source, use **`mesh_preview`** for fast interactive `$fn`, not the JSON field `preview`.

## `exports[]` entry

| Field | Type | Description |
|-------|------|-------------|
| `scad` | string | Input path relative to the **product** folder (e.g. `v2-clam-shell/reservoir.scad`). |
| `stl` | string | Output path relative to the **product** folder (e.g. `latest-build/reservoir.stl`). |
| `stamp` | object or null | If `{ "face": "bottom" }`, export script sets `{stem}_stamp_bottom = true` for that file. |

## Stem → OpenSCAD variable

- Take **basename** of `scad` (last path segment) without extension: `v2-clam-shell/reservoir.scad` → `reservoir`.
- Replace `-` with `_`: `.../needle-seal.scad` → `needle_seal`.
- Boolean flag name: `{stem}_stamp_bottom` (for `face: "bottom"` only today).

## Generated `stamp_generated.scad`

Assignments only (no modules):

- `info_stamp_line1`, `info_stamp_line2`, `info_stamp_line3` — escaped strings (brand, product, version [+ ` Prototype` if preview]).
- For every stem appearing in `exports`, `{stem}_stamp_bottom = true|false`.

File lives in the **product** folder; versioned `.scad` files **`include <../stamp_generated.scad>`**.

## Line composition (export script)

- **Line 1:** `brand_name`
- **Line 2:** `product_name`
- **Line 3:** `product_version` trimmed; if `preview`, append ` Prototype`.

Trim fields as needed; empty lines are skipped in the OpenSCAD deboss.
