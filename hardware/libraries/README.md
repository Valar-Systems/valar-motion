# Shared KiCad libraries

Vendor symbols, footprints and 3D models used by the VAL3000 and VAL3100/3101 boards.
One folder per part: `<part>/<part>.kicad_sym`, `<part>.pretty/`, `<part>.3dshapes/`.

Every board project references this folder **relative to itself**, so a fresh clone
opens with all footprints and 3D models and nothing depends on a drive letter, a
Google Drive folder, or a per-machine environment variable:

- `hardware/<board>/PCB/fp-lib-table` and `sym-lib-table` point at `${KIPRJMOD}/../../libraries/…`
- every `(model …)` in the `.kicad_pcb` files is `${KIPRJMOD}/../../libraries/…`
- KiCad's own parts use `${KICAD10_3DMODEL_DIR}` (projects are on KiCad 10)

## Adding a part
Drop it in here in the same layout, add one line to both lib tables in each board
that uses it, and reference the model as `${KIPRJMOD}/../../libraries/<part>/<part>.3dshapes/<file>`.
Never point a model at Downloads, Google Drive, or an `${ENV_VAR}` only your PC has.

## Still resolved from KiCad's global tables (schematic only)
The schematics also use symbols from `SparkFun-PowerSymbol`, `SparkFun-Capacitor`,
`SparkFun-Resistor`, `00-kiml-power-protection`, and two auto-named import libs
(`2025-11-27_21-36-42`, `2025-11-30_16-50-14`). Those symbols are embedded in the
`.kicad_sch` files so the schematics open fine; the libraries are only needed to
*update* those symbols. Vendor them here if that ever matters.

## Notes
- `OSTVN04A150/footprints.pretty` carries the nickname `footprints` (matching the
  boards). `CONN_OSTVN02A150_OST` was recovered from the VAL3000 board file; it has
  no 3D model.
- AO3407A, AS5600L-ASOM_AMS and ESP32-C6-MINI-1-N4 had no 3D model on the VAL3100
  before; they now point at the vendor STEP with zero offset/rotation. Check them
  once in the 3D viewer and nudge if needed.
- The Google Drive folder `Products/Ropener/Hardware/libraries` is an archive now.
  Nothing reads from it.
