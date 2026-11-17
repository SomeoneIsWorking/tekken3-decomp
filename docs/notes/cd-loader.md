# The BNS loader and its libds read path

Tekken 3 reads `TEKKEN3.BNS` through Psy-Q libds (`DsPacket`, `DsStartReadySystem`, `DsGetSector`), not libcd's
`CdRead`. Every load in the game goes through one queue at `g_LoaderHead` (`0x800A0698`).

## State (`0x800A0698`)

| offset | what |
|---|---|
| `+0` | head record pointer, records sorted by disc position |
| `+4` | s16 count of queued records (`LoaderCancel` zeroes it) |
| `+6` | cancel request: `LoaderMarkCancel` sets it while reading; the next sector callback ends the read |
| `+7` | reading: set by `LoaderStartRead`, cleared when the last record lands |
| `+8` | current file id |
| `+0xC` | bytes left in the current record |
| `+0x10` | current destination |
| `+0x14` | current record's done callback |

A record (`g_LoaderRecords`, 8 x `0x1C`): `+0` file id, `+2` starts a new packet (cleared when contiguous with the
previous record), `+4` CdlLOC, `+8` destination, `+0xC` disc sector, `+0x10` bytes, `+0x14` done callback,
`+0x18` next.

## Flow

1. `bns_queue_read(file, dest, done)` maps the file id through `g_BnsFileIndex` to `g_BnsFileTable`
   `{sector offset, bytes}`, adds `DsPosToInt(g_BnsBase)` and inserts the record by sector.
2. `LoaderStartRead` marks contiguous runs, sets reading, and submits
   `DsPacket(0xA0, &record.loc, CdlReadN, LoaderChainComplete, -1)` until accepted.
3. `LoaderChainComplete(2)` (DslComplete) calls `DsStartReadySystem(LoaderSectorCallback, -1)`.
4. `LoaderSectorCallback(1)` (DslDataReady) copies `min(left, 0x800)` bytes with `DsGetSector(dest, words)`.
   At the end of a record it calls the done callback and moves to the next. A record that starts a new packet
   gets `DsEndReadySystem` and a fresh `DsPacket`. After the last, it ends the ready system and clears reading.
   Class 5 (DslDiskError) retries through `0x8008ED1C`.
5. Callers poll `LoaderHeld` (`0x8006C23C`, the reading byte) or `LoaderOverlayBusy`.

Every sector arrives through the drive's INT1 and the libds ready callback, so a load lasts as long as the drive
takes to pass its sectors.
