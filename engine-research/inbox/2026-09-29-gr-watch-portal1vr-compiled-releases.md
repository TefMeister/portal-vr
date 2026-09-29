# /gr watch → engine-research: portal1vr now ships compiled release builds

From: `/gr` estate sweep, 2026-09-29. The watched mod is BowmanFox's **Portal 1 VR** (<https://github.com/BowmanFox/portal1vr>, a fork of Gistix/portal2vr). It was listed in `WATCHING.md` on 2026-09-23 as "playable, 6DoF, motion controls".

**What changed since the 2026-09-23 check** (GitHub API, read 2026-09-29) `[reported]`:

- **Six pre-releases published** between 2026-09-23 21:49 UTC and 2026-09-24 12:05 UTC. The newest is `v2026.09.24-bowman.1`, and all six are marked pre-release, so `releases/latest` returns 404. A commit on the same day says the release assets are limited to "the compiled build and checksum", so players can now download it instead of building it themselves.
- Commits in that window: gun and portal-shot effects lined up with the barrel, VR handedness controls, a support-hand grip, the head colliding correctly near portals, the camera staying aligned through portal crossings, a fix for a death-ragdoll crash, and runtime config defaults carried over on install.
- Last push was 2026-09-24 12:04 UTC. Nothing has been pushed since, so there are 5 quiet days.

**Meaning for us:** the mod has moved from "playable from source" to "downloadable builds, polished daily". That strengthens the pause. If we ever build on top of it, the "portal-aware" camera and collision work is what our own `sourcevr` route would otherwise have to solve. No action is needed; this is for the watch record (`WATCHING.md`, last-checked column).
