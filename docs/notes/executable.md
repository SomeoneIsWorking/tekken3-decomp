# SLUS_004.02 executable structure

## Target

The selected target is the supplied USA disc image (NTSC-U/C). Its `SYSTEM.CNF` names
`cdrom:\TEKKEN3\SLUS_004.02;1`, and the disc directory places that executable at LBA 25.

| Field | Measured value |
|---|---|
| Executable | `TEKKEN3/SLUS_004.02` |
| Disc extent | LBA 25, 1,185,792 bytes |
| SHA-256 (complete PS-X EXE) | `fbda8b68e5799dbef4af39a161783bc670c15b0aa0e87dce65e210717da19b8c` |
| Entry (`pc0`) | `0x80079C70` |
| Load address (`t_addr`) | `0x80010000` |
| Text size (`t_size`) | `0x00121000` bytes |
| Text extent | `[0x80010000, 0x80131000)` |
| Header stack | `0x801FFFF0` |

The startup body zeroes BSS `[0x8009B9A8, 0x800B0548)`, derives GP `0x8009B9A8`, stack top
`0x801FFFF8`, heap base `0x800B0548`, and heap size `0x00147AB0`, then stores the heap size/base
through `0x80098A6C` and `0x80098A68`. Ghidra decompilation and a post-decompile instruction
spot-check agree that the JAL at `0x80079D04` targets `0x80028BA0` with a `nop` delay slot.

That target is Tekken 3's non-returning main loop, not a libc initializer. `crt0_extract` labels the
first startup JAL `libcInit` generically and explicitly reports that this target is not the A(39h)
InitHeap thunk. The project-owned model instead names the boundary `game_main`: the entry's first JAL
at `0x80079D04` targets `0x80028BA0`, its delay slot is a nop, and a MIPS `break` immediately guards
against return at `0x80079D0C`. Inside `game_main`, the unconditional jump at `0x80028E0C` returns to
the frame-loop body at `0x80028BCC`.

The first six instructions of `game_main` allocate its 32-byte frame, save `ra/s2/s1`, and call
`0x80079D10` at `0x80028BB0`; its delay slot saves `s0` with word `0xAFB00010`. The tracked manifest
records those facts.

`FUN_80079d10` is a 28-instruction executable function `[0x80079D10,0x80079D80)`. Ghidra and narrow
instruction inspection agree that it guards and sets the word at `0x80098A64`, contains a
zero-count constructor-loop path on this boot, restores its frame, and returns through `jr ra` at
`0x80079D78` with a nop delay slot. The continuation at `0x80028BB8` immediately calls
`0x800B0548`; its delay word `0x3C11800B` loads `s1`'s high half.

A fresh Ghidra 12.0.4 decompile of the provisioned image (SHA-256 above) confirms both semantics:
`FUN_80079c70` calls `FUN_80028ba0` and then traps, while `FUN_80028ba0` performs one-time calls and
then loops forever around the mode dispatch and two `FUN_8007bab0` calls. The tracked executable and
startup facts live in `executable.json`; historical
interpreter and independent
Mednafen runs agreed on all 35 CPU fields at `0x80028BA0`. A current execution claim needs a bounded
Lightrec/oracle discriminator and cannot be
inferred from `game_main` structure, devices, frames, or gameplay.

The tracked executable writes I_MASK at `0x80085D94`, reads it at `0x80085D98`, writes that result to
I_STAT at `0x80085DA0`, then stores `0x33333333` to DPCR `0x1F8010F0` at `0x80085DB0`. The shared
oracle routes only I_STAT/I_MASK through vendored Mednafen `irq.c`, so the same independent CPU executes
the complete interrupt reset with its real load delay; its retained GPUSTAT negative case still stops.
The oracle models DPCR and continues through the executable's context-save path to a strict capture at
the measured pre-BIOS call site `0x80085DE4`; it does not execute the jump into unmodeled vector `0xB0`.
The executable proves the exact DPCR value, while a separate IRQ process demonstrates non-zero and
zero controller states. Deliberate register/source/boundary/capture changes must fail. This does not independently model B(19),
establish agreement at its return, later initialization, a frame, or gameplay.

