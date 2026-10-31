# Clean-Room Charter — Thox Kernel Lab

**Decision:** G0b, Tommy Xaypanya, 2026-10-29 — Kernel Lab is built clean-room.
**Applies to:** every crate, document and test in `thox-kernel-lab`, from the first commit.
**Fleet precedent:** `thox-kvd` (clean-room Rust, no LMCache embedding), ThoxHarness verdict (concepts only, port zero code), ThoxHuman clean-room migration (boundary doc, zero foreign code in tree).

---

## 1. The rule

No source code is copied, adapted, transliterated or paraphrased from:

- the retired `thoxos-kernel` repository (any branch, any version),
- any other prior THOX kernel experiment,
- any third-party kernel whose license is not Apache-2.0 / MIT / BSD compatible with this project’s Apache-2.0 license (in particular: no AGPL-3.0 code, e.g. Maestro, in any form).

Reading *specifications* is allowed and required. Reading *source* of the retired kernel in order to reimplement it is not clean-room and is prohibited for implementers. A person who has recently studied retired kernel source on a given subsystem may write the *specification* for that subsystem; a different person implements from the specification. Where the fleet is too small for two people, a 30-day separation plus a specification written from design documents only is the fallback, and the provenance log must say which mode was used.

## 2. Allowed specification sources

- `thoxos-kernel` **design documents as specifications**: `DESIGN.md`, the ADR set, `docs/INVARIANTS.md` (as behavioural requirements), `docs/AGENT_MODEL.md`, the ecosystem map. These describe *what* the system does, in the open, and they are the Lab’s requirements input.
- Public standards and papers (syscall design, capability systems, microkernel literature).
- The implementer’s own knowledge of Rust, `no_std` development, and OS fundamentals.

If a design document contains a code excerpt longer than an interface signature, that excerpt is treated as source: it may define an API shape (types, function names in a trait), it may not be copied as an implementation.

## 3. Provenance log (mandatory, per crate)

Every crate carries `PROVENANCE.md` with one entry per work session:

```
## <crate> — <YYYY-MM-DD> — <implementer>
- Specification used: <doc + section>
- Mode: two-person | separation-fallback
- Attestation: No source code from thoxos-kernel or any prohibited source was
  copied, adapted or consulted during this implementation. Interfaces follow
  the specification only.
- Tests: <how behaviour was verified against the specification>
```

A crate without a current provenance log does not merge. The log is the Lab’s legal hygiene record; treat it with the same seriousness as the old kernel’s QEMU evidence.

## 4. First implementation order (specification-first)

1. **Spec extraction (no code):** turn the allowed documents into `specs/` — ABI types and invariants as requirements, each with a source-document citation. This step may be done by anyone, including people who know the old code, because its output *is* the specification.
2. **`lab-abi`** — ABI types from `specs/abi.md` only.
3. **`lab-util`** — ring buffer, alignment, ID allocation from general knowledge + spec requirements.
4. **`lab-tier-memory`** — tier model (T0–T6 concepts) from `specs/memory-tiers.md`.
5. Everything else only after 2–4 have provenance logs and passing host tests.

## 5. What success looks like

A Lab crate is done when: it builds `no_std`, its host tests pass against the specification, its provenance log is complete, and a reviewer who did not write it can trace every public behaviour to a specification line. Resemblance to the retired kernel’s *interfaces* is expected (they come from the same specification). Resemblance in implementation is a finding to be explained in review, not a goal.

## 6. Breach handling

If copied source is discovered in Lab at any point: the affected crate is removed, the incident is recorded in the crate’s provenance log successor and in the programme `DECISIONS.md`, and the crate is reimplemented under this charter. There is no “keep it and note it” option. This rule is what makes the Lab’s Apache-2.0 license statement trustworthy.
