# Hardware

PCB source, schematics, and libraries for each board we sell, one folder per board.

    hardware/val3000/   # ESP32-C3 board — KiCad source + libraries (from the old VAL3000 repo)
    hardware/val3100/   # ESP32-C6 board — KiCad source + libraries (from the old VAL3100 repo)
    hardware/val3101/   # ESP32-C6 + AS5600 — SAME PCB as val3100 (see below)

Board-specific documentation (schematic PDFs, datasheets, media) lives under
`docs/<board>/` — e.g. `docs/val3000/`.

## VAL3101 shares the VAL3100 board

The **VAL3101 is not a separate PCB** — it is the VAL3100 board with the AS5600
magnetic encoder **populated**. The VAL3100 layout already carries the AS5600
footprint and its I²C routing; on a plain VAL3100 the chip is simply left
unstuffed. Populate it (plus the shaft magnet) and it becomes a VAL3101.

So `hardware/val3101/` intentionally has no separate KiCad project — use
`hardware/val3100/`. The encoder's I²C pins, read from that schematic, are
**SDA = GPIO14, SCL = GPIO15** (see `boards/val3101/val3101.yaml`).

## What was migrated

Only real design source was brought over: `.kicad_pcb`, `.kicad_sch`,
`.kicad_pro`, vendor `Libraries/`, and fabrication config. KiCad auto-save
`*-backups/` archives, project-local `.kicad_prl` state, lock files, and caches
were intentionally left out (and are `.gitignore`d) — they remain in the
original board repos, which are archived rather than deleted.
