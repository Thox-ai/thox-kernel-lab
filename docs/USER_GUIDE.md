# thox-kernel-lab — User Guide

Thox Kernel Lab is experimental Rust microkernel research: agents as
first-class kernel objects, tiered memory, in-kernel mesh cognition.
It is built clean-room, from design specifications only. No shipping
device runs Lab code, and none is promised to — shipping devices run
ThoxOS Core on linux-thox.

Your AI. Your Data. Your Rules.™

## What is in v0.1.0

Specifications only. There is no kernel code in this release.

- `specs/SPEC-INDEX.md` — the extracted specification set S1–S9:
  syscall ranges (S1), design principles (S2), tiered memory T0–T6
  (S3), device fabric framing (S4), mesh cognition (S5), compliance
  at the boundary (S6), the agent object lifecycle and capability
  model (S7), invariants as behavioural requirements (S8), and
  scheduler behaviour (S9). One item is openly unfinished: S9.3 names
  the inference-waiter wake policy without defining it; defining it
  is an open specification task.
- `CLEAN_ROOM_CHARTER.md` — the provenance rules every Lab artefact
  follows: specifications first, zero code ported from any prior THOX
  kernel repository, a provenance log entry per crate naming the
  specification it was implemented from.

## How to read the specifications

Start with `specs/SPEC-INDEX.md`. Each specification cites the design
document and section it was extracted from. Requirements are stated
behaviourally; where a source design names a mechanism without
defining it, the specification says so instead of inventing a
definition.

## What comes next

Implementation crates (`lab-abi`, `lab-util`, `lab-tier-memory`)
follow the Charter's specification-first order and ship in later
releases as they land, each with its provenance log and passing host
tests. The Lab has no release SLA, no device commitment, and no
backer promise attached.

License: Apache-2.0 (specifications and docs CC BY 4.0). See
[LICENSE](../LICENSE) and [NOTICE.md](../NOTICE.md).
