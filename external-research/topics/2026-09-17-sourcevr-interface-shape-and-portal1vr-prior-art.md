# Valve's public header gives the SourceVirtualReality001 interface shape; portal1vr already runs Portal in VR

**Status:** 🆕 new · **Priority:** high — it answers an open `[PD]` row with Valve's own public
source, and there is a working Portal 1 VR mod.

## The interface shape is public

Valve's official **Source SDK 2013** repository carries `src/public/sourcevr/isourcevirtualreality.h`,
defining `SOURCE_VIRTUAL_REALITY_INTERFACE_VERSION "SourceVirtualReality001"` `[reported, read
2026-09-17 via the GitHub API]`. `ISourceVirtualReality` derives from `IAppSystem` and declares, in
this order (names only — we copy no code):

1. the `IAppSystem` block: `Connect`, `Disconnect`, `QueryInterface`, `Init`, `Shutdown`
2. `ShouldRunInVR`, `IsHmdConnected`, `GetViewportBounds(eye, x, y, w, h)`
3. `DoDistortionProcessing(eye)`, `CompositeHud(eye, ndcHudBounds[4], undistort, blackout, translucent)`
4. `GetMideyePose()` → `VMatrix`, `SampleTrackingState(playerFov, predictionSeconds)`
5. `GetDisplayBounds(VRRect_t*)`, `GetEyeProjectionMatrix(VMatrix*, eye, zNear, zFar, fovScale)`,
   `GetMidEyeFromEye(eye)` → `VMatrix`, `GetVRModeAdapter()`
6. `WillDriftInYaw()`
7. `CreateRenderTargets(IMaterialSystem*)`, `ShutdownRenderTargets()`,
   `GetRenderTarget(eye, which)` → `ITexture*`, `GetRenderTargetFrameBufferDimensions(w&, h&)`
8. `Activate()`, `Deactivate()`, `ShouldForceVRMode()`, `SetShouldForceVRMode()`

Enums: `VREye { Left, Right }`, `EWhichRenderTarget { RT_Color, RT_Depth }`.

⚠️ **The SDK header is not proof of Portal's build.** Portal's `engine.dll` may predate or postdate
the header; the vtable order must be checked against engine.dll's own calls before a replacement
`sourcevr.dll` is trusted `[hypothesis]`. The matching client-side caller is public too:
`src/game/client/client_virtualreality.cpp` in the same repository.

Only the official ValveSoftware repository was used. Other public copies of this header exist in
trees derived from leaked engine code; we do not use those.

## Working prior art: portal1vr

**portal1vr** by **BowmanFox** (forked from **Gistix/portal2vr**, which builds on the L4D2VR and
Portal 2 VR work) targets the Windows x86 Steam build of Portal, with 6DoF SteamVR tracking and
motion controllers `[reported]`. It hooks through a DXVK-based `d3d9.dll` and hooks `client.dll`
functions such as `TraceFirePortal`, keyed to one `client.dll` timestamp `[reported]`. Status per its
README: head tracking, movement and controller-aimed portals confirmed; a full campaign not yet
`[reported]`. fholger's `portal2vr` repository is part of the same lineage.

## Why it matters here

1. The `[PD]` row "recover the shape of `SourceVirtualReality001`" has a public answer to check
   against, rather than a from-scratch vtable recovery.
2. The dossier's route (Valve's dormant VR mode plus a replacement `sourcevr.dll`) is **a different
   route from portal1vr's** (d3d9 + client hooks). portal1vr is the fallback and the benchmark.

## Next step

Match the header's method order against the calls `engine.dll` makes through the interface pointer,
then run the `[FLAT]` `mat_enable_vrmode 1` test.

## Sources

- ValveSoftware, Source SDK 2013 — <https://github.com/ValveSoftware/source-sdk-2013> (`src/public/sourcevr/isourcevirtualreality.h`)
- BowmanFox, portal1vr — <https://github.com/BowmanFox/portal1vr>
- Gistix, portal2vr — <https://github.com/Gistix/portal2vr>
