# thox-kernel-lab — Development

## Method

Clean-room, specification-first (programme decision G0b, 2026-10-29).
Specifications are extracted from design documents only; no source
code from any prior THOX kernel repository is consulted or ported.
Every implementation crate will carry a provenance log entry naming
the specification it was implemented from, the implementer, and a
"no source copied" attestation.

## Architecture (as specified)

Agents are first-class kernel objects; inference is a scheduled
syscall with kernel-enforced agent memory budgets and capability
access. Seven memory tiers, T0 (SRAM) through T6 (mesh-remote).
Kernel-mediated agent-to-agent messaging with typed frames. A
compliance preflight gate at agent creation and at
capability-sensitive syscall boundaries. Fail-closed everywhere;
`no_std` throughout; no heap allocation in interrupt paths. Full
detail: `specs/SPEC-INDEX.md`.

## Conventions

- Default branch `main`. DCO sign-off (`Signed-off-by`) on every
  commit. AI-generated contributions must be disclosed in the PR
  description.
- A specification gap is closed by amending the specification first,
  never by letting an implementation invent behaviour. S9.3
  (inference-waiter wake policy) is the standing example: openly
  undefined until specified.

## Known limitations (v0.1.0)

- Specifications only; no code, no tests, nothing to run.
- S9.3 is undefined in the source design and remains open.

## Work log

- 2026-10-29: Specifications S1–S6 extracted under the Charter.
- 2026-10-30: Round 2 extraction (S7–S9) completed.
- 2026-10-31: Public release v0.1.0 (specifications) under programme
  decision D10.
