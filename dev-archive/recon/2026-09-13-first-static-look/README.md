# First static look (2026-09-13)

Read from the installed Steam copy on the home PC, without launching the game. Every claim
below is `[inferred-static 2026-09-13]` unless tagged otherwise: it comes from reading file headers
and strings, not from running anything.

- **Install:** `Portal`, 4.1 GB.
- **Identity:** Portal, Steam build. Launcher `hl2.exe` (linked 2024-11-13); `bin\engine.dll` 2024-12-09 and `portal\bin\client.dll` 2025-05-27, so this is the updated Source branch, not the 2007 original.
- **Engine:** Valve's Source engine `[reported]`, in its updated 2024 form. Game code is in `portal\bin\client.dll` and `server.dll`.
- **Binary:** **32-bit** throughout (`hl2.exe`, `engine.dll`, `client.dll`); no 64-bit binaries found in this install `[inferred-static 2026-09-13]`.
- **Renderer:** Direct3D 9 by default (`shaderapidx9.dll`), with a Vulkan path also shipped (`shaderapivk.dll`, `dxvk_d3d9.dll`) `[inferred-static 2026-09-13]`.
- **Protection:** Steam only `[inferred-static 2026-09-13]`.
- **Other files:** Standard Source layout (`hl2\`, `portal\`, `platform\`, VPK archives). Valve's own mod tools ship in `bin\` (Hammer, VRAD and others).

## Method

PE headers read with a short script: machine type, link timestamp, section names and sizes.
Then a case-insensitive search of each binary for renderer DLL names (`d3d9`, `d3d11`, `d3d12`,
`dxgi`, `vulkan-1`, `opengl32`), protection markers (`denuvo`, `securom`, `.bind`) and middleware
names. A string match shows a name is present in the file, not that the code path is used.

## Risks noted

- ⭐ **Valve's own leftover VR mode is still in the code:** `engine.dll` and `client.dll` both contain OpenVR messages ("VR mode will not be enabled") and references to `sourcevr`, but no `sourcevr.dll` ships `[inferred-static 2026-09-13]`. Whether that code can still be switched on is unknown, and worth checking first.
- Community Source VR projects exist; whether any covers Portal is for `/gr` to establish, not assumed here.
