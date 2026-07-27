# CPLD Firmware Build — CUPL toolchain (Mac + Linux server)

WinCUPL is Windows-only, so we compile on the Ubuntu server under Wine.
Applies to `PortableRAM8.pld` — **v3 8MB board, revision 02**. Do not use with v2 source files.

## Where it's set up (server 192.168.0.52)
- SSH user: `matthew`
- Wine prefix: `~/.wine-cupl` (WINEARCH=win32) with WinCUPL installed at `C:\Wincupl`
- Fitters upgraded to **v1918** (from Atmel ProChip 5.0.1) in `C:\Wincupl\WinCupl\Fitters`
- `5vcomp` wrapper (from github peterzieba/5Vpld) at `/usr/local/bin/5vcomp`

## To compile
```
scp firmware/PortableRAM8.pld matthew@192.168.0.52:~/PortableRAM8_v3.pld
ssh matthew@192.168.0.52 'WINEPREFIX=$HOME/.wine-cupl xvfb-run -a 5vcomp PortableRAM8_v3.pld'
scp matthew@192.168.0.52:~/PortableRAM8_v3.jed firmware/PortableRAM8.jed
scp matthew@192.168.0.52:~/PortableRAM8_v3.fit firmware/PortableRAM8.fit
scp matthew@192.168.0.52:~/PortableRAM8_v3.pin firmware/PortableRAM8.pin
```
Outputs: `.jed` (program this), `.fit` (CHECK pin assignments + "JTAG = ON"), `.pin`.

## .PLD source rules (CUPL is picky — these caused real failures)
1. **Pure ASCII only** — no em-dashes (—), smart quotes, etc. CUPL chokes on UTF-8.
2. **CRLF line endings** — LF-only makes the cuplx preprocessor abort.
3. **No `*/` inside comments** — e.g. write `UDS*, LDS*` NOT `UDS*/LDS*`; the `*/`
   closes the C-style comment early and the rest parses as code.
4. **`PROPERTY ATMEL { jtag = on };`** — keeps JTAG pins live so the Tigard can
   re-program. (Default is ON for the ispTQFP44 device, but be explicit.)

## JED -> SVF (DONE — cross-platform, on the Mac, no Wine)
Uses fuseconv from whitequark/prjbureau (pure Python):
```
pip3 install --user bitarray
git clone https://github.com/whitequark/prjbureau   # has util/fuseconv.py
cd prjbureau && python3 -m prjbureau.util.fuseconv ~/PortableRAM8_v3.jed ~/PortableRAM8_v3.svf
scp matthew@192.168.0.52:~/PortableRAM8_v3.svf firmware/PortableRAM8.svf
```
The .svf self-verifies (includes readback vectors). ATF1502AS: IR length 10,
IDCODE 0x0150203f.

## Programming (when boards arrive) — Tigard + OpenOCD, on the Mac
1. `brew install open-ocd`
2. Wire Tigard JTAG to J2: TDI=pin1, TMS=pin7, TCK=pin26, TDO=pin32 (+GND, +5V sense).
3. Power the board (5V).
4. Run the bundled script:  `./program.sh PortableRAM8.svf`
   (wraps: openocd -f interface/ftdi/tigard.cfg, jtag newtap -irlen 10
    -expected-id 0x0150203f, svf <file>)
