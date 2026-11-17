# Tekken 3 decomp

A decompilation of **Tekken 3** (USA, `SLUS-00402`), set up with [splat](https://github.com/ethteck/splat). It
holds the build-matched split configuration for the executable `SLUS_004.02`, a symbol map from our own reverse
engineering, and the RE notes. No C is matched yet.

The PC port [psx-tekken-3](https://github.com/SomeoneIsWorking/psx-tekken-3) consumes this repository as a
submodule and reads its guest addresses from the symbol map.

## Bring your own disc

No game bytes are ever committed here: not the executable, not `TEKKEN3.BNS`, not the split assembly or data.
Copy `TEKKEN3/SLUS_004.02` from your own disc to `disc/SLUS_004.02` (gitignored). The config carries the file's
SHA-1 and splat refuses a mismatch.

## Split

```
uv run --frozen splat split config/slus_004.02.yaml
```

Output goes to `asm/` and `build/`, both gitignored. splat is pinned in `pyproject.toml`/`uv.lock`.

## Layout

| path | what |
|---|---|
| `config/slus_004.02.yaml` | splat config: tables, code `0x80028BA0..0x80095278`, data, zeroed BSS, init code `0x800B0548..0x800B0CC0`, packed tail |
| `config/symbols/slus_004.02.txt` | the symbol map, splat `symbol_addrs` format |
| `docs/unplaced-overlay-names.md` | named routines and data in the `TEKKEN3.BNS` overlays, waiting for their image configs |
| `docs/re-frontier.md` | the ordered RE evidence chain |
| `docs/notes/` | behaviour notes on guest routines |
| `docs/prior-art.md` | other Tekken 3 work, licences and what we take |

Notes cite psx-tekken-3 paths (`game/...`, `docs/issues/...`, `titles/tekken3/executable.json`) where a port owner
or an issue holds the evidence.

## Licence

MIT (`LICENSE`), covering this repository's own files only. Tekken 3 is © Bandai Namco Entertainment; nothing of
it is distributed here.

Built with AI assistance (Claude and other LLMs).
