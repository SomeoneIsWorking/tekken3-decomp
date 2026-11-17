# Unplaced overlay names

Tekken 3 loads its overlays from logical files inside `TEKKEN3.BNS` over the executable's packed tail
(`0x800B0CC0` up), so these addresses hold different code per overlay. A name enters a
`config/symbols/<image>.txt` once that image is extracted and has a config. The image column is the
logical index Tekken3-xport tags the function with (`SLUS_004.02-BNS-logical-NN`).

| address | name | image |
|---|---|---|
| `0x800C2434` | `Mode17_Ranking` | 06 |
| `0x800D3228` | `Mode6_Attract` | 07 |
| `0x800DA910` | `MenuGameStart` | 05 |
| `0x800DAB3C` | `MenuBothSidesHuman` | 05 |
| `0x800DB1B8` | `Mode3_Title` | 05 |
| `0x800DB5B4` | `Mode4_MainMenu` | 05 |
| `0x800DDE64` | `Mode5` | 05 |
| `0x800E1374` | `MovieStep` | 05 |
| `0x800E2054` | `MovieStop` | 05 |
| `0x800E22F0` | `MovieStopTail` | 05 |
| `0x800EEE8C` | `Mode12` | 09 |
| `0x800EF92C` | `Mode13` | 09 |
| `0x800F0458` | `Mode14` | 09 |
| `0x800F18E8` | `Mode15` | 09 |
| `0x800FF0C4` | `Mode19` | 08 |
| `0x8010FF4C` | `Mode9` | 04 |

Data in the same range, read while that overlay is resident:

| address | name | read by |
|---|---|---|
| `0x800B8D58` | `g_SelectionBlock` (per player `0xAC` bytes, `character << 2 \| costume` at `+0x50`) | `Mode10_CharacterSelect` |
| `0x800B981C` | `g_MovieDescriptors` | `MovieStep` |
| `0x800B9C70` | `g_MovieTable` (entry 0 is descriptor 21) | `MovieStep` |
| `0x800EC526` | `g_MovieResult` (`-1` end, `-2` skipped) | `MovieStep` |
| `0x800EC528` | `g_MoviePhase` | `MovieStep` |
| `0x800EC548` | `g_MovieFirstSector` | `MovieStep` |
