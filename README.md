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

Heads up before you start: this whole build took me a weekend, but most of that was waiting for parts. Actual hands-on time is maybe 4–5 hours if you're slow with a soldering iron like me.

## Stuff you need

Parts are all in the BOM up there. Beyond that you'll want:

- soldering iron + solder (any decent iron works, I used a TS100)
- flush cutters for trimming diode legs
- tweezers — really helpful for placing diodes, get a pair if you don't have one
- a multimeter (optional but you'll regret skipping it if something goes wrong)
- some masking tape
- a USB cable for the Pro Micro

You'll also need a printed case. STL is in `CAD/KEEBASSEMBLY.stl`, or grab the STEP if you want to tweak it. I printed mine in PETG at 0.2 layer height and it was fine.

## 1. Get the PCB made

The gerbers are sitting in `KiCad/KiCad_Source_File/Keyboard Gerbers.zip`. Just upload that zip to JLCPCB, leave everything on default (1.6mm, HASL, whatever color you want — black hides flux residue, white shows off the diodes), and order 5. They're cheap and you'll probably want spares.

While that's shipping, kick off the case print so they arrive around the same time.

## 2. Solder the diodes

Easy step but boring — there are 64 of them. The diodes are directional, so pay attention here: the black band on the diode lines up with the **square pad** on the PCB. Get this wrong and that key won't register, or worse, will ghost when you press other keys.

What worked for me:
- bend all 64 diodes into little staples first, in a batch
- drop them all into the board before soldering anything (saves you from picking up the iron 64 separate times)
- tape something flat over the back, flip the board, and the diodes will sit flush from gravity
- solder one leg of each, do a pass for alignment, then solder the other legs
- snip the leads when you're done

If you have a multimeter, hit a couple of random diodes in continuity mode just to make sure you didn't put any in backwards. Way easier to fix now than after the switches are on.

## 3. Solder the Pro Micro

Do the Pro Micro **before** the switches. I cannot stress this enough — once switches are on you can't get to those pads without desoldering half the board.

You can either solder the Pro Micro directly to the PCB (lower profile, permanent) or use machined headers + sockets so you can swap it later. I went direct because I'm cheap. Components face down, toward the PCB, with the USB port lined up with the case cutout.

One pro tip: put a dab of hot glue or some kapton over the USB connector after soldering. The Pro Micro USB port is famous for ripping off the board the first time someone trips on the cable.

## 4. Switches

Drop the plate (if your case has one — mine doesn't, switches mount straight to the PCB) over the board, push 4 switches into the corners, and one in the middle. These hold everything in place.

Flip it, solder just those 5, then check that the plate is sitting flat. If it's tilted you want to know now, not after you've soldered all 64. Reflow whichever corner is high and push it down.

Once it's flat, fill in the rest. Both pins on each switch.

## 5. Put it together

Keycaps push on, PCB drops into the case, screws in. Plug it in. Don't panic when it doesn't type — we still have to flash firmware.

# Firmware

Runs QMK. The `keyboard.json` and `keymap.c` are in `Keyboard_firmware/`. There are three layers:

- base — normal typing
- fn (hold the key in the bottom right corner where you'd expect right-arrow) — gives you F1–F12, PrtSc, Pause, and the missing arrow nav keys
- reset (fn + the second mod key) — hitting `R` on this layer puts the board into the bootloader, so you can re-flash without opening the case

## Setting up QMK

If you've never used QMK before:

```
python3 -m pip install --user qmk
qmk setup
```

Hit yes when it asks to clone `qmk_firmware` to your home directory.

## Copy the files in

QMK expects keyboards to live under `qmk_firmware/keyboards/`, so:

```
mkdir -p ~/qmk_firmware/keyboards/keeb/keymaps/default
cp Keyboard_firmware/keyboard.json ~/qmk_firmware/keyboards/keeb/keyboard.json
cp Keyboard_firmware/keymap.c       ~/qmk_firmware/keyboards/keeb/keymaps/default/keymap.c
```

## Compile

```
qmk compile -kb keeb -km default
```

Should drop a `.hex` in the QMK root after a few seconds. If it complains about `LAYOUT_64_ansi` not existing, your `keyboard.json` didn't make it over — re-check the copy.

## Flash

Plug the keyboard in, then:

```
qmk flash -kb keeb -km default
```

When it says `Detecting USB port, reset your controller now...`, you've got about 8 seconds to short the `RST` pin to `GND` on the Pro Micro twice in quick succession. Tweezers work. The flasher will grab it and write the firmware.

If you ever brick the keymap and lose access to your reset key, the layer-2 bootloader binding saves you — fn + mod + `R`. So even if you mess up the keymap badly, you can always get back into the bootloader without cracking the case open.

To customize anything — different keys, macros on the arrow positions, whatever — edit `keymap.c` and rerun the compile + flash steps.

# Testing

Don't put the keycaps on yet. Seriously. Reflowing one cold joint with the keycaps off takes 30 seconds; with caps on it takes ten minutes and a lot of swearing.

**1. Plug it in.** The little power LED on the Pro Micro should come on. If it doesn't, unplug right away and look for solder bridges between adjacent pins on the Pro Micro.

**2. Check that the OS sees it.** On Windows it'll show up under Keyboards in Device Manager. On Mac you'll get the keyboard identification wizard popping up — just close it. On Linux, `lsusb` will show a `feed:0000` device. (That's the default VID/PID baked into the firmware, no, it's not malware.)

**3. Hit every key.** https://config.qmk.fm/#/test is what I use. Press each key, check it lights up on screen, move on.

If something's wrong, it's almost always one of these:
- one key does nothing → cold joint, reflow both pins
- one key types something else → either a solder bridge between two switches, or a diode flipped on a different key (matrix scanning makes this confusing)
- a whole row or column is dead → cold joint at the Pro Micro pin for that row/col, or a totally fried diode somewhere on the row
- pressing one key triggers two → diode missing or backwards on one of them (the matrix can't isolate without it)
- nothing at all → firmware didn't flash, or you're holding a perfectly good paperweight

**4. Test the fn layer.** Hold the fn key (bottom-right area, next to up-arrow) and tap the number row — should give you F1 through F12.

**5. Test the bootloader combo.** fn + the second mod + `R`. The keyboard should disappear from your machine and reappear as `Arduino Leonardo` or similar. Unplug and replug and you're back to normal. Once you've confirmed this works, you can flash new firmware forever without touching the reset pins again.

That's it. Keycaps on, screw the case shut, go play Valorant.
