# Valar Motion — Valar Systems motor controllers

Open-source firmware, hardware, and docs for the **Valar Motion** family of tiny
Wi-Fi stepper motor controllers. One board family, one repo: fix a bug once, ship
it to every board.

All three boards share a Trinamic **TMC2209** driver, a synchronous DC/DC
converter, three buttons, end-stop inputs, and a USB-C port for flashing. They
differ only in the microcontroller and (on the 3101) an onboard encoder.

| Board | MCU | Radios | Extra | Price |
|-------|-----|--------|-------|-------|
| **VAL3000** | ESP32-C3 | Wi-Fi | — | $34.99 |
| **VAL3100** | ESP32-C6 | Wi-Fi / Thread / Zigbee / Matter / BLE | Status LED | $44.99 |
| **VAL3101** | ESP32-C6 | Wi-Fi / Thread / Zigbee / Matter / BLE | Status LED + AS5600 encoder (closed-loop) | $54.99 |

Buy them at [valarsystems.com](https://valarsystems.com). Sold out? Hit the
"Notify me when available" button — that's how we know to make more.

## Repository layout

```
valar-motion/
├── boards/
│   ├── val3000/val3000.yaml   # ESP32-C3 pin map + substitutions
│   ├── val3100/val3100.yaml   # ESP32-C6 + status LED
│   └── val3101/val3101.yaml   # ESP32-C6 + status LED + AS5600 encoder
├── common/
│   └── valar-core.yaml        # shared, board-independent behaviour
├── hardware/                  # PCB, schematics, libraries (per board)
├── docs/                      # datasheets, guides, media
└── .github/workflows/         # CI: builds a factory + OTA bin per board
```

The important idea: **behaviour lives in `common/valar-core.yaml`**, which every
board pulls in as a package. Each `boards/<board>/<board>.yaml` is a thin wrapper
that only sets that board's chip, GPIO pin map, and OTA asset name. Change the
core once and all three boards inherit it.

## Flash a board (the fast way)

The board ships bare. Grab the prebuilt binary for your board from the
[latest release](../../releases/latest) and flash it from your browser at
[web.esphome.io](https://web.esphome.io) with a USB-C cable — no toolchain, no
soldering, no Home Assistant required. On first boot the board makes its own
Wi-Fi hotspot; connect, join it to your network, and control it from any
browser. It's discoverable in Home Assistant over the ESPHome API too.

## Build from source

```bash
python3 -m venv venv && source venv/bin/activate
pip install esphome
cp secrets.example.yaml secrets.yaml     # not needed for a bare flash

esphome compile boards/val3100/val3100.yaml
# → factory image at:
#   .esphome/build/val3100/.pioenvs/val3100/firmware.factory.bin
```

Swap `val3100` for `val3000` or `val3101`. CI (`.github/workflows/build-firmware.yml`)
builds all three on every push and attaches `*.factory.bin` / `*.ota.bin` to
tagged releases.

## Firmware updates (OTA)

Boards can update themselves over Wi-Fi — no computer needed — by fetching the
newest release asset for their board. **Release asset names are an API**: keep
`val3000.factory.bin`, `val3100.ota.bin`, etc. stable so units already in the
field keep updating after any repo move. See `docs/MIGRATION.md`.

## Applications

This repo is the **board firmware**. Products built on these boards live in their
own repos and include `common/valar-core.yaml` (or their own core) on top — e.g.
[The Ropener](https://github.com/Valar-Systems/Ropener), a smart curtain opener,
adds cover/position logic and homing on the same boards.

## License

See [LICENSE](LICENSE).
