# Sourcevr interface shape is in valves sdk

**From:** `/gr` estate sweep, home PC, 2026-09-17.

**Answers:** `[PD]` row "recover the shape of the `SourceVirtualReality001` interface (vtable order and
signatures) — from public Source SDK headers or by disassembling engine.dll".

Valve's official Source SDK 2013 has it: `src/public/sourcevr/isourcevirtualreality.h`, interface
string `SourceVirtualReality001`, `IAppSystem`-derived, 22 methods after the `IAppSystem` block
(list in the topic) `[reported]`. ⚠️ Confirm the order against engine.dll's calls before trusting it
for Portal's build `[hypothesis]`. Also: **portal1vr** (BowmanFox) already runs Portal in VR via
d3d9 + client.dll hooks `[reported]`.

Suggested dossier change: move the row to "check header against engine.dll", and add portal1vr to §11.
Topic: `external-research/topics/2026-09-17-sourcevr-interface-shape-and-portal1vr-prior-art.md`
