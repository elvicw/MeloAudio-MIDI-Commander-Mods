# MeloAudio MIDI Commander (TSMIDI2.0) — Mods Hub

Community hub for **experimental modifications** to the MeloAudio MIDI Commander (USB ID `2EEE:0301`, product string `TSMIDI2.0`, STM32F1-class MCU).

> Unofficial and use at your own risk. Not affiliated with MeloAudio.
> **No vendor firmware is distributed here.** Mods are published as patch tools that work on *your own* copy of the official firmware and refuse to run on anything else (SHA-256 check).

Research behind these mods (reverse engineering of the original firmware) lives in the private/ongoing project repo; findings relevant to a mod are summarised in each mod's notes.

## Available mods

| Mod | Target | What it does | Status |
|---|---|---|---|
| [JAMP for Ampero II Stomp](docs/release-notes-v0.1.0-jamp-mod.md) (`v0.1.0-jamp-mod`) | built-in **JAMP** profile | Footswitch CC map for the Ampero II Stomp, EXP1/EXP2 = CC11, 5/E = patch up/down CCs | Hardware-tested by the author (keys, pedals); one cosmetic OLED flaw |

### JAMP → Ampero II Stomp: CC map
| Footswitch | CC | Value | Mode | Ampero II Stomp function |
|---|---|---|---|---|
| 1 / 2 / 3 | CC25 | 1 / 2 / 3 | Momentary | SCENE 1 / 2 / 3 |
| 4 | CC28 | 127/0 | Toggle | n/a |
| A | CC79 | 127/0 | Toggle | EFFECTS 1 |
| B | CC80 | 127/0 | Toggle | EFFECTS 2 |
| C | CC81 | 127/0 | Toggle | DLY/RVB |
| D | CC60 | 127/0 | Toggle | n/a |
| 5 | CC27 | 0 | Momentary | PATCH UP |
| E | CC26 | 0 | Momentary | PATCH DOWN |
| EXP1 / EXP2 | CC11 | pedal | continuous | expression |

Side effects: JAMP loses bank / program-change navigation; the "10x" bank-step menu option becomes stride 1 (labelled "01x"). Other presets are byte-identical to the original. Known flaw: noisy pixel block in the OLED CC# field (cosmetic; MIDI is correct) — see `docs/v0.1.0-jamp-mod-oled-flaw.jpg`.

## How to apply (JAMP mod)
1. Get the official firmware `TSMIDI_DFU_V3_STD2.dfu` from MeloAudio (DfuSe container, SHA-256 `a52597d0cceb665f1e476783ca788ef293d8b47bd069b414d5079cdfa222f944`).
2. Extract the app element: `python tools/dfuse_parse.py TSMIDI_DFU_V3_STD2.dfu out/` → the element loaded at `0x08003000` (77,451 bytes, SHA-256 `ff672fc4…4eee`).
3. `pip install capstone` then `python tools/patch_jamp_test.py out/<app element>.bin` → `TSMIDI_app_JAMPmod_TEST_0x08003000.bin` (SHA-256 `1476018a…202abe`).
4. Enter DFU: unplug, hold PA6 (E) + PA4 (D), plug in, release (USB ID `0483:df11`).
5. `dfu-util -d 0483:df11 -a 0 -s 0x08003000:leave -D TSMIDI_app_JAMPmod_TEST_0x08003000.bin` (only the app region; the bootloader at `0x08000000–0x08002FFF` is not touched).

**Recovery:** repeat steps 4–5 with the original app element. Keep a backup of the original first.

## Verification
The patch was verified by running the *original* code under a Unicorn ARM emulator (22 checks) and on hardware against an Ampero II Stomp. The emulator harness needs RAM state derived from the vendor firmware and is therefore not included here.

## Contributing
Open an issue or PR to add a mod. Rules for this hub: patch tools only (no vendor binaries), SHA-256-gated inputs, document every changed field, side effects and recovery, and say what was and was not tested on hardware.
