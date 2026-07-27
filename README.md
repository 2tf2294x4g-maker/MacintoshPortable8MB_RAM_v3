# Mac Portable 8MB RAM Card (v3 — All-5V)

Expands a Macintosh Portable to **9 MB total** (1 MB onboard + 8 MB this card) — the maximum the Portable's address space supports.

All-5V design using 1M×8 SRAM chips and an ATF1502ASL CPLD. Works with both the **M5120** (non-backlit) and **M5126** (backlit) Portable via the MODEL jumper. Banks are added sequentially — no jumpers needed as you populate.

> ⚠️ **PRE-PRODUCTION — NOT TESTED**
> This board has been designed and ordered but has not yet been assembled or validated on real hardware. Schematics, firmware, and Gerbers are provided as-is. Do not build for production use until hardware validation is complete.

---

## What's new in v3 vs v2

| | v2 | v3 |
|---|---|---|
| Model compatibility | M5120 only | M5120 + M5126 (MODEL jumper) |
| Bank indicator | none | 4 green latching LEDs (one per bank) |
| Ghost RAM protection | none | 10 KΩ pull-downs on SD0–SD15 |
| Population jumper | SIZE jumper (JP1) | none — sequential by design |
| DTACK output | bodge wire to CPLD pad | J3 header (pin 1 = /DTACK, pin 2 = /EXT_DTACK) |
| SRAM count | 8× AS6C8008 | 8× AS6C8008 (same chips) |

---

## Specs

| | |
|---|---|
| Capacity | 8 MB (8× 1M×8 SRAM) |
| System total | 9 MB M5120 (open) / 7 MB M5126 (closed) |
| Bus width | 16-bit (two 8-bit SRAMs per bank) |
| Banks | 4 × 2 MB, populated sequentially |
| Logic | ATF1502ASL CPLD (5V, TQFP-44) |
| SRAM | 8× AS6C8008-55ZIN (5V, TSOP-II-44) |
| Layers | 4 (F.Cu signal / GND plane / +5V plane / B.Cu signal) |
| Connector | Samtec BCS-125-F-D-HE (50-pin 2×25, horizontal entry) |
| MODEL jumper | JP1: OPEN = M5120 non-backlit (9 MB) / CLOSED = M5126 backlit (7 MB) |

---

## Bill of Materials

| Ref | Qty | Value | Package | Part / Notes |
|-----|----:|-------|---------|--------------|
| U1–U8 | 8 | AS6C8008-55ZIN | TSOP-II-44 | Alliance Memory — 1M×8 5V SRAM 55 ns. Order spares. |
| U9 | 1 | ATF1502ASL | TQFP-44 | Microchip ATF1502ASL-**25AU44** — must be **ASL** (5V), not ASV |
| U10, U11 | 2 | 74HC245 | SOIC-20 **wide** | SN74HC245**DW** — verify wide (7.5 mm) body |
| J1 | 1 | RAM connector | Samtec BCS-125-F-D-HE | 50-pin socket, horizontal entry |
| J2 | 1 | JTAG header | 1×6 2.54 mm pin header | GND/VCC/TDO/TDI/TMS/TCK left-to-right |
| J3 | 1 | PDS DTACK header | 1×2 2.54 mm pin header | Pin 1 = /DTACK → PDS B7, Pin 2 = /EXT_DTACK → PDS B30 |
| JP1 | 1 | MODEL jumper | 1×2 2.54 mm pin header + shunt | OPEN = M5120 / CLOSED = M5126 |
| D1–D4 | 4 | Green LED | 0603 | Latch ON at boot when each bank is confirmed by the CPLD |
| R1–R4 | 4 | 330 Ω | 0402 | Series resistors for D1–D4 |
| R5–R20 | 16 | 10 KΩ | 0402 | Pull-downs on SD0–SD15 (SRAM side of 74HC245) — prevent ghost RAM |
| R21–R24 | 4 | 10 KΩ | 0805 | Pull-ups on CPLD unused inputs and MODEL jumper pin |
| C1–C14 | 14 | 100 nF | 0805 | X7R 50V ceramic bypass (one per IC) |
| C15–C19 | 5 | 10 µF | 0805 | X5R 16V ceramic bulk |

Full BOM: [PortableRAM-8MB-v3-BOM.csv](PortableRAM-8MB-v3-BOM.csv)

---

## ⚠️ Important — SRAM Pinout

The AS6C8008 (1M×8) and AS7C4096A (512K×8 used in the 4MB card) are **both TSOP-II-44 but have different pinouts**. Do not mix them between boards.

---

## Population Guide

Banks are populated sequentially — each pair of SRAMs adds 2 MB. No jumpers required.

| Step | Chips | Total RAM |
|------|-------|-----------|
| 1 | U1 + U5 | 2 MB |
| 2 | U2 + U6 | 4 MB |
| 3 | U3 + U7 | 6 MB |
| 4 | U4 + U8 | 8 MB |

The green LED for each bank (D1–D4) latches on at boot when the CPLD confirms that bank is active. A dark LED means the bank is not yet populated.

---

## MODEL Jumper (JP1)

| JP1 | Portable model | System RAM |
|-----|---------------|-----------|
| Open (no shunt) | M5120 non-backlit | 9 MB |
| Closed (shunt installed) | M5126 backlit | 7 MB |

The M5126 uses slot CS0 for its backlight controller, which conflicts with the top 2 MB bank. Closing JP1 disables that bank so the two don't fight. Leave open for M5120.

---

## Files

```
PortableRAM-8MB.kicad_pcb     — PCB layout (KiCad 9)
PortableRAM-8MB.kicad_sch     — Schematic
PortableRAM-8MB.kicad_pro     — Project file
PortableRAM.kicad_sym         — Custom symbol library
PortableRAM.pretty/           — Custom footprint library
Library.pretty/               — Additional footprints
Gerbers/                      — Gerber + drill files (ready to order)
PortableRAM-8MB-v3-Gerbers.zip — Same, zipped (upload directly to fab)
firmware/
  PortableRAM8_v3.pld            — CUPL source (ATF1502ASL logic)
  PortableRAM8_v3.jed            — Compiled JEDEC fusemap
  PortableRAM8_v3.svf            — SVF for OpenOCD programming
  PortableRAM8_v3.fit            — WinCUPL fit report
  program.sh                  — One-command programming script (Tigard)
  BUILD.md                    — Firmware build instructions (WinCUPL + Wine)
PortableRAM-8MB-v3-BOM.csv
```

---

## Ordering the PCB

Upload `PortableRAM-8MB-v3-Gerbers.zip` to your preferred fab (JLCPCB, PCBWay, OSHPark, etc.).

**Board settings:**
- Layers: **4**
- Thickness: 1.6 mm
- Surface finish: **ENIG** (recommended — fine-pitch 0.8 mm pads on TSOP-II-44 and TQFP-44)
- All other settings: fab defaults

---

## Assembly Notes

Solder in this order:

1. **U9** (ATF1502ASL CPLD, TQFP-44) — center first, then drag-solder
2. **U1–U8** (SRAM, TSOP-II-44) — fine pitch; flux generously. Populate in bank pairs: U1+U5 → U2+U6 → U3+U7 → U4+U8
3. **U10, U11** (74HC245, SOIC-20 wide)
4. **R5–R20** (10 KΩ 0402 pull-downs)
5. **R1–R4** (330 Ω 0402 LED resistors)
6. **R21–R24** (10 KΩ 0805 pull-ups)
7. **C1–C14** (100 nF 0805 bypass caps)
8. **C15–C19** (10 µF 0805 bulk caps)
9. **D1–D4** (green 0603 LEDs — observe polarity)
10. **JP1** (1×2 header — add shunt for M5126, leave open for M5120)
11. **J2** (1×6 JTAG header)
12. **J3** (1×2 PDS DTACK header — on board edge)
13. **J1** (Samtec connector) — last

---

## Programming the CPLD

The ATF1502ASL must be programmed before the card will work. Use J2 (1×6 header on the board edge).

### J2 pinout (left to right, component side)

| Pin | Signal |
|-----|--------|
| 1 | GND |
| 2 | VCC |
| 3 | TDO |
| 4 | TDI |
| 5 | TMS |
| 6 | TCK |

### With a Tigard programmer (FT2232H-based)

Set the Tigard voltage jumper to **5V** before connecting.

```bash
cd firmware
./program.sh PortableRAM8_v3.svf
```

Success output ends with `shutdown command invoked` and no TDO mismatch errors.

See [firmware/BUILD.md](firmware/BUILD.md) for the full build and programming pipeline (WinCUPL + Wine on Linux, prjbureau for SVF generation).

---

## Installation

1. Power off and unplug the Mac Portable.
2. Remove the battery and bottom cover.
3. Set JP1 (MODEL jumper) for your machine — open for M5120, closed for M5126.
4. Seat the card on the RAM expansion header — J1 is a friction-fit horizontal-entry socket.
5. Reassemble and power on.

With a full build (all 8 SRAMs), you should see **9 MB** in About This Macintosh (M5120) or **7 MB** (M5126). All four green LEDs D1–D4 will be lit.

### DTACK Performance Fix — J3 header

The Mac Portable's GLU chip generates /DTACK for the 68000 with region-dependent timing:

| Address range | GLU /DTACK | Condition |
|---|---|---|
| 0x000000–0x4FFFFF | 2 clocks | Always — hardwired fast path |
| 0x500000–0x8FFFFF | 6 clocks | Normal operation (register configured) |
| 0x500000–0x8FFFFF | 18 clocks | After sleep/wake (register reset) |

The 8 MB card spans 0x100000–0x8FFFFF, straddling the split. Banks 0–1 (0x100000–0x4FFFFF) already get the GLU's 2-clock fast path. Banks 2–3 (0x500000–0x8FFFFF) run at 6 clocks and collapse to 18 clocks after sleep/wake when the GLU's DTACK register loses its configuration.

The fix is built into v3: the CPLD asserts /DTACK directly via **J3 pin 1**, bypassing the GLU for all card accesses. Connect a wire from J3 pin 1 to **PDS slot pin B7** (/DTACK) on the Mac Portable motherboard. The CPLD's path is purely combinatorial — faster than even the GLU's 2-clock mode — and immune to sleep/wake state loss.

J3 pin 2 (/EXT_DTACK → PDS B30) is an optional second output for experimentation.

**Validated on v2 hardware (July 2026):**

| Configuration | Speedometer | Snooper | Post-sleep |
|---|---|---|---|
| 1 MB card (baseline) | 1.990 | 87% | 1.985 |
| 8 MB card — no DTACK fix | 1.486 | 51% | 0.974 / 22% |
| **8 MB card + CPLD DTACK** | **2.170** | **102%** | **2.170** |

The CPLD outperforms the GLU's fastest mode (102% vs 87% baseline) because its combinatorial propagation delay is well under one bus clock. Post-sleep collapse is fully eliminated. The fix also has no effect on a 4 MB configuration since all banks fall in the GLU's 2-clock region and do not suffer the post-sleep collapse.

The PDS connector is the 96-pin DIN-41612 on the Mac Portable motherboard (3 rows A/B/C, 32 pins each). Pin B7 is in row B, position 7 from the component side.

![PDS slot and RAM connector locations, with /DTACK (B7) marked](images/pds-slot-dtack-location.png)

---

## References & Acknowledgements

**Dynamic Engineering Mac Portable RAM Card**
The address decode and bus control logic in `PortableRAM8_v3.pld` is derived from GAL22V10 PLD source files originally written for the Dynamic Engineering commercial RAM expansion card. These files define the core signal equations — bank chip-select generation, byte-lane write control via UDS*/LDS*, bus transceiver direction, and output enable timing. The original `.PLD` files were recovered and shared by *techknight* on the TinkerDifferent forum and #skunkworks Discord. Translated here from three GAL22V10 chips to a single ATF1502ASL CPLD, with corrections to the `r_w_u` equation and address decode base, and extended for 8MB capacity and M5126 compatibility.

**miejas — Macintosh Portable 4MB Memory Expansion**
[https://github.com/miejas/Macintosh-Portable-4-MB-Memory-Expansion](https://github.com/miejas/Macintosh-Portable-4-MB-Memory-Expansion)
Open-source 4MB RAM card used as a cross-reference for connector pinout, bus transceiver approach, and firmware structure.

**Reza Fouladian — PortableRAM BGA 8MB**
[https://github.com/rezafouladian/PortableRAM-BGA-8MB](https://github.com/rezafouladian/PortableRAM-BGA-8MB)
BGA-format 8MB card using the ATF1502ASV (3.3V CPLD) and SN74LVC4245A level translators. Reference for the 8MB address decode strategy and CPLD pin planning. Reza also provided direct guidance on driving /DTACK from the CPLD and on the Mac Portable PDS connector pinout — specifically confirming that /DTACK is available on PDS pin B7 and can be driven directly from a CPLD output. This insight is the basis for the J3 DTACK header on v3 and the validated bodge fix on v2.

**Apple Mac Portable Developer Notes & Schematics**
Apple's original hardware documentation for the Mac Portable expansion bus, memory map, and GLU register behaviour (including the $FC0200 DTACK register).

---

## License

Hardware released under [CERN-OHL-S v2](https://ohwr.org/cern_ohl_s_v2.txt). Firmware source (`PortableRAM8_v3.pld`) released under MIT.
