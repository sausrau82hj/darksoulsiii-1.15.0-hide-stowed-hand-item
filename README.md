# Hide Stowed Hand Item — *Dark Souls III*

While you two-hand one hand's weapon, the **other hand's sheathed model is no longer drawn on your back**. Works for anything hand-equippable: weapons, shields, catalysts, bows, staves.

> Local memory + model changes only. Online play has a server-side save sanity check — use it offline.

## Requirements

* A **loose-file setup** (`Game\chr\*.chrbnd.dcx` exists — the normal state after unpacking with UXM, which model mods require).
* Windows.

## Install

The game must be closed.

1. **Back up** `Game\chr\*.chrbnd.dcx` (about 340 MB for the full 165-model set) and, if it already exists, `Game\xinput1_3.dll`.
2. **Copy the payload** into `Game\`: `HideSheathedLeft.dll`, `HideSheathedLeft.ini`, `xinput1_3.dll`.
3. **Patch every character model** (run from the folder you unpacked this repository into, with `<game>` = `...\DARK SOULS III\Game`):

```
tools\ds3tae.exe parkdir "<game>\chr" INPLACE "382,383" "0,-1000,0" 0 "chrbnd"
```

4. **Verify** — the two numbers must match:

```
tools\ds3tae.exe verify "<game>\chr" chrbnd "382,383"
```

A smaller second number means some model was missed; re-run step 3 (already-patched files are skipped).

`xinput1_3.dll` only exists to make the game load `HideSheathedLeft.dll`: it forwards the XInput exports to the system DLL, so controllers are unaffected. Only one `xinput1_3.dll` can live in the game folder — if another mod already owns it, rename that file first.

Step-by-step with troubleshooting: [`docs/MANUAL-INSTALL.txt`](docs/MANUAL-INSTALL.txt).

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
| `Rescan` | `1` | `0` = stop the periodic address-space re-scan |
| `ScanMapped` | `0` | `1` = also consider MEM_MAPPED regions (diagnostic) |
| `Probe` | `0` | `1` = observation only, writes nothing |
| `Log` | `0` | `1` = write `HideSheathedLeft.log` next to the ini (diagnostics only; off by default) |
| `LoadCore` | `1` | `0` = the loader shell loads nothing (lets you keep `xinput1_3.dll` in place while the mod is off) |
| `ExtraMods` | *(empty)* | must stay empty — loading additional DLLs on 1.15.0 crashes the game |

`HideLeftHang=0` restores vanilla for the right-hand two-hand stance only; `HideRightHang=0` does the same for the left-hand stance. The older key `HidePairedOffhand` is still read when `HideBothPaired` is absent.

## Uninstall

1. Delete `Game\HideSheathedLeft.dll` and `Game\HideSheathedLeft.ini`.
2. Put your own `Game\xinput1_3.dll` back (or delete the file if you had none).
3. Put the character models back from your backup. Without a backup, strip the dummies instead:

```
tools\ds3tae.exe stripdir "<game>\chr" INPLACE "382,383" "chrbnd"
```

A stripped bundle is structurally identical but **not byte-identical** (the DCX is re-compressed). For exact bytes use your backup, or delete the loose `chr\*.chrbnd.dcx` files and let the game read the packed ones. Prove the result with the `verify` command from the install section — a clean game ends in `verify: CLEAN`.

To only switch the effect off, set `HideLeftHang=0` and `HideRightHang=0` in the ini; that applies within a second and leaves the install in place.

## Known limitations

* **NPCs hide the item too.** The parameter table is shared between the player and NPCs, and the parking dummy is valid for every character model, so an NPC in the same stance simply does not draw its sheathed item instead of producing artefacts. Player-only behaviour would require patching the engine's attach resolver.
* After another mod replaces files in `chr\`, run the `parkdir` command again; already-patched files are skipped.

## Source

`src\` holds the C++ sources of both DLLs, `validator\` the C# source of `tools\ds3tae.exe`. Rebuild from a VS x64 developer prompt:

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
validator\                    C# source of tools\ds3tae.exe
tools\ds3tae.exe              patch / verify / strip tool (+ SoulsFormats.dll)
docs\MANUAL-INSTALL.txt       step-by-step manual install
```

## Notes

* Backups are your uninstall path — 340 MB for the full `chr\` set, and the reason step 1 exists.

## License

MIT — see [`LICENSE`](LICENSE). Not affiliated with FromSoftware or Bandai Namco; single-player, offline use only.
