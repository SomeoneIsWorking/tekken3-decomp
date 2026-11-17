# Display and projection ownership

A complete word scan of the hashed loaded image found six canonical writes to GTE control registers
CR24, CR25, and CR26. Ghidra's independent instruction listing agrees on the same six sites:

| Address | Owner | Measured operation |
|---|---|---|
| `0x80081CBC`, `0x80081CC0` | `FUN_80081c50` (`InitGeom`) | initialize OFX and OFY to zero |
| `0x80082730`, `0x80082734` | `FUN_80082728` (`SetGeomOffset`) | publish `a0 << 16` to OFX and `a1 << 16` to OFY |
| `0x80081C9C` | `FUN_80081c50` (`InitGeom`) | initialize H to 1000 |
| `0x80082748` | `FUN_80082748` (`SetGeomScreen`) | publish `a0` to H |

The scan also found the word `0x48CCCCCE` at `0x800BAC20`. It is undisassembled resident data with no
function or control-flow owner, and its reserved low 11 bits make it a noncanonical COP2 move. This
exposed a shared decoder defect which is now fixed at the framework owner: psxport rejects the word's
reserved bits instead of labeling it `ctc2`. Tekken neither exempts that address nor duplicates the
instruction decoder.

The title-level owners above the Psy-Q leaves are now identified:

- `FUN_80080a40(width, height)` owns the current view dimensions. `FUN_80081148` derives the retail
  centre as `(width / 2, height / 2)`, and `FUN_80080da8` adds the current double-buffer offsets,
  calls `SetGeomOffset`, and records the live OFX/OFY at `0x800ADE7C`/`0x800ADE7E`.
- `FUN_80063c64(h)` clamps H to the title's current minimum and maximum before calling
  `SetGeomScreen`. `FUN_80064080` selects a six-field fight-camera pose whose final field is H;
  `FUN_80064170` blends two authored poses, including H, and republishes it through
  `FUN_80063c64`. Fight-camera initialization sets the current/minimum/maximum H to 500.
- The resident table at `0x800B0CC8` contains two 16-byte display/view presets. Preset 0 owns an
  active display rectangle `(0,20,368,448)`, a 384x480 title view, and initial OFX/OFY 192/240.
  Preset 1 owns `(0,10,320,224)`, a 320x240 title view, and initial OFX/OFY 160/120. Both publish
  H=500. Boot calls `FUN_800B0840(0)`.
- That preset call at `0x800B0574` is the first measured widescreen owner after the current
  `FUN_8006AB64` CD-init wedge at `0x800B0564` (the only intervening call is `FUN_800B0788`).
  `FUN_800B0840` calls `FUN_80080848` at `0x800B086C`, which reaches dimension owner
  `FUN_80080A40` at `0x80080890` before the preset derives centre and H. All five call edges are
  recorded in `executable.json`.
- `FUN_8006D014` is the stage submit owner. It calls `FUN_8006D95C` with a horizontal visibility
  angle of 600 normally and `0x30C` in mode 6. That helper traces the two rays at camera yaw plus and
  minus half the supplied angle and selects visible cells from the stage's 6x6 tile grid.
- `FUN_8006CC28` converts the selected stage primitives into ordering-table packets. Its triangle,
  quad, and sprite paths contain eleven exact signed `-368` comparisons and discard primitives whose
  projected vertices are wholly beyond the retail active-display right edge. `FUN_8006E44C` applies
  the twelfth rendering-path comparison to an effect-primitive path. These are title culling owners, not host
  viewport policy. The separate `-368` use in `FUN_80054B48` is a player-select text slide distance
  and must remain in the retail 2D layout.

The 384-wide title projection and 368-wide active display are deliberately different facts. A wide
implementation must therefore carry the title-authored projection width/centre separately from the
PSX display mode, resolve both retail modes from their 4:3 presentation semantics, keep H and the
vertical centre unchanged, and widen guest geometry, draw coverage, and final sampling as one plan.
Changing only the host viewport would stretch the picture; changing only OFX would crop it.
The framework decodes GP1(08) bit 6, so the 368-pixel preset resolves generically; a title-side 368
override would only split the two owners.
The stage/effect right-edge tests must consume the same resolved wide display bound, while the stage
tile visibility angle must be checked against the resolved wide frustum. Static evidence identifies
that owner but does not select an angle formula: if the authored wedge becomes too narrow, any change
must derive from the retail angle and resolved projection rather than a replacement constant. Porting
those title functions must call their authenticated original guest bodies as the 4:3 differential control; the
evidence does not justify a new renderer or producer.

Static ownership does not establish a rendered frame. The next execution boundary is generic DPCR/DMA
modeling after the unsupported write at `0x80085DB0`; once independent execution reaches a real
display/frame boundary, a final-presentation A/B must show a bit-identical 4:3 control, unchanged
vertical projection, horizontal translation about the widened centre without scale change, and new
scene geometry in the added margins.

