# MiceyMousey

everything from the wolfiemouse repo (kbumsik's IEEE region 1 micromouse, GPLv2.1 — https://github.com/kbumsik/WolfieMouse), restructured like pokeyboardy.

## layout

- `MiceyMousey.kicad_pro` / `.kicad_sch` / `.kicad_pcb` — the live kicad project (open the .kicad_pro)
- `lib/symbols/` — 26 custom symbols converted to modern .kicad_sym (legacy format from 2017 won't load in kicad 10, hence conversion)
- `lib/footprints_micromouse.pretty/` — 27 custom footprints, converted to modern format
- `reference/` — wolfie's original 2017 schematic files (legacy format, kicad 10 can't open em — read-only reference)
- `production/` — the original CAM/gerber zips (fab reference)
- `3D_Model/` — wheels, skate, motor seat, sensor housing stls
- `firmware/` — original c++ firmware (freeRTOS + own maze solver)
- `simulation/` — algorithm simulator
- `tools/` — vagrant env, sensor data tools
- `docs/` — original documentation (read these, they're gold)
- `BOM.xlsx` — the shopping list

## notes

- the original `.kicad_pcb` still loads fine in kicad 10 (zones convert best-effort) — but if u edit it, save-as creates the modern format
- schematic is blank on purpose — rebuild it with the converted symbols; original sch lives in `reference/` for copying symbol placement
- keep the GPLv2.1 credit in this readme if u ever share it
