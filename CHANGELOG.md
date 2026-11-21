# Changelog

## 1.0.0

* First release.
* Neither DLL writes files any more: diagnostics go to the debugger output (`OutputDebugString`), so the `Log`
  key is gone and the game folder stays clean.
* Release assets: `ds3-hshi-v1.0.0.zip` (DLL, ini, docs), `ds3-hshi-chr-models-v1.0.0.zip` (165 prepared
  character models — no unpacking, no patching), `ds3-hshi-ds3tae-v1.0.0.zip` (patcher, run with
  `dotnet ds3tae.dll`). No archive contains an executable.
* License split: the mod is MIT, `validator/` and `ds3tae` are GPL-3.0 (they link SoulsFormats), and the models
  pack is modified game data outside the MIT grant.
