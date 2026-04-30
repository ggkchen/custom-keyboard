# Custom Keyboard

This is a keyboard shortened mainly to optimize mouse space for Valorant while keeping enough keys to everyday computer use. It uses an arduino micro pro as a microcontroller
and Kailh Box Thick Clicky switches. PCB is not hot swappable but should be simple to mount microcontroller, switches and keycaps onto it. The case has minimal parts for easy
lightweight assembly. Arrow keys have been replaced to serve as customizable macros or other special functions.

# PCB
<img width="1096" height="474" alt="Screenshot 2026-02-22 at 7 22 25 PM" src="https://github.com/user-attachments/assets/adc8cc7f-9087-4c8a-8eda-e573a9dc08f5" />

# CAD ASSEMBLY
<img width="920" height="479" alt="Screenshot 2026-02-22 at 11 21 22 PM" src="https://github.com/user-attachments/assets/23f21d3a-8871-4e47-b62a-6514b9cfcbdd" />
Onshape Link: https://cad.onshape.com/documents/7541f2100a52e2a8fb9d96d0/w/174a04ea7bc7f7ecaec00ce4/e/b31f0979a24da6a144661240?renderMode=0&uiState=699c00031957867eacc0531d

# BOM


|Name| Qty | Manufacturer | Cost | Description| URL| Total Cost
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Navy-CPG1511F01S09  | 110 | Kailh | 36 | Keyboard Switches | https://www.kailh.net/products/kailh-box-thick-clicky-switch-set?variant=43650830369010 | 71.23
|1N4148FS-ND	 | 70  | Digikey | 2.73 | Diodes for switches | https://www.digikey.com/en/products/detail/onsemi/1N4148/458603	 |
|Arduino Micro Pro	 | 1  | aliexpress | 5.33 | Arduino Microcontroller | https://www.aliexpress.com/ssr/300000512/BundleDeals2?spm=a2g0o.productlist.main.11.2c78f612vaY1aB&productIds=1005006322199481:12000036755772596&pha_manifest=ssr&_immersiveMode=true&disableNav=YES&sourceName=SEARCHProduct&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005006322199481%7C_p_origin_prod%3A&pvid=08416216-fbe8-4685-8c26-a513ac16dcbe	 |
|Keyboard PCB	 | 5 | JLCPCB | 27.17 | Keyboard PCB |

# Build Guide

## What you'll need

**Parts** (see BOM above for the full list)
- 1× PCB (order from JLCPCB using `KiCad/KiCad_Source_File/Keyboard Gerbers.zip`)
- 1× Pro Micro (ATmega32U4)
- 64× Kailh Box Thick Clicky switches
- 64× 1N4148 through-hole diodes
- 64× keycaps (ANSI 60% set with the layout this PCB uses)
- 1× printed/CNC case (`CAD/KEEBASSEMBLY.stl` or `CAD/Keyboard_Assembly.step`)
- USB-C or Micro-USB cable matching your Pro Micro
- M2 screws/standoffs sized to your case print

**Tools**
- Soldering iron (~330 °C for leaded, ~360 °C for lead-free) and solder
- Flush cutters
- Tweezers
- Multimeter (continuity mode)
- Masking tape
- Hot glue or kapton tape (for reinforcing the Pro Micro USB port — optional but recommended)

## Step 1 — Order the PCB

1. Zip up or grab the prebuilt gerbers at `KiCad/KiCad_Source_File/Keyboard Gerbers.zip`.
2. Upload to JLCPCB (or any board house). The default settings (1.6 mm FR-4, HASL, any color) work fine.
3. While you wait, print the case from `CAD/KEEBASSEMBLY.stl`.

## Step 2 — Solder the diodes

The matrix is wired **column-to-row (COL2ROW)** — the diode's cathode (the end with the black band) goes toward the row trace. Every PCB footprint has a square pad and a round pad; the **black band lines up with the square pad** on this board.

1. Bend each 1N4148 into a U-shape and drop it into its footprint. Get all 64 placed before you solder anything.
2. Tape a piece of cardboard or a flat surface over the back, flip the PCB, and let gravity hold the diodes flush.
3. Solder one leg of each diode, then go back and solder the other leg.
4. Trim leads flush with flush cutters.

Sanity check before moving on: pick any diode and put your multimeter in continuity/diode mode. You should read ~0.6 V in one direction and open in the other.

## Step 3 — Solder the Pro Micro

Do this **before** the switches — once the switches are on, you can't reach the Pro Micro pads.

1. Solder header pins or wires into the Pro Micro pads on the PCB. Many builders solder the Pro Micro directly to the board for a lower profile; sockets let you swap it later but add height.
2. Place the Pro Micro **components facing down** (toward the PCB) so the USB port lines up with the case cutout. Double-check against the case STL before soldering.
3. Solder all pins. A drop of hot glue between the USB connector and the Pro Micro PCB will save you the day the cable gets yanked.

## Step 4 — Mount and solder the switches

1. Drop the top plate (if your case uses one) over the PCB.
2. Push 4–5 switches into the **corners and center** of the plate first, making sure the pins go through cleanly without bending. These anchor the plate.
3. Flip the assembly and solder the corner switches. Inspect that the plate is sitting flush on the PCB before continuing — if it's tilted, reheat and adjust now.
4. Fill in the rest of the switches and solder both pins of each one.

Tip: hold each switch firmly against the plate while you solder its first pin, otherwise it can sag and end up uneven once keycaps are on.

## Step 5 — Final assembly

1. Press keycaps onto the switches.
2. Drop the PCB+plate stack into the case bottom and screw it down.
3. Plug in USB. Don't worry that nothing types yet — that's firmware.

# Firmware

The keyboard runs **QMK**. Source files live in `Keyboard_firmware/`:
- `keyboard.json` — board definition (ProMicro, COL2ROW matrix, 64-key ANSI layout)
- `keymap.c` — three layers: base, MO(1) function, MO(2) reset

## Step 1 — Set up QMK

```bash
python3 -m pip install --user qmk
qmk setup
```

This clones `qmk_firmware` to `~/qmk_firmware`. Accept the default when it asks.

## Step 2 — Drop in the keyboard files

From the root of this repo:

```bash
mkdir -p ~/qmk_firmware/keyboards/keeb/keymaps/default
cp Keyboard_firmware/keyboard.json ~/qmk_firmware/keyboards/keeb/keyboard.json
cp Keyboard_firmware/keymap.c       ~/qmk_firmware/keyboards/keeb/keymaps/default/keymap.c
```

## Step 3 — Compile

```bash
qmk compile -kb keeb -km default
```

You should get a `keeb_default.hex` in the QMK root. If the compile fails complaining about `LAYOUT_64_ansi`, double-check that `keyboard.json` copied over cleanly.

## Step 4 — Flash

1. Plug the keyboard in.
2. Run:
   ```bash
   qmk flash -kb keeb -km default
   ```
3. When QMK prints `Detecting USB port, reset your controller now...`, **short the `RST` and `GND` pins on the Pro Micro twice quickly** (or press the reset button if you wired one). The bootloader window is short — about 8 seconds.
4. QMK will detect the bootloader, flash, and the board will reboot.

If you ever flash bad firmware and lose access to the reset key, the layer-2 `QK_BOOT` binding (`MO(1)` + `MO(2)` + `R`) drops the board into the bootloader from any working firmware.

## Layer reference

| Layer | How to reach it | What it gives you |
| --- | --- | --- |
| 0 (base) | default | ANSI typing layer |
| 1 (fn) | hold the `MO(1)` key (top-right next to ↑) | F-row, PrtSc/ScrLk/Pause, Home/PgUp/PgDn/End on the arrow cluster |
| 2 (reset) | hold `MO(1)` + `MO(2)` (right Ctrl on layer 1) | `R` enters QMK bootloader for re-flashing |

To customize, edit `Keyboard_firmware/keymap.c` and re-run steps 2–4.

# Testing

Do this **before** you put keycaps on and screw the case shut — fixing a cold joint is easy now and annoying later.

## 1 — Smoke test the Pro Micro

Plug in USB. The Pro Micro's power LED should light up. If not, unplug immediately and check for solder bridges between adjacent Pro Micro pads.

## 2 — Confirm the OS sees the keyboard

- **Windows:** Device Manager → Keyboards → there should be a new HID Keyboard Device. The QMK USB descriptor in `keyboard.json` uses VID `0xFEED` / PID `0x0000`.
- **macOS:** the keyboard setup assistant pops up asking you to identify the layout — cancel out, that's enough.
- **Linux:** `lsusb` should list a `feed:0000` device.

## 3 — Test every key

Open https://config.qmk.fm/#/test or https://keyboard-tester.com/ and press every key on the base layer. Each press should light up exactly one entry on screen.

Common failures and what they mean:

| Symptom | Likely cause |
| --- | --- |
| One key does nothing | Cold joint on that switch — reflow both pins |
| One key registers a different letter | Bridged solder between two switch pads, or a diode in backwards on the wrong key |
| Whole row dead | Cold joint on the row trace at the Pro Micro, or a damaged diode somewhere on that row |
| Whole column dead | Same, but on the column side |
| Pressing one key triggers two | Diode missing or installed backwards (no isolation, so the matrix scan ghosts) |
| Nothing types at all | Firmware didn't flash, or matrix pins in `keyboard.json` don't match the PCB |

## 4 — Test the function layer

Hold the `MO(1)` key (top-right, next to ↑) and confirm the F-row registers as F1–F12.

## 5 — Test bootloader access

Hold `MO(1)` + `MO(2)` + `R`. The keyboard should disappear from the OS and re-enumerate as the Pro Micro bootloader (`Arduino Leonardo` or `ATmega32U4`). Unplug and replug to return to normal — confirming this works means you can always recover from a bad keymap without opening the case.

Once all five tests pass, screw the case together and you're done.
