# VAL3101 hardware — same board as VAL3100

The VAL3101 is a **VAL3100 (ESP32-C6) with the AS5600 magnetic encoder
populated**. It is *not* a separate PCB, so there is no KiCad project in this
folder — the board source lives in [`../val3100/`](../val3100/).

On the VAL3100 layout the **AS5600 footprint and its I²C routing are already
present**; the encoder chip (and the dipole shaft magnet) are just not stuffed
on a plain VAL3100. The VAL3100 schematic even notes this directly:

> "Only the footprint has been added to the VAL3100, but the encoder has not.
> It can be easily soldered if required."

Populate the AS5600 + magnet and the same board becomes a VAL3101 with absolute,
power-loss-surviving closed-loop position feedback.

## Encoder I²C pins (verified from the VAL3100 schematic)

| Net | ESP32-C6 pin |
|-----|--------------|
| SDA | GPIO14 |
| SCL | GPIO15 |

These are wired in [`boards/val3101/val3101.yaml`](../../boards/val3101/val3101.yaml).
