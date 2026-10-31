# Handoff — thox-kernel-lab

Status: PUBLIC v0.1.0 released 2026-10-31 (programme decision D10).
Repository: https://github.com/Thox-ai/thox-kernel-lab — release:
https://github.com/Thox-ai/thox-kernel-lab/releases/tag/v0.1.0

Done:
- Specifications S1–S9 extracted under the Clean-Room Charter
  (rounds 1 and 2), including S7 lifecycle/capability model, S8
  invariants, S9 scheduler behaviour.
- Clean-Room Charter in tree; zero code ported (decision G0b).

Next:
- Define S9.3 (inference-waiter wake policy) as a specification task.
- Implementation crates (lab-abi, lab-util, lab-tier-memory) in
  Charter order, each with provenance log + host tests.

Key files: specs/SPEC-INDEX.md, CLEAN_ROOM_CHARTER.md.

Conventions: specification-first; DCO sign-off; no code is ever
ported from the retired kernel repository.

Blockers: none for the specification track.

Last verified: 2026-10-31, publish pass.
