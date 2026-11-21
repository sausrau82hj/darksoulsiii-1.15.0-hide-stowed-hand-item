# Hide Stowed Hand Item — Dark Souls III 1.15.0

While you two-hand one hand's weapon, the other hand's sheathed model is no longer drawn on your back. Works for
anything hand-equippable: weapons, shields, catalysts, bows, staves.

Single-player, offline. Verified on Dark Souls III 1.15.0 (Steam, 64-bit, Windows).

## How it works

`HideSheathedLeft.dll` points the sheathed and off-hand model fields of `WEP_ABSORP_POS_PARAM_ST` at a parking
dummy (`382`, spare `383`). That dummy is a zero-size locator, so the item attaches and draws nothing. The dummies
have to exist in the character models, which is what `ds3tae` adds — the tool edits the files you already own, and
no game assets are redistributed here.

## Requirements

- Dark Souls III 1.15.0, Steam, 64-bit, Windows.
- Loose `Game\chr\*.chrbnd.dcx` (a UXM-unpacked game), or a mod loader that overrides `chr\` — see below.
- [.NET 10 runtime](https://dotnet.microsoft.com/download/dotnet/10.0) for the patcher.
- The `ds3tae` patcher from the [releases page](https://github.com/sausrau82hj/darksoulsiii-hide-stowed-hand-item/releases).
  It is not inside the mod archive: executables are not allowed in Nexus uploads.

## Install

Close the game. `<game>` below is `...\DARK SOULS III\Game`.

1. Copy into `Game\`: `HideSheathedLeft.dll`, `HideSheathedLeft.ini`, `xinput1_3.dll`.
2. Back up `Game\chr\*.chrbnd.dcx` (about 340 MB for 165 models). That backup is your uninstall.
3. Add the parking dummies to every character model:

   ```
   ds3tae.exe parkdir "<game>\chr" INPLACE "382,383" "0,-1000,0" 0 "chrbnd"
   ```

4. Check the result — both numbers must match:

   ```
   ds3tae.exe verify "<game>\chr" chrbnd "382,383"
   ```

5. Start the game. Models are read at process start, so a restart is needed.

`xinput1_3.dll` only exists to make the game load `HideSheathedLeft.dll`; it forwards the XInput exports to the
system DLL, so controllers are unaffected. One `xinput1_3.dll` can live in the game folder — if another mod owns
it, rename that file or load this DLL through a mod loader instead.

## With a mod loader

A loader loads `HideSheathedLeft.dll` for you and can serve the patched `chr\` files from its own mod folder, so
neither `xinput1_3.dll` nor `Game\chr` has to change. Extract the originals once, run `parkdir` on the copies,
then point the loader at that folder:

- **me3** (maintained) — [releases](https://github.com/garyttierney/me3/releases): a profile with a `natives`
  entry for the DLL and a `packages` entry for the patched `chr` folder.
- **Mod Engine 2** (discontinued) — [releases](https://github.com/soulsmods/ModEngine2/releases): `external_dlls`
  plus a mod folder.
- **Mod Engine** — [Nexus](https://www.nexusmods.com/darksouls3/mods/332): `modOverrideDirectory="\mod"` and
  `chainDInput8DLLPath="\HideSheathedLeft.dll"` in `modengine.ini`.

## Configuration

`HideSheathedLeft.ini` is re-read every second: edit, save, done — no restart. A missing file means all defaults.

| Key | Default | Meaning |
|---|---|---|
| `HideLeftHang` | `1` | hide the sheathed left-hand item (reached by two-handing the **right** weapon) |
| `HideRightHang` | `1` | mirror image: hide the sheathed right-hand item (reached by two-handing the **left** weapon) |
| `HideBothPaired` | `1` | hide the spare model of a one-handed paired weapon, **both hands** |
| `PairedFields` | `1,2,3` | which models of a paired weapon to hide: `1` = blade only (keeps the scabbard), `none` = leave paired weapons alone |
| `HidePoly` | `382` | parking dummy id (`383` spare) |
| `IntervalSeconds` | `1` | how often the parameter values are re-asserted |
| `Rescan` | `1` | `0` = stop the periodic address-space re-scan (the performance switch) |
| `ScanMapped` | `0` | `1` = also consider MEM_MAPPED regions (diagnostic) |
| `Probe` | `0` | `1` = observation only, writes nothing |
| `Log` | `0` | `1` = write `HideSheathedLeft.log` next to the ini (diagnostics only) |
| `LoadCore` | `1` | `0` = the loader shell loads nothing, so the mod can stay installed but off |
| `ExtraMods` | *(empty)* | must stay empty — loading additional DLLs on 1.15.0 crashes the game |

`HideLeftHang=0` restores vanilla for the right-hand two-hand stance only; `HideRightHang=0` does the same for the
left-hand stance. The older key `HidePairedOffhand` is still read when `HideBothPaired` is absent.

## Uninstall

1. Delete `Game\HideSheathedLeft.dll` and `Game\HideSheathedLeft.ini`.
2. Put your own `Game\xinput1_3.dll` back, or delete it if you had none.
3. Restore `chr\` from your backup. Without one, strip the dummies instead:

   ```
   ds3tae.exe stripdir "<game>\chr" INPLACE "382,383" "chrbnd"
   ```

   A stripped bundle is structurally identical but not byte-identical (the DCX is re-compressed). For exact bytes
   use the backup, or delete the loose `chr\*.chrbnd.dcx` files and let the game read its archives. Either way,
   `verify` must end in `verify: CLEAN`.

To only switch the effect off, set `HideLeftHang=0` and `HideRightHang=0`; that applies within a second.

## Known limitations

- **NPCs hide the item too.** The parameter table is shared, so an NPC in the same stance simply does not draw its
  sheathed item. Player-only behaviour would mean patching the engine's attach resolver.
- **Other model mods.** After another mod replaces something in `chr\`, run `parkdir` again; patched files are
  skipped.
- **Black triangles or flicker** mean a model is missing the dummy — run `verify`.
- **Online.** Only local memory and local model files change, but online play has a server-side save check: stay
  offline.

## Build

From a VS x64 developer prompt, in the repository root:

```
cl /nologo /O2 /EHsc /std:c++17 /DNDEBUG /LD src\HideSheathedLeft.cpp /Fe:HideSheathedLeft.dll /link /INCREMENTAL:NO
cl /nologo /O2 /EHsc /std:c++17 /DNDEBUG /LD src\xinput1_3_proxy.cpp /Fe:xinput1_3.dll /link /DEF:src\xinput1_3_proxy.def /INCREMENTAL:NO
dotnet build validator\validator.csproj
```

## Repository layout

```
HideSheathedLeft.dll / .ini   mod core and its configuration
xinput1_3.dll                 loader shell (forwards the system XInput)
src\                          C++ sources of both DLLs
validator\                    C# source of ds3tae (patch / verify / strip)
tools\                        ds3tae build output and SoulsFormats.dll
docs\MANUAL-INSTALL.txt       longer walkthrough with troubleshooting
```

## License

- Mod DLLs, `src\`, `docs\` and this README: MIT — see [`LICENSE`](LICENSE).
- `validator\` and `tools\ds3tae.*`: GPL-3.0 — see [`validator/LICENSE`](validator/LICENSE). `ds3tae` links
  [SoulsFormats](https://github.com/JKAnderson/SoulsFormats), which is GPL-3.0.

Not affiliated with FromSoftware or Bandai Namco. See [`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt).
