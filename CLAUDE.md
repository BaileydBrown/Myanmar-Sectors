# CLAUDE.md

## What this repo is
EuroScope sector package for the **Yangon FIR (VYYF, Myanmar)** on VATSIM, maintained by VATSIM Southeast Asia Division (VATSEA). Copyright 2021-2023. Credits: Bailey Brown (1376159), Amos Ng (1357854), Resh (1405734), David Lee (1161277), Duc Nguyen (1502513).

There is no code, build or test suite. It is data and config files for the EuroScope ATC client. Users install it by creating a "Myanmar Sectors" folder in their EuroScope folder and dropping the files in (see README.md).

## Layout
- `*.prf` (repo root): one EuroScope profile per position, e.g. `VYAN Tower.prf`, `VYBG Approach.prf`, `VYYF Enroute.prf`, `VYYY Tower.prf`. About 37 files of roughly 3.2 KB each, all near-identical.
  - They point at the shared files under `\Data\...`: sector file, settings, sounds, ICAO lists, plugins, alias.
  - Per-profile differences are in the `LastSession` lines (`callsign` such as `VYAN_TWR`, `facility`, `rating`, `server`, `atis2`, `atis4`) and in `RecentFiles`, which points to that position's `.asr`.
  - Profiles default to the SINGAPORE server and connect to VATSIM.
- `ipaddr.txt`: EuroScope server list (AFV data, regional servers, SweatBox).
- `Data/Sector/`
  - `VYYF.sct`: geography. Sections: [INFO], [VOR], [NDB], [AIRPORT], [RUNWAY], [FIXES], [ARTCC] (about 1,400 lines, FIR and neighbour boundaries), [ARTCC LOW], [SID], [STAR], [LOW AIRWAY], [HIGH AIRWAY], [GEO] (about 6,000 lines, airport taxiways and ground layout), [REGIONS].
  - `VYYF.ese`: ATC data. Sections: [POSITIONS] (callsign:name:freq:id:...), [FREETEXT], [SIDSSTARS], [AIRSPACE] (about 6,500 lines, sector volumes), [RADAR].
  - Header: sector version `Rangoon2209`, center VYYY (N020.05.54 E098.32.00).
- `Data/ASR/`: radar-screen display files, one per position (e.g. `VYMD Approach.asr`, `VYYF.asr`). They list which ARTCC boundaries, free text (neighbour FIRs, sector frequencies) and geo layers to show.
- `Data/Settings/CTR/`: shared EuroScope settings (symbology, screens, lists, plugins, VCCS, voice, profiles, login, sector inbound and exit lists). All `APP_CTR_*` files and `Plug-in settings.txt` plus related files.
- `Data/Alias/alias.txt`: controller dot-commands (`.on`, `.off`, `.clr`, `.clr2`, `.cli`, ...) using `$callsign`, `$arr`, `$cruise`, `$sid`, `$asquawk` and similar variables.
- `Data/ICAO/`: aircraft types, airlines (code, name, callsign) and airports lookup tables.
- `Data/NAVDATA/`: Navigraph worldwide data in EuroScope format, now **AIRAC 2611** (valid 29 Oct - 25 Nov 2026; `cycle.json` and `cycle_info.txt` carry the stamp). Licensed for simulation use only.
  - `AIRWAY.txt`: tab-separated, one line per fix per airway: `fix lat lon 14 airway L|H|B prevFix lat lon alt Y|N nextFix lat lon alt Y|N`. About 105k lines.
  - `ISEC.txt` (`isec.txt` is the same file on Windows): `name lat lon 15`, waypoints only, with padded decimal coordinates. About 311k lines, matching Navigraph's `Waypoints.txt`.
  - `icao.txt`: `ICAO lat lon name` for about 16k airports.
  - The AIRAC files were refreshed on 2026-11-02 and are uncommitted. The old 2307 headers (`;` comment lines) are gone.
- `Data/Plugins/`: `AfvEuroScopeBridge.dll` (Audio for VATSIM bridge), `RDFPlugin.dll` (radio direction finder), and the Discord EuroScope radio callsign list for Hong Kong and others.
- `Data/Sounds/`: `.wav` alert sounds referenced by the profiles.
- `Data/vATIS/`: vATIS 4.0+ profile `vATIS Profile - VYYF Yangon FIR.json` and a readme.
  - Covers VYYY and VYMD runway configurations. For VYYY: SOD, South, North VMC and North IMC, in that order of preference.
  - **Runway 03 must be spelt "O3"** (letter O) so the ATIS reads "ZERO THREE". Do not "fix" it.

## Facilities covered
Tower and approach positions for Myanmar airports, named by ICAO code, e.g. VYYY Yangon, VYMD Mandalay, VYNT Nay Pyi Taw, VYAN Ann, VYAS Anisakan, VYBG Nyaung-U (Bagan), VYKP, VYSW, VYTL and others. Enroute is VYYF_CTR, with Rangoon sector splits 1-4 in the ASR files. Neighbour FIRs: VECC, VEGF, VGFR, VLVT, VOMF, VTBB, WMFC, ZPKM.

## Gotchas
- **Case mismatches.** `.prf` files reference `\Data\NAVDATA\airway.txt` but the file is `AIRWAY.txt`. Both `ISEC.txt` and `isec.txt` exist. Commit `97ad84a` deleted the lowercase `airway.txt`. This is harmless on Windows and breaks on case-sensitive systems.
- **Missing file.** The profiles reference `\Data\Settings\CTR\APP_CTR_Tags.txt`, which is not in the repo.
- `.gitignore` ignores `Data/Sector/VYYF.rwy`, a file EuroScope generates.
- **Working-tree state.** `Data/NAVDATA/ISEC.txt` shows as modified with a huge diff (about 467k lines). This is likely line endings or re-export noise, not intentional edits. Check with `git diff --ignore-all-space --stat` before committing it.
- Files are large plain text. Avoid reading them whole. Use Grep with section headers (`^\[`) or `offset`/`limit`.
- **Version stamp.** If you edit the .sct or .ese files, update the "Rangoon sector version" header comment and the [INFO] name. Both files carry it.
- Commit history is mostly "Update README.md" and "Add files via upload", so there is no strong commit-message convention.

## Development tools (gitignored)
`Development/Tools/` is the author's local, untracked toolbox (listed in `.gitignore`). It holds Python scripts and a `Navigraph Data/` folder with a raw Navigraph export in the Aerosoft/FSX "custom" format (`Airports.txt`, `Navaids.txt`, `Waypoints.txt`, `ATS.txt`, `proc/` per-airport SID/STAR/approach files, `Cycle.txt`).
- `convertcoords.py`: decimal degrees to the sector file's `N020.05.54.758 E098.32.00.275` format.
- `extractnavaid.py` and `extractwaypt.py`: filter `Navaids.txt` and `Waypoints.txt` to a lat/long box and write `[VOR]`, `[NDB]` and `[FIXES]` lines. The box is hard-coded to Bangladesh and must be edited per FIR.
- `generate_airway.py`: asks for an FIR name, reads `<FIR>_Waypoints.txt` and `<FIR>_Navaids.txt`, and writes `<FIR>_1Airways.txt` in `[LOW/HIGH AIRWAY]` form. The `- Copy` variant extracts a hard-coded airway list instead.
- These tools produce **sector-file (.sct) output for a bounding box**, not the worldwide `NAVDATA` formats. `ATS.txt` has no altitude or L/H/B data, so `AIRWAY.txt` cannot be rebuilt from it.
- On this machine `python` on PATH is the Windows Store stub and fails. Use `%LOCALAPPDATA%\Programs\Python\Python312\python.exe`.

## STAR and approach connector convention (VYYF.ese)
- A `STAR:` line ends where the published STAR ends (the last fix in the PFPX/Navigraph `STR` record), not at the approach.
- Each STAR (and each transition STAR such as `KTxENSIT1B`) gets one **connector** line per approach type available on that runway. A connector is named `<STAR name>x<ILS|RNP>` and holds the **full route**: the whole STAR plus the approach feeder fixes. Example: `STAR:VYHH:36:REMIN1BxRNP:REMIN DUMER HH901 BINAN PASNO`.
- The `x` separator is also used inside STAR names for transitions, so a connector for a transition reads `KTxENSIT1BxILS`.
- Which types exist comes from `Development/Tools/Navigraph Data/proc/<ICAO>.txt` (`I` = ILS, `R` = RNP; `D`/`Q`/`N` are VOR, RNAV, NDB). Generate both xILS and xRNP when both exist. Use only xRNP where there is no ILS.
- Current feeders (STAR end fix, then the connector tail):
  - VYHH 36: DUMER or KALSI, then `HH901 BINAN PASNO` (RNP).
  - VYMD 17: MDN01, then `TADAU` (ILS) or `MDN05 MDN04` (RNP). VYMD 35: MDS02 or MDS03, then `MDS05 MDS04` (RNP).
  - VYTD 02: NODEN or SUSEN, then `TD901 SALNI` (RNP).
  - VYYY 21: PAKSU, DANSO or GONAS, then `HLEGU BAVEL SULAP` (ILS) or `HLEGU SULAP` (RNP). VYYY 03: EMREL or PALPO, then `YY902 LUNGO` (RNP).
- Transitions that PFPX folds into the base procedure (e.g. `MAPKO1A` containing `POXEM`) stay as separate `x` transitions in the ESE.
- `HLEGU` (waypoint) and `HGU` (VOR) are the same point. The maintainer confirmed the SID `BAGOO1A` starts at `HLEGU`.
- `.sct` `[SID]`/`[STAR]` drawing blocks must match the `.ese`: one block per `.ese` SID/STAR with two or more fixes, labelled `<ICAO> SID|STAR <rwy> <name>` with the exact `.ese` name (connectors and transitions included). Each block draws `a a b b` segments for consecutive route fixes.
  - When regenerating a block, keep its existing hand-drawn segments that use real coordinates (arcs, runway tracks) and any `COLOR_*` tags on segments that still exist. Zero-coordinate (`N000.00.00.000`) placeholders are replaced.
  - Leave `Holding`, `RNP`/`RNP MISAPP` approach drawings and the old-style transition fragments (`VYYY STAR LUDVI ENSIT`) alone.
  - `INP11` and `INPT6` (VYNT ILS) are used by the `.ese` but are not defined as fixes in the `.sct`. `VYTD RNP 02` is still an all-zero placeholder.
- The sector version string ("Rangoon sector version NNNN" in the `.sct` and `.ese` headers, and `RangoonNNNN` in the `.sct` [INFO]) is set to the AIRAC cycle.

## Generated holds and initial-climb drawings (.sct)
Navigraph's Myanmar procedure data has no RF/AF arc legs, so curves are not drawn from the source. Two things are generated instead (script: `Development/Tools/draw_holds_and_climbs.py`; it edits `VYYF.sct` in place, so run it on a clean copy only):
- **Holds.** Every `HM`/`HF` record in `proc/VY*.txt` becomes a `<FIX> Holding` block in `[STAR]` (with the inbound course appended, e.g. `MIA Holding 016`, when a fix has several). Racetrack = inbound course, turn direction (`1` left, `2` right), 4.3 NM legs, standard-rate (3 deg/s) turns at the record's speed (210 kt if none), magnetic variation 0 (as in the sct [INFO]). These are computed geometry, not charted tracks.
- **SID initial climbs.** For SIDs whose first leg is `CA`/`VA`, a stub is added from the departure end of the runway along the heading, then straight to the SID's first fix. Stub length is an assumption: (CA altitude minus runway elevation) / 200 ft per NM, between 0.5 and 6 NM.
- SID blocks that already contain hand-drawn coordinate segments are never touched. SIDs whose first leg is not `CA`/`VA` (most VYMD, VYSW, VYME) get no stub.
