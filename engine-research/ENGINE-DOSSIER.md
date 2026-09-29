# Engine Dossier — Portal (Source)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `dev-archive/` and `modding-notes/` folders; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** M0, static recon done on both machines (2026-09-13 home, 2026-09-14 dev PC); the game has **not** been launched yet. · **VR-readiness verdict:** the most promising of the 2026-09-13 batch — Valve's own VR *client* still ships whole inside `client.dll`, and exactly one module (`sourcevr.dll`) is missing. Nothing has been run, so this is a paper verdict.

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
- Renderer API (D3D11/12, DXGI, GL, Vulkan) with evidence: Direct3D 9 by default (`shaderapidx9.dll`), with a Vulkan path also shipped (`shaderapivk.dll`, `dxvk_d3d9.dll`, `shaderapiempty.dll`) `[inferred-static 2026-09-13]`, re-confirmed from the `bin\` listing 2026-09-14.
- Developer console / cvar system present? how opened?: Source's own cvar system. `engine.dll` accepts the switches `-console -dev -window -width -height -insecure`; **no `-vr` switch exists** `[inferred-static 2026-09-14]`.

## 4. DRM / anti-debug & injection foothold
- DRM (CEG/Denuvo/GOG/none); launch-time-debugger behaviour: Steam only `[inferred-static 2026-09-13]`. ⚠️ **ASLR is ON** for `hl2.exe`, `engine.dll` and `client.dll` `[inferred-static 2026-09-14]` — unlike Hard Reset, Prototype and Dead Space 2, which all load at a fixed base. Addresses here are only valid within one run; resolve by module base + offset.
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

## 9a. Valve's leftover VR mode — what is actually in the files `[inferred-static 2026-09-14]`

Full evidence: `dev-archive/recon/2026-09-14-dev-pc-static-pass/` (README + raw string dumps).

**The client half ships complete.** `portal\bin\client.dll` (linked 2025-05-27) holds
`CClientVirtualReality` / `IClientVirtualReality` with its full app-system registration chain, the
build path `src\game\client\client_virtualreality.cpp`, `CClientVirtualReality::DrawMainMenu`, the
config hook `exec sourcevr_%s.cfg`, and **36 `vr_*` cvars** — far more than a stereo toggle:

| group | cvars |
| --- | --- |
| activation | `vr_activate`, `vr_activate_default`, `vr_deactivate`, `vr_toggle` |
| aiming | `vr_moveaim_mode`, `vr_moveaim_mode_zoom`, `vr_aim_yaw_offset`, `vr_cycle_aim_move_mode`, the four `vr_moveaim_reticle_*_limit` |
| HUD in world | `vr_render_hud_in_world`, `vr_hud_forward`, `vr_hud_display_ratio`, `vr_hud_max_fov`, `vr_hud_axis_lock_to_world`, `vr_hud_never_overlay` |
| view model | `vr_viewmodel_offset_forward`, `vr_viewmodel_offset_forward_large`, `vr_viewmodel_translate_with_head`, `vr_first_person_uses_world_model` |
| projection / eyes | `vr_projection_znear_multiplier`, `vr_stereo_swap_eyes`, `vr_stereo_mono_set_eye`, `vr_translation_limit` |
| zoom | `vr_zoom_multiplier`, `vr_zoom_scope_scale` |
| windowing / debug | `vr_force_windowed`, `vr_debug_remote_cam` + its six position/target cvars |

**The UI switch survives too:** `bin\GameUI.dll` carries `#GameUI_VRMode`, `VRModeLabel`,
`#GameUI_VRModeRelaunchMsg`, and writes `mat_enable_vrmode %d`.

**The engine side:** `mat_enable_vrmode`, `mat_vrmode_adapter` (also in `materialsystem.dll`),
`ForceStereoRenderToFrameBuffer`, and the interface names `SourceVirtualReality001` /
`ClientVirtualReality001`.

**Exactly one thing is missing — `sourcevr.dll`.** It is named by `engine.dll`, is not shipped, and is
absent from this whole machine. Two strings say what it does and what it must satisfy:

```
Unable to get VRModeAdapter from OpenVR. VR mode will not be enabled. Try restarting and then enabling VR again.
Preventing connections to secure servers because sourcevr.dll is not signed.
```

⚠️ **The signature check is a real constraint**, not a footnote: a hand-built replacement will trip
it. The stated consequence is only "no secure servers", which for a single-player game sounds
harmless — but that is what the message says, not what has been observed `[hypothesis]`.

⚠️ **None of this has been run.** Every item above is a name in a binary. Whether the code paths
survived a decade of Source updates, and whether `mat_enable_vrmode 1` does anything without
`sourcevr.dll`, is unknown until a launch.

## 9b. The Half-Life 2 VR mod is installed on the dev PC — and is NOT this project's shape

`D:\SteamLibrary\steamapps\common\Half-Life 2 VR` `[inferred-static 2026-09-14]`. It ships its own
`client.dll`/`server.dll` (a Source-SDK rebuild with its own `#hlvr_GameUI_Comfort_*` locomotion and
comfort options) and its `bin\d3d9.dll` is **DXVK** (imports `vulkan-1.dll`), not a VR shim.

⭐ HLVR's route needs the game's **source code**, which Portal does not have. Portal's route is the
opposite — the VR client already exists in the shipped DLL and the module underneath it is missing.
Two different problems; do not copy HLVR's shape onto this project.

## 11. Dead ends & false leads (save future time)
- none yet.

## 12. Open risks toward the North Star
- ⭐ **Valve's own leftover VR mode is still in the code:** `engine.dll` and `client.dll` both contain OpenVR messages ("VR mode will not be enabled") and references to `sourcevr`, but no `sourcevr.dll` ships `[inferred-static 2026-09-13]`. Whether that code can still be switched on is unknown, and worth checking first.
- Community Source VR projects exist; whether any covers Portal is for `/gr` to establish, not assumed here.

## Inbox folds, 2026-09-29

**The `SourceVirtualReality001` interface shape is in Valve's SDK (`/gr` 2026-09-17).** Source SDK 2013 `src/public/sourcevr/isourcevirtualreality.h`, `IAppSystem`-derived, 22 methods after the base block `[reported]`; confirm the order against `engine.dll` before trusting it for Portal `[hypothesis]`. Topic: `external-research/topics/2026-09-17-sourcevr-interface-shape-and-portal1vr-prior-art.md`.

**Watch: portal1vr now ships builds (`/gr` 2026-09-29).** BowmanFox published six pre-release builds on 2026-09-23/24 (newest `v2026.09.24-bowman.1`), with portal-aware camera and collision work; no push since 2026-09-24 `[reported]`. The pause stands; WATCHING.md updated.

