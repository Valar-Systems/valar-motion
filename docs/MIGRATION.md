# Migration checklist — three board repos → `valar-motion`

Consolidating `VAL3000`, `VAL3100`, and `VAL3101` into this single
`valar-motion` mono-repo. Work top to bottom; the ⚠️ items are the ones
that can break things for customers if skipped.

## Scope — what moves and what stays put

**Moves into `valar-motion` (then the old repo is archived):** the firmware,
PCB/hardware, and docs of the boards we sell standalone — VAL3000, VAL3100,
VAL3101.

**Stays in its own repo — do NOT pull into `valar-motion`:** the product repos
`Ropener` and `Glasscalibur`. Their wikis, 3D-print files, mechanical designs,
and product-specific PCB files belong with the product. They only *reference*
`valar-core` for shared firmware — nothing of theirs moves here. `valar-motion`
holds boards we sell + the shared core; it is not a home for product hardware.

## 1. Create the new repo
- [ ] Create `Valar-Systems/valar-motion` (this scaffold is the starting point).
- [ ] Push this scaffold to `main`.

## 2. Move real content in from the old repos
- [ ] Copy each board's **hardware/** (PCB, schematics, libraries) into
      `hardware/val3000/`, `hardware/val3100/`, `hardware/val3101/`.
- [ ] Copy **docs/** (datasheets, media, schematic PDFs) into `docs/`.
- [ ] Port any alternative firmware you want to keep (VAL3000 had `-api`,
      `-simple`, `-matter-arduino`, `-matter-esp-idf`) into
      `boards/<board>/variants/` or drop what's obsolete.
- [ ] ⚠️ Verify the VAL3101 **I2C pins** for the AS5600 in
      `boards/val3101/val3101.yaml` (placeholders are marked `TODO`).
- [ ] Confirm each board's YAML **compiles**: `esphome compile boards/<b>/<b>.yaml`.

## 3. Preserve OTA / release continuity  ⚠️
- [ ] Note the exact release **asset names** the shipped firmware fetches for
      self-update (from the Ropener firmware and any board OTA config).
- [ ] Make this repo's release assets use the **same names** (see the CI
      workflow) so units already in the field keep updating.
- [ ] If any device fetches from a hardcoded `github.com/Valar-Systems/VAL3xxx`
      release URL, update that URL in the firmware to point at
      `valar-motion` **and** ship that update before archiving the old repo.

## 4. Redirect the old repos (don't delete)  ⚠️
- [ ] In each of `VAL3000`, `VAL3100`, `VAL3101`: replace the README with a
      one-line "Moved to Valar-Systems/valar-motion" pointer.
- [ ] **Archive** them (Settings → Archive) rather than deleting — this keeps
      stars, issues, forks, and inbound links alive, and GitHub keeps the old
      URLs redirecting.
- [ ] Do **not** rename/transfer if you want the old URLs to survive as-is;
      archiving + a README pointer is the low-risk path.

## 5. Update the storefront links
- [ ] The Shopify listings link to `github.com/Valar-Systems/VAL3000` for
      "custom firmware." Point them at `valar-motion` once it's public.
      (Claude can do this in the Cowork session that made the listing edits.)
- [ ] Add a matching **"Valar Motion"** collection in Shopify if you want the
      store taxonomy to mirror the repo.

## 6. Nice-to-haves
- [ ] Single CHANGELOG at the repo root; tag releases `vX.Y.Z`.
- [ ] Issue templates that ask "which board?" up front.
- [ ] A board-comparison table in the README (already included).

## Why archive instead of delete or transfer
- **Delete** breaks every inbound link, bookmark, and any firmware pointing at a
  release URL — highest risk.
- **Transfer/rename** keeps a redirect, but the old name disappears and you now
  maintain a redirect you can't see; fine, but archiving is simpler.
- **Archive + README pointer** keeps history, stars, issues, and search presence
  while making the new home obvious — lowest risk, recommended.
