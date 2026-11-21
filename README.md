# Hide Stowed Hand Item — *Dark Souls III*

Two-hand one weapon and the other hand's sheathed model stops being drawn on your back — weapons, shields,
catalysts, bows, staves. Single-player, offline.

## How it works

`HideSheathedLeft.dll` points the sheathed and off-hand model fields of `WEP_ABSORP_POS_PARAM_ST` at a parking
dummy (382, spare 383) that sits out of view. Those dummies have to exist in the character models, so either install
the prepared models or add them to your own with `ds3tae`.

## Files

| Release asset | Contents |
|---|---|
| `ds3-hshi-v1.0.0.zip` | `HideSheathedLeft.dll`, `HideSheathedLeft.ini`, docs — 215 KB |
| `ds3-hshi-chr-models-v1.0.0.zip` | 165 patched `chr\*.chrbnd.dcx` — 342 MB |
| `ds3-hshi-ds3tae-v1.0.0.zip` | the patcher: .NET 10 assembly, run `dotnet ds3tae.dll <command>` |

## Install

Two choices to make: where the patched models come from, and how the DLL gets loaded. The game must be closed.

### Models — pick one

**A. Prepared models.** Unzip `ds3-hshi-chr-models-v1.0.0.zip` and treat its `chr` folder as a mod folder. Nothing
is unpacked or patched. It replaces every character model, so it conflicts with other mods that edit `chr`.

**B. Patch your own.** Extract `chr\*.chrbnd.dcx` (UXM **Unpack** is enough — no exe patch needed), keep the copies
in one folder, then:

```
dotnet ds3tae.dll parkdir "<chr>" INPLACE "382,383" "0,-1000,0" 0 "chrbnd"
dotnet ds3tae.dll verify  "<chr>" chrbnd "382,383"
```

The two numbers printed by `verify` must match. Nothing is redistributed on this route, and other model mods keep
working, but you need `ds3tae` and the [.NET 10 runtime](https://dotnet.microsoft.com/download/dotnet/10.0).

### Loading — pick one

Load `HideSheathedLeft.dll` with [me](https://www.nexusmods.com/darksouls3/mods/2410),
[me2](https://github.com/soulsmods/ModEngine2) or [me3](https://github.com/garyttierney/me3) — or with the bundled
`xinput1_3.dll` shell. Either way the DLL and the ini sit next to the `chr` folder. With a loader, nothing in the
game folder changes; with the shell, copy `HideSheathedLeft.dll`, `HideSheathedLeft.ini` and `xinput1_3.dll` into
`Game\` and the patched models into `Game\chr`, which needs the game unpacked with UXM **Unpack** *and* **Patch**.
The shell only loads this DLL and forwards XInput, so controllers are unaffected; one `xinput1_3.dll` per game
folder, so rename another mod's copy first if there is one.

## Uninstall

- **Loader** — remove the mod folder from the loader profile.
- **No loader** — delete `HideSheathedLeft.dll` and `.ini`, put your own `xinput1_3.dll` back, then restore
  `Game\chr` from your backup, or delete the loose `chr\*.chrbnd.dcx` files and let the game read its archives.
- **Patched models left behind** — strip the dummies instead:

  ```
  dotnet ds3tae.dll stripdir "<chr>" INPLACE "382,383" "chrbnd"
  ```

  A stripped bundle is structurally identical but not byte-identical (the DCX is re-compressed); `verify` ending in
  `verify: CLEAN` is what proves it either way.
- **Effect off only** — `HideLeftHang=0` and `HideRightHang=0` in the ini, applied within a second.

## Configuration

`HideSheathedLeft.ini` is re-read every second: edit, save, done. A missing file means all defaults.

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
| `LoadCore` | `1` | `0` = the shell loads no core DLL, so the mod can stay installed but off |
| `ExtraMods` | *(empty)* | further DLLs for the shell to load as well, comma- or space-separated; when the key is missing entirely it falls back to `dkp_loader.dll` |
| `ExtraModsDelayMs` | `0` | how long the shell waits before loading `ExtraMods` |

`HideLeftHang=0` restores vanilla for the right-hand two-hand stance only; `HideRightHang=0` does the same for the
left-hand stance. The older key `HidePairedOffhand` is still read when `HideBothPaired` is absent.

## Known limitations

- **NPCs hide the item too.** The parameter table is shared, so an NPC in the same stance simply does not draw its
  sheathed item. Player-only behaviour would mean patching the engine's attach resolver.
- **Other model mods.** After another mod replaces something in `chr\`, run `parkdir` again; patched files are
  skipped. The prepared model pack cannot merge with such mods at all.
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
- The models pack contains modified FromSoftware assets, single-player use only. Details in
  [`THIRD-PARTY-NOTICES.txt`](THIRD-PARTY-NOTICES.txt).

Not affiliated with FromSoftware or Bandai Namco.
