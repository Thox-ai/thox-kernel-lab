# thox-kernel-lab

[![License](https://img.shields.io/badge/license-Apache--2.0-blue?style=flat-square)](LICENSE)
[![THOX.ai](https://img.shields.io/badge/THOX.ai-portfolio-05A451?style=flat-square)](https://thox.ai)

Thox Kernel Lab: experimental Rust microkernel research — agents as first-class kernel objects, tiered memory, in-kernel mesh. Clean-room implementation from design specifications. No shipping device runs this code, and none is promised to.

**Status:** v0.1.0 specifications package (2026-10-31). This package contains
the Lab's extracted specifications S1–S9 (`specs/SPEC-INDEX.md`) and the
Clean-Room Charter. It contains **no kernel code**: implementation crates
(`lab-abi`, `lab-util`, `lab-tier-memory`, …) follow the Charter's
specification-first order and ship in later releases as they land with
provenance logs and passing host tests.

## Experimental

This is a research project. It has no release SLA, no device commitment, and
no backer promise attached. Shipping THOX devices run ThoxOS Core on
`linux-thox`.

## Clean-room

Implemented from specifications only (see `CLEAN_ROOM_CHARTER.md`).
Zero code is ported from any prior THOX kernel repository (decision G0b,
2026-10-29).

## Quick start

```sh
git clone https://github.com/Thox-ai/thox-kernel-lab.git
# then read CLEAN_ROOM_CHARTER.md and specs/SPEC-INDEX.md — v0.1.0 is specifications only
```

## Documentation

- [User Guide](docs/USER_GUIDE.md)
- [Quickstart](docs/QUICKSTART.md)
- [Development](docs/DEVELOPMENT.md)
- [Previews](docs/previews/)
- Agent handoff: [.agents/HANDOFF.md](.agents/HANDOFF.md)

## Releases

Formal releases live on the
[Releases page](https://github.com/Thox-ai/thox-kernel-lab/releases).
v0.1.0 is the specifications release; its tarball and SHA-256 sidecar
are attached there.

## Governance

THOX.ai LLC is the steward of this repository. Changes to public contracts
(ABIs, local APIs, the free/paid line) require an RFC. The THOX trademark
policy applies: the license covers the code, not the THOX marks.
Publishing and announcement decisions for THOX programmes are made by
the THOX.ai founders.

## Research

The architecture, evidence boundaries and validation protocol for this work are documented in:

**ThoxOS and ThoxInux: Architecture, Evidence, and a Validation Protocol for Local-First AI** (public technical draft, version 0.2.0-review, 1 November 2026), Craig Ross and Tommy Xaypanya, THOX.ai LLC. Paper, source and evidence ledger: https://github.com/ttracx/thox-research-paper/tree/main/papers/thoxos — Release: https://github.com/ttracx/thox-research-paper/releases/tag/thoxos-0.2.0-review — DOI: https://doi.org/10.5281/zenodo.23876230

Status: public technical draft; experimental validation is incomplete; not peer reviewed.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE.md](NOTICE.md).
