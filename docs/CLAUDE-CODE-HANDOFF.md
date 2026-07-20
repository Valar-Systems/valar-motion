# Claude Code handoff — stand up `valar-motion` and migrate

This repo is a validated scaffold. Your job is to turn it into the live
`Valar-Systems/valar-motion` mono-repo, migrate real content from the three old
board repos, wire the Ropener to consume the shared core without copying it, and
retire the old repos safely. Read `docs/FIRMWARE-POLICY.md` and `docs/MIGRATION.md`
first — they define the invariants below.

## Hard invariants (do not violate)
1. **Never copy firmware between repos.** Shared board firmware lives once in
   `common/valar-core.yaml`; downstream (Ropener) references it as a remote
   ESPHome package. See `docs/examples/ropener-val3100.yaml`.
2. **OTA release-asset names are an API.** Field devices self-update by asset
   name. Preserve existing names (`Ropener-VAL3000.factory.bin`, etc.). Do not
   rename release assets without shipping a firmware update first.
3. **Archive, don't delete, the old repos.** Keep stars/issues/redirects.
4. All three board configs must still `esphome config` cleanly after any change.
5. **Only the standalone board repos (VAL3000/VAL3100/VAL3101) consolidate into
   `valar-motion`.** The product repos `Ropener` and `Glasscalibur` STAY as their
   own repos and keep their wikis, 3D-print files, mechanical designs, and
   product PCB files. Do NOT move product content into `valar-motion` — those
   repos only get rewired to reference `valar-core` remotely. `valar-motion` is
   boards-we-sell + the shared core, not a home for product hardware.

## Tasks
1. Create `Valar-Systems/valar-motion`, push this scaffold to `main`, tag `v0.1.0`.
2. Migrate from the old `VAL3000` / `VAL3100` / `VAL3101` repos:
   - `hardware/` (PCB, schematics, libraries) → `hardware/val3000|val3100|val3101/`
   - `docs/` (datasheets, media, schematic PDFs) → `docs/`
   - Keep any alternative VAL3000 firmware worth saving under
     `boards/val3000/variants/`; drop what's obsolete.
3. ⚠️ Fill in the real **AS5600 I2C pins** for the VAL3101 in
   `boards/val3101/val3101.yaml` (currently ESP32-C6 default placeholders,
   marked TODO) from the board schematic, then `esphome config` it.
4. Verify CI (`.github/workflows/build-firmware.yml`) builds a
   `factory.bin` + `ota.bin` for all three boards and attaches them on tags.
5. Refactor the **Ropener repo** (separate repo) to pull `valar-core` as a remote
   package instead of carrying its own board core:
   - Replace the board-level bits of `ropener-core.yaml` with the remote package
     reference (see `docs/examples/ropener-val3100.yaml`).
   - Keep only the Ropener-specific layer (cover, homing, gear math) in the
     Ropener repo.
   - Confirm each Ropener board config still compiles and that release asset
     names are unchanged.
6. **Glasscalibur repo** (separate product repo, window opener) — same treatment
   as Ropener. It's a finished product on its own PCB (NOT a Valar Motion catalog
   board — its board wrapper lives in the Glasscalibur repo), but it's the same
   ESP32 + TMC2209-over-UART stack, so it consumes `valar-core` as a remote
   package unchanged. See `docs/examples/glasscalibur.yaml`. Wire it up the same
   way; keep its OTA asset name (`Glasscalibur*`) stable.
7. Retire old board repos: replace each README with a one-line
   "Moved to Valar-Systems/valar-motion" pointer, then Archive.
8. Open a short PR/summary of what moved where.

Note on future boards: everything today is ESP32 + TMC2209-over-UART, so
`valar-core` is shared as-is. If a future board uses a different motor/driver/MCU,
split `valar-core` into a universal layer (Wi-Fi/API/OTA/Improv/Matter/UI) and a
stepper-specific layer before adding it — don't fork the whole core.

## Explicitly NOT your job (handled elsewhere)
- Shopify listing link updates and the "Valar Motion" collection are being done
  in the Cowork/Shopify session. Once `valar-motion` is public, that session will
  repoint the three product-page GitHub links from the old `VAL3000` repo to
  `valar-motion`. Don't touch storefront copy.

## Definition of done
- `valar-motion` public, scaffold + migrated hardware/docs pushed, `v0.1.0` tagged.
- CI green; all three boards compile; VAL3101 I2C pins real (not placeholder).
- Ropener repo references `valar-core` remotely; no duplicated board firmware;
  Ropener builds unchanged in behaviour and asset names.
- Old three repos archived with redirect READMEs.
