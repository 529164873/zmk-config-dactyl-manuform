# ZMK Config — Dactyl Manuform 6×7 Wireless

A wireless, split [Dactyl Manuform](https://github.com/abstracthat/dactyl-manuform) keyboard (6 columns + 1 thumb column per side = 7 columns, 6 rows) running [ZMK firmware](https://zmk.dev).

---

## Hardware

| Part | Details |
|------|---------|
| Case | Dactyl Manuform 6×7 (3-D printed) |
| Controller (×2) | nice!nano v2 **or** SuperMini nRF52840 (pin-compatible clone) |
| Switches | Any MX-compatible switch |
| Diodes (×~80) | 1N4148, wired **COL2ROW** (anode on column wire, cathode on row wire) |
| Connection | Wireless BLE split; left half is the central (connects via USB or BLE to the host) |

Total key count: **~80 keys** (6 rows × 14 cols; row 5 only populates thumb positions at inner columns 4, 5, 8, 9).

---

## Matrix / Pin Assignment

### Left half

| Signal | Pins |
|--------|------|
| Rows 0–5 | P0.22, P0.24, P1.00, P0.11, P1.04, P1.06 |
| Cols 0–6 | P0.31, P0.29, P0.02, P1.15, P1.13, P1.11, P0.10 |

### Right half

Uses the same row and column pin numbers as the left half (both halves wired identically). The firmware applies a `col-offset = <7>` so ZMK maps the right-half columns to positions 7–13 in the unified 14-column matrix.

> **Note:** the right-half `col-gpios` in `dactyl_manuform_right.overlay` are currently marked *TODO: verify* — double-check them against your actual hand-wiring before flashing.

---

## Firmware

| Setting | Value |
|---------|-------|
| ZMK version | `v0.3` |
| BLE TX power | +8 dBm |
| Idle sleep timeout | 15 minutes (900 000 ms) |
| Battery reporting | Enabled (nice!nano voltage divider) |

### Building locally

```bash
# From a fully initialised ZMK workspace:
west build -p -b nice_nano_v2 -- -DSHIELD=dactyl_manuform_left
west build -p -b nice_nano_v2 -- -DSHIELD=dactyl_manuform_right
```

### Building via GitHub Actions (recommended)

Push (or fork) this repo — GitHub Actions will automatically compile both halves and a `settings_reset` firmware using the matrix defined in `build.yaml`. Download the `.uf2` artefacts from the **Actions** tab.

---

## Flashing

1. Double-tap the **reset** button on the nice!nano to enter bootloader mode — it appears as a USB mass-storage drive (`NICENANO`).
2. Copy the corresponding `.uf2` file to the drive:
   - `dactyl_manuform_left` → left half
   - `dactyl_manuform_right` → right half
3. The controller reboots automatically when the copy finishes.

**First flash / pairing issues?** Flash `settings_reset.uf2` to *both* halves, then re-flash the normal firmware. This clears stale bonding data.

---

## Keymap

Five layers are defined in `config/dactyl_manuform.keymap`.

| Layer | Index | How to activate |
|-------|-------|-----------------|
| Default (Dvorak) | 0 | Power-on default |
| Lower | 1 | Hold **RSE** key (right side of row 4) |
| Raise | 2 | Hold **LWR** key (right side of row 4, opposite side) |
| Adjust | 3 | Hold **Lower + Raise** simultaneously (tri-layer) |
| QWERTY | 4 | **Adjust** → `QWR` key; returns to Dvorak via `DVK` |

### Default layer (Dvorak)

```
GRV  1    2    3    4    5    6    ||  7    8    9    0    [    ]    BSPC
PgUp '    ,    .    P    Y    ⬅🖥  || 🖥➡  F    G    C    R    L    Home
PgDn A    O    E    U    I    LCtrl|| Menu D    H    T    N    S    End
\    ;    Q    J    K    X    LShft|| RCtrl B    M    W    V    Z    /
Caps Menu Up   Down Tab  Spc  RSE  || LWR  Spc  Ret  Left Rght -    =
                     GUI  ADJ      ||      Alt  Esc
```

`⬅🖥` / `🖥➡` = Ctrl+Win+Left/Right (previous / next virtual desktop on Windows)

### Lower layer — numpad + arrow island

Arrows (WASD-style) on the left; full numpad on the right. Everything else passes through to the default layer.

### Raise layer — function keys, media, shortcuts

F1–F12 across the top row; calculator, snipping tool (`Win+Shift+S`), Task Manager (`Ctrl+Shift+Esc`), and media controls (play/pause, stop, previous, next, volume up/down) scattered across both halves.

### Adjust layer — Bluetooth / output / firmware

| Key | Action |
|-----|--------|
| `BT_CLR` | Clear current BLE profile |
| `BT0`–`BT4` | Select BLE profile 0–4 |
| `USB` / `BLE` | Force USB or BLE output |
| `DVK` | Switch to Dvorak layer |
| `QWR` | Switch to QWERTY layer |
| `RESET` | Soft reset |
| `BOOT` | Enter UF2 bootloader |

### QWERTY layer

Alpha keys only (`Q W E R T …`); all modifiers, punctuation, and thumb keys fall through to the Dvorak layer beneath.

---

## Repository Layout

```
zmk-config-dactyl-manuform/
├── build.yaml                          # GitHub Actions build matrix
├── config/
│   ├── west.yml                        # ZMK v0.3 manifest
│   ├── dactyl_manuform.conf            # Firmware config (BLE, sleep, battery)
│   └── dactyl_manuform.keymap          # Keymap (edit this to customise)
└── boards/shields/dactyl_manuform/
    ├── dactyl_manuform.dtsi            # Shared matrix transform + kscan node
    ├── dactyl_manuform_left.overlay    # Left-half GPIO pin assignments
    ├── dactyl_manuform_right.overlay   # Right-half GPIO pin assignments
    ├── dactyl_manuform.zmk.yml         # Shield metadata
    ├── Kconfig.defconfig               # Sets keyboard name; marks left as central
    └── Kconfig.shield                  # Shield selection macros
```

---

## Customisation

- **Keymap** — edit `config/dactyl_manuform.keymap`. The [ZMK key codes reference](https://zmk.dev/docs/codes) lists everything available.
- **GPIO pins** — update the `row-gpios` / `col-gpios` entries in the `.overlay` files if your hand-wiring differs.
- **Sleep timeout** — change `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` in `config/dactyl_manuform.conf` (value is in milliseconds).
- **USB logging** — uncomment `CONFIG_ZMK_USB_LOGGING=y` in `config/dactyl_manuform.conf` to enable serial debug output.

---

## References

- [ZMK Documentation](https://zmk.dev/docs)
- [ZMK Keycodes](https://zmk.dev/docs/codes)
- [Dactyl Manuform case generator](https://github.com/abstracthat/dactyl-manuform)
- [nice!nano pinout](https://nicekeyboards.com/docs/nice-nano/pinout-schematic)
