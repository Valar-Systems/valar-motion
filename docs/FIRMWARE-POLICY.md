# Firmware policy — where things live & what ships on what

The one rule that keeps this from turning back into three drifting copies:
**shared firmware exists once and is referenced, never copied.**

## The layers

There are two axes — the **platform** (boards we sell + the shared core) and the
**products** (finished devices built on that core). Products are siblings of each
other, each in its own repo, each referencing the core.

```
valar-core  (shared firmware — one source of truth)
   ├── Valar Motion catalog boards: VAL3000 / VAL3100 / VAL3101  (sold standalone)
   ├── Ropener        (product — curtain opener)
   └── Glasscalibur   (product — window opener; own PCB, not sold standalone)
```

| Layer | Repo | Owns | Depends on |
|-------|------|------|-----------|
| **Platform** | `valar-motion` | `common/valar-core.yaml` + hardware for boards we sell | — |
| **Shared firmware** | `valar-motion/common/valar-core.yaml` | Wi-Fi, Improv, Matter (planned), TMC2209, buttons, tuning | — |
| **Product: Ropener** | `Ropener` | curtain/cover layer, homing, gear math + mechanical | `valar-core` |
| **Product: Glasscalibur** | `Glasscalibur` | window-opener layer + its own PCB board wrapper | `valar-core` |

`valar-core` is the single source of truth for board behaviour. Every product
pulls it in — none keeps its own copy.

**Where a board's pin map lives:** catalog boards we sell (VAL3000/3100/3101)
live in `valar-motion/boards/`. A product-specific PCB we *don't* sell standalone
(e.g. Glasscalibur's board) lives in that product's repo as its own board
wrapper — it still references `valar-core`, it's just not a catalog board. All of
these are ESP32 + TMC2209-over-UART, so they share `valar-core` unchanged. If a
future board uses a different motor/driver/MCU, split `valar-core` first: a lower
universal layer (Wi-Fi/API/OTA/Improv/Matter/UI) plus a stepper-specific layer
that only TMC2209 boards include.

## How a product pulls in valar-core (no copying)

ESPHome supports remote packages: a config in one repo can `!include` YAML from
another repo by git ref. The Ropener's per-board configs use this instead of
duplicating the core. See `docs/examples/ropener-val3100.yaml` for a full example.

```yaml
packages:
  valar_core:
    url: https://github.com/Valar-Systems/valar-motion
    ref: v1.0.0                     # pin a tag — never float on main
    files: [common/valar-core.yaml]
    refresh: 1d
```

Pin `ref:` to a released tag so a change to `valar-core` never silently alters
the Ropener build. Bump it deliberately, rebuild, re-test, release.

## What firmware ships on which product

We flash **one image — the Ropener firmware — on every board we ship**, including
standalone VAL3000/VAL3100 boards. This is a deliberate operations choice: one
binary to build, flash, and QA instead of two. The Ropener firmware still works
as a general stepper controller (buttons + tuning controls are all there); the
cover entity is simply unused until a Ropener mechanism is attached.

| Product | Ships with | Customer can reflash to |
|---------|-----------|--------------------------|
| Ropener kit / assembled | Ropener firmware | — |
| Glasscalibur | Glasscalibur firmware | — |
| VAL3000 (standalone) | Ropener firmware (factory default) | generic `valar-core` build, or their own |
| VAL3100 (standalone) | Ropener firmware (factory default) | generic `valar-core` build, or their own |
| VAL3101 (standalone) | Ropener firmware (factory default) | generic `valar-core` build, or their own |

Because standalone boards ship with curtain firmware, the standalone listings
should say so plainly — e.g. *"ships pre-loaded with our Ropener firmware for
convenience; reflash the generic build anytime."* That keeps the "bare board,
flash what you want" promise honest while preserving the one-image workflow.

The **generic reference build** is just the board wrapper in this repo
(`boards/<board>/<board>.yaml` → `valar-core`), for customers who want a clean,
non-curtain firmware.

## Releases & OTA — asset names are an API

Devices self-update by fetching a release asset **by name**. Keep these names
stable across the repo move or field units stop updating:

- Generic builds (this repo's CI): `val3000.factory.bin`, `val3100.ota.bin`, …
- Ropener builds (Ropener repo's CI): `Ropener-VAL3000.factory.bin`, …

If any shipped firmware fetches from a hardcoded old repo URL, update that URL and
release the update **before** archiving the old repo. See `MIGRATION.md`.

## The golden rule, restated

If you ever find yourself copying a `.yaml` from one repo into another, stop —
add a remote package reference instead. Copies drift; references don't.
