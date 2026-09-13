# Engine Dossier — Portal (Source)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `dev-archive/` and `modding-notes/` folders; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** M0, first static look (2026-09-13); the game has not been launched yet. · **VR-readiness verdict:** TBD. Nothing seen so far rules it out.

## 1. Identity
- Game / build / version: Portal, Steam build. Launcher `hl2.exe` (linked 2024-11-13); `bin\engine.dll` 2024-12-09 and `portal\bin\client.dll` 2025-05-27, so this is the updated Source branch, not the 2007 original.
- Platform & store; unofficial port? (extra fragility/legal notes): Steam (PC). Official release, not a fan port.
- Legitimacy: owned copy confirmed.

## 2. Engine lineage
- Family / base engine and how it was modified: Valve's Source engine `[reported]`, in its updated 2024 form. Game code is in `portal\bin\client.dll` and `server.dll`.
- Middleware (animation, audio, physics, megatexture, CUDA, etc.):
- Distinctive file formats / build tags / symbol naming: Standard Source layout (`hl2\`, `portal\`, `platform\`, VPK archives). Valve's own mod tools ship in `bin\` (Hammer, VRAD and others).

## 3. Binary & memory
- 32/64-bit, size, module base, ASLR behaviour (stable base? relocations?): **32-bit** throughout (`hl2.exe`, `engine.dll`, `client.dll`); no 64-bit binaries found in this install `[inferred-static 2026-09-13]`.
- Renderer API (D3D11/12, DXGI, GL, Vulkan) with evidence: Direct3D 9 by default (`shaderapidx9.dll`), with a Vulkan path also shipped (`shaderapivk.dll`, `dxvk_d3d9.dll`) `[inferred-static 2026-09-13]`.
- Developer console / cvar system present? how opened?: not yet investigated.

## 4. DRM / anti-debug & injection foothold
- DRM (CEG/Denuvo/GOG/none); launch-time-debugger behaviour: Steam only `[inferred-static 2026-09-13]`.
- Attach workflow that works: not yet tested.
- Injection vector that works (proxy DLL name / injector / framework): not yet tested.

## 5. Threading & frame structure
- Immediate context only, or deferred contexts + command lists?:
- Which thread(s) do what; render-thread name(s):
- One-frame walkthrough (record → replay → present):

## 6. Camera & projection delivery (the crucial section)
- How the world transform reaches the GPU (shared VP buffer / per-draw MVP /
  other), with **shader-reflection / disassembly evidence**:
- Exact constant-buffer slot, parameter name(s), byte offset(s), layout,
  handedness, row/column convention:
- Where projection `P` / FOV comes from:
- The per-eye override maths (`K_eye = …`):

## 7. Constant-buffer fill mechanism
- Map/DISCARD ring / UpdateSubresource / D3D11.1 offset / **persistent map +
  memcpy** (trap):
- Can source contents be read cheaply (captured CPU pointer) or need staging
  read-back?:
- The chosen override patch point and why:

## 8. Pass inventory (by render target)
- Main scene (res/formats):
- Shadow passes (depth-only sizes):
- Post / AA chain (SMAA/TAA/motion vectors; downscale sizes):
- UI / HUD (how it's kept separate):

## 9. cvar / console cheat sheet
| command / cvar | effect | use |
|---|---|---|
| | | |

## 10. Autonomous harness recipe (this game)
- Launch to a known scene (commands used):
- In-process input / camera drive method that worked:
- Frame-capture method; where images land:

## 11. Dead ends & false leads (save future time)
- none yet.

## 12. Open risks toward the North Star
- ⭐ **Valve's own leftover VR mode is still in the code:** `engine.dll` and `client.dll` both contain OpenVR messages ("VR mode will not be enabled") and references to `sourcevr`, but no `sourcevr.dll` ships `[inferred-static 2026-09-13]`. Whether that code can still be switched on is unknown, and worth checking first.
- Community Source VR projects exist; whether any covers Portal is for `/gr` to establish, not assumed here.
