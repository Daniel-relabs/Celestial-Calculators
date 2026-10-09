# Celestial Calculators — NavPac Port

Celestial navigation ("sight reduction") programs ported to three handheld calculator platforms: the **HP-48GX**, the **HP Prime G2**, and the **HP-28S**. All three implementations expose the same function names, inputs, and outputs, so a sight worked on one machine can be checked on another.

| Platform | File |
|---|---|
| HP-48GX | [NAV48.txt](NAV48.txt) |
| HP Prime G2 | [NAVPRIME.txt](NAVPRIME.txt) |
| HP-28S | [NAV28S.txt](NAV28S.txt) |

Full function/accuracy/internals reference: `NavPac_functions.xlsx`.

## Conventions

- Angles are decimal degrees; **north and east are positive**.
- HP-48 and HP-28S: UT is entered as `H.MMSS`. HP Prime: decimal hours or the `° ′ ″` template.
- HP-28S results are plain, untagged numbers on the stack, in the order shown below.
- Global settings (variables): `DT` = Delta T / TT−UT in seconds, `IC` = index correction in arcminutes, `HE` = height of eye in metres, `TEMP` = air temperature in °C, `PRES` = pressure in hPa.

## Functions

| Function | Inputs | Returns | HP-48GX | Prime G2 | HP-28S |
|---|---|---|---|---|---|
| `SUN` | year, month, day, UT | GHA, Dec, SD (′), HP (′) | Yes | Yes | Yes |
| `MOON` | year, month, day, UT | GHA, Dec, SD (′), HP (′) | Yes | Yes | Yes |
| `PLANET` | planet (1 Venus, 2 Mars, 3 Jupiter, 4 Saturn, or name), year, month, day, UT | name, GHA, Dec, HP (′) | Yes | Yes | Yes |
| `STAR` | star (Nautical Almanac number 1–57, or name), year, month, day, UT | name, GHA, SHA, Dec | Yes | Yes | Yes |
| `SFIND` | star name | its number | Yes | No (`STAR` accepts names) | Yes |
| `ARIES` | year, month, day, UT | GHA Aries | Yes | Yes | Yes |
| `LHA` | GHA, longitude (east +) | LHA | Yes | Yes | Yes |
| `SRED` | latitude, declination, LHA | Hc, Zn | Yes | Yes | Yes |
| `CORR` | sextant altitude Hs, limb (+1 lower, −1 upper, 0 centre/star), SD (′), HP (′) | observed altitude Ho | Yes | Yes | Yes |
| `INT` | Ho, Hc | intercept in nm (+ towards) | Yes | Yes | Yes |
| `FIX` | DR latitude, DR longitude, list `{Zn1 p1 Zn2 p2 …}` | fix latitude, longitude | Yes | Yes | Yes |
| `DR` | latitude, longitude, course, speed (kn), hours | latitude, longitude (exact rhumb-line sailing) | Yes | Yes | Yes |
| `RL` | departure lat/lon, destination lat/lon | rhumb-line distance (nm), course | Yes | Yes | Yes |
| `OBLQ` | Ho, latitude, Zn, HP (′) | Ho corrected for Earth oblateness (Moon sights, after `CORR`) | Yes | Yes | Yes |
| `PLOT` | DR latitude, DR longitude, list `{Zn1 p1 Zn2 p2 …}` | draws the plotting sheet; leaves fix latitude, longitude | Yes | Yes | Yes |
| `GC` | departure lat/lon, destination lat/lon | distance (nm), initial course, final course | Yes | Yes | Yes |
| `GCVX` | departure lat/lon, destination lat/lon | vertex latitude, longitude | Yes | Yes | Yes |
| `GCPT` | departure lat/lon, destination lat/lon, distance along route (nm) | latitude, longitude | Yes | Yes | Yes |
| `DT`, `IC`, `HE`, `TEMP`, `PRES` | settings variables | ΔT (s), index correction (′), height of eye (m), temperature (°C), pressure (hPa) | Yes | Yes | Yes |
| `CHK` | data table | one checksum per row | No | No | Yes |

Notes:
- Intercepts for `FIX`: reduce each sight from the DR position for its own time (use `DR` to get it), then pass the DR position at fix time.
- Manual-entry route: take GHA and Dec from the printed almanac, then use `LHA` and `SRED`.
- `PLOT`: DR at the axis origin, north up, equal scale. HP-48: scroll with the arrow keys, `CANCEL` to exit. Prime and 28S: any key to exit.

## Accuracy

Worst-case error against JPL DE421, 2024–2050 (arcminutes):

| Body | HP-48GX / Prime | HP-28S |
|---|---|---|
| Sun | 0.03 | 0.1 |
| Moon | 0.16 | 1.2 |
| Venus | 0.04 | 0.21 |
| Mars | 0.05 | 1 |
| Jupiter | 0.07 | 0.28 |
| Saturn | 0.03 | 0.31 |
| Stars | 0.01 | 0.01 |

- GX/Prime: VSOP87 (~990 terms) and the full Meeus/ELP lunar series. Moon figure from 400 dates; others from the validation runs.
- HP-28S: fitted orbital elements plus 36 periodic terms, and a 46-term lunar series. **Fitted to 2020–2050; do not use outside that window.**
- Stars on all machines: Yale Bright Star Catalogue J2000 positions with proper motion, precession, nutation, and aberration.

## Internal routines (normally left alone)

| Name | Purpose | Platform |
|---|---|---|
| `GM` | Matrix element fetch helper | HP-48, HP-28S |
| `JDN`, `JDT` | Julian date; Julian centuries of Terrestrial Time | All |
| `NUTN`, `GST2`, `GAST` | Nutation and true obliquity; apparent sidereal time | All |
| `APP` | Geometric to apparent place (aberration, nutation, frame correction) | All |
| `HPOS`, `PXYZ` | Heliocentric position of a body; geocentric coordinates | HP-48, HP-28S |
| `VSUM`, `HEL` | VSOP87 series evaluation | HP-48, Prime |
| `KEPL`, `PSUM` | Kepler-equation solver and periodic terms | HP-28S |
| `MSUM` | Lunar series evaluation | All |
| `RV`, `VLL`, `XPR`, `NRM` | Unit-vector helpers for great-circle work | All (`XPR`, `NRM` on Prime) |
| `AT2` | Four-quadrant arctangent | Prime |
| `STARS`, `SNAM`, `PNAM` | Star catalogue, star names, planet names | All |
| `MTL`, `MTB`, `MTR` | Lunar series terms (full on GX/Prime, reduced on 28S) | All |
| `VEAR`, `VVEN`, `VMAR`, `VJUP`, `VSAT` | VSOP87 series | HP-48, Prime |
| `EL`, `PT` | Fitted orbital elements and periodic terms | HP-28S |

## Installation

- **HP-28S**: create a directory first (`'NAV' CRDIR`), then press `NAV` in the USER menu. Key in each object from [NAV28S.txt](NAV28S.txt) exactly as shown, then store it with the name in quotes (e.g. `69.2 'DT' STO`). After keying a data table, verify it with `CHK` against the checksum listed for that table — a typo too small to change the 10-digit checksum is too small to matter.
- **HP-48GX**: load [NAV48.txt](NAV48.txt) (HP object/ASCII format, `%%HP:` header) via a connectivity kit or serial transfer.
- **HP Prime G2**: paste the contents of [NAVPRIME.txt](NAVPRIME.txt) into a new program named `NAV`.

## Remaining work

- Testing on the real calculators is pending — all code has been verified in emulation only.
- HP-28S commands to confirm first: `POS` on a list, `→LIST`, `PIXEL`, `PMIN`/`PMAX`, `CLLCD`, `KEY`.
- HP-48GX commands to confirm first: `PLOT` graphics (`PDIM`, `LINE`, `ARC`, `REPL`, `PVIEW`).
- HP Prime commands to confirm first: `TYPE` on strings, program-level variables, `RECT_P` / `LINE_P` / `TEXTOUT_P` / `WAIT`.
