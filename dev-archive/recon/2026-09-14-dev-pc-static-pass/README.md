# 2026-09-14 — Portal, dev-PC static pass (NO LAUNCH)

**Machine:** dev PC `DESKTOP-V8GTSIR`. **Install:** `D:\Program Files (x86)\Steam\steamapps\common\Portal`,
Steam app 400, `StateFlags=4` (fully installed) `[inferred-static 2026-09-14]` — the home PC's
2026-09-13 note said "installed on the home PC only, as far as this session knows"; **it is installed
here too**, so this project's `[PD]` rows can run on this machine.

**The game was not launched. Nothing in this folder was produced by running anything** — it is all
PE headers and string extraction from files on disk.

## Files here

| File | What it is |
| --- | --- |
| `pe-imports.txt` | PE header + full import table for `hl2.exe`, `bin\engine.dll`, `portal\bin\client.dll` |
| `vr-cvars-client.txt` | the 36 `vr_*` console variables found in the shipped `client.dll` |
| `leftover-vr-strings.txt` | every VR-related string in `engine.dll`, `client.dll`, `GameUI.dll`, `materialsystem.dll` |
| `install-listing.txt` | contents of `bin\` |

## The headline: Valve's VR mode is not a leftover fragment — it is the whole client half

The 2026-09-13 board row asked to "map Valve's leftover VR-mode code … and whether any of it can
still run". The answer is more encouraging than "leftover" suggested.

**`portal\bin\client.dll` (linked 2025-05-27) contains the complete Source VR *client*
implementation** `[inferred-static 2026-09-14]`:

- the class itself — `CClientVirtualReality`, `IClientVirtualReality`, and its whole
  `CBaseAppSystem`/`CTier1AppSystem`/`CTier2AppSystem`/`CTier3AppSystem` registration chain;
- the original source path is still in the binary:
  `c:\buildslave\rel_singleplayer_win32\build\src\game\client\client_virtualreality.cpp`;
- **36 `vr_*` console variables**, covering far more than a stereo toggle — head-relative aiming
  (`vr_moveaim_mode`, `vr_aim_yaw_offset`, the reticle pitch/yaw limits), HUD placement in world
  space (`vr_hud_forward`, `vr_hud_display_ratio`, `vr_hud_axis_lock_to_world`,
  `vr_render_hud_in_world`), view-model handling (`vr_viewmodel_offset_forward`,
  `vr_viewmodel_translate_with_head`, `vr_first_person_uses_world_model`), the near-plane fix
  (`vr_projection_znear_multiplier`), and eye handling (`vr_stereo_swap_eyes`,
  `vr_stereo_mono_set_eye`);
- activation commands `vr_activate` / `vr_deactivate` / `vr_toggle`;
- a menu-drawing path written for VR: `CClientVirtualReality::DrawMainMenu`;
- a per-mode config hook: `exec sourcevr_%s.cfg`.

**`bin\GameUI.dll` still carries the user-facing switch** — `#GameUI_VRMode`, `VRModeLabel`,
`#GameUI_VRModeRelaunchMsg` ("relaunch" implies VR mode is chosen, then applied on restart), and it
writes `mat_enable_vrmode %d` `[inferred-static 2026-09-14]`.

**`bin\engine.dll` holds the engine side:** `mat_enable_vrmode`, `mat_vrmode_adapter`,
`ForceStereoRenderToFrameBuffer`, and the two interface names the engine asks for —
`SourceVirtualReality001` and `ClientVirtualReality001`.

## What is actually missing, and it is exactly one thing

`engine.dll` names **`sourcevr.dll`** and does not ship it. Two strings pin down what it is for:

```
Unable to get VRModeAdapter from OpenVR. VR mode will not be enabled. Try restarting and then enabling VR again.
Preventing connections to secure servers because sourcevr.dll is not signed.
```

So `sourcevr.dll` is the module that talks to OpenVR and publishes the `SourceVirtualReality001`
interface; the engine loads it by name, asks it for the adapter, and gives up with that message when
it cannot. **`sourcevr.dll` is absent from this entire machine** — checked across both Steam
libraries `[inferred-static 2026-09-14]`.

⚠️ **Second string is a real constraint, not a footnote:** the engine expects that DLL to be
**signed**, and says so. A hand-built replacement will trip that check. For a single-player game the
stated consequence is only "no secure servers", which sounds harmless — but it has not been tested
and the check may do more than the message says. `[hypothesis]`

## What this does NOT establish

- **That any of it still runs.** Every fact above is a name in a binary. Whether the engine's VR code
  paths survived a decade of Source updates intact, and whether `mat_enable_vrmode 1` does anything
  at all without `sourcevr.dll`, is unknown until the game is launched.
- **What `SourceVirtualReality001`'s actual vtable looks like.** The interface name is present; its
  shape is not, and would have to be recovered by disassembly or from public Source SDK headers
  (`/gr`'s to find, not this session's to assume).
- **Whether `-vr` exists as a launch switch.** It does **not** appear in `engine.dll`'s switch
  strings; the switches present are `-console -dev -window -width -height -insecure`
  `[inferred-static 2026-09-14]`. VR looks to be turned on by cvar and menu, not by command line.

## Binary facts that matter for tooling

| | |
| --- | --- |
| Bitness | **32-bit** throughout `[inferred-static 2026-09-14]` |
| `hl2.exe` | linked 2024-11-13, ASLR **on** |
| `bin\engine.dll` | linked 2024-12-09, ASLR **on** |
| `portal\bin\client.dll` | linked 2025-05-27, ASLR **on** |
| Renderer modules present | `shaderapidx9.dll` (D3D9), `shaderapivk.dll` + `dxvk_d3d9.dll` (Vulkan via DXVK), `shaderapiempty.dll` |

⚠️ **ASLR is ON for all three** — unlike Hard Reset, Prototype and Dead Space 2, which all load at a
fixed base. Any address written down for Portal is only valid within one run, so tooling here must
resolve by module base + offset, never by absolute address.

## A working reference implementation is on this machine

**The Half-Life 2 VR mod is installed** at `D:\SteamLibrary\steamapps\common\Half-Life 2 VR`
`[inferred-static 2026-09-14]`. Worth knowing how it is built, because it is **not** the route
described above: it ships its own `client.dll` and `server.dll` (a Source-SDK rebuild with its own
`#hlvr_GameUI_Comfort_*` locomotion and comfort options), and its `bin\d3d9.dll` is **DXVK**
(imports `vulkan-1.dll`), not a VR shim.

⭐ **That is the useful distinction.** HLVR's route needs the game's source code, which Portal does
not have. Portal's route is the opposite: **the game's own VR client already exists in the shipped
DLL**, and what is missing is the one module underneath it. Two different problems; do not copy
HLVR's shape onto this project.
