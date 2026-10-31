# Specification Index — Thox Kernel Lab

Specifications are extracted from design documents only, per `CLEAN_ROOM_CHARTER.md`.
Each entry cites its source document and section. Implementation starts from
these specifications, never from prior source code.

## Extracted (from `thoxos-kernel` DESIGN.md, read 2026-10-29)

### S1. Syscall ranges (DESIGN.md, ADR-0005)
Three ranges: THOX-native time/mesh (1–127), POSIX-shaped (128–511),
THOX-native compliance/persona/device (512–767). Linux compatibility, if any,
lives in a userspace shim; the kernel stays Linux-agnostic (ADR-0008).

### S2. Design principles (DESIGN.md, Design Philosophy)
1. Agents are first-class kernel objects; inference is a scheduled syscall,
   with kernel-enforced agent memory budgets and capability access.
2. Fail-closed everywhere.
3. `no_std` throughout; no heap allocation in interrupt paths.

### S3. Tiered memory (DESIGN.md, crate map: thox-tier-memory)
Seven tiers, T0 (SRAM) through T6 (mesh-remote), with placement hints at
allocation time. (Note: the userspace KV programme uses a separate T0–T4
vocabulary for `thox-kvd`; Lab must not conflate the two in public docs.)

### S4. Device fabric framing (DESIGN.md, crate map: thox-fabric)
Framed, kernel-owned wire format; 40-byte header; ABI stable per major version.

### S5. Mesh cognition (DESIGN.md, ADR-0002)
Agent-to-agent messaging is a kernel-mediated primitive, not a userspace
library. Typed frames, topics, capabilities, route decisions; inline payload
limit recorded in the design documents (≤256 bytes).

### S6. Compliance at the boundary (DESIGN.md, ADR-0006)
A compliance preflight gate runs at agent creation and at capability-sensitive
syscall boundaries.

## To extract before implementation passes S6

- Agent object lifecycle and capability table semantics (from AGENT_MODEL.md).
- Invariants I1–I5 restated as behavioural requirements (from INVARIANTS.md).
- Ring-3 entry/resume requirements stated as *requirements* (the Defect 5
  lesson — in-place clear, stack discipline — belongs in the specification as
  a constraint on any implementation, with its root-cause narrative as the
  rationale; the fix itself is re-derived, not copied).
- Scheduler behaviour: preemptive, timer-driven, wait-queues, inference-waiter
  wake policy (from DESIGN.md version history as requirements).

## Provenance of this index

Compiled 2026-10-29 by Sadie Weaving from DESIGN.md only. No source files were
consulted. Charter mode: specification extraction step (Charter §4.1).

---

## Round 2 extraction (2026-10-30)

This round discharges the four "To extract" items listed above. Requirements
are behavioural restatements of the cited design documents only; where a
source document names a mechanism without defining it, the requirement says
so and the definition is left as an implementation-time specification task.

### S7. Agent object lifecycle and capability-table semantics (from AGENT_MODEL.md, Part 1)

- **S7.1 Creation.** An agent comes into existence only through the agent-creation
  syscall, which allocates an entry for it in the kernel's global agent object
  table. (AGENT_MODEL.md, "Agent as a kernel object")
- **S7.2 Declared spec at creation.** Creation takes a declared specification:
  a memory budget and an initial capability set accompany the create request;
  there is no create path with undeclared budget or capabilities.
  (AGENT_MODEL.md, "Syscall surface (thox-abi)", AgentCreate row; "Agent as a
  kernel object")
- **S7.3 Budget admission.** The declared memory budget is enforced at admission
  time, and an agent must never be able to exceed its declared footprint.
  (AGENT_MODEL.md, "Agent as a kernel object")
- **S7.4 Compliance gate at creation.** Agent creation is gated by the
  compliance preflight (HIPAA/GDPR/SOC2/PCI/ISO27001 family of checks named in
  the source); creation is the checkpoint, not a post-hoc audit.
  (AGENT_MODEL.md, "Workspace crates", thox-compliance row)
- **S7.5 Per-agent capability table.** Every agent owns a capability table, and
  its syscall access is checked against that table's rights; an agent has no
  syscall reach beyond what its table grants. (AGENT_MODEL.md, "Agent as a
  kernel object"; "Capability rights")
- **S7.6 Rights semantics.** The rights vocabulary is closed and each right
  governs exactly one domain: READ governs reading kernel-managed shared
  memory segments; WRITE governs writing shared segments or device memory;
  EXEC governs executing in Ring-3 and is required for agent task entry;
  INFER governs submitting inference requests; MESH governs sending and
  receiving mesh frames; COMPLIANCE_AUDIT governs reading compliance gate
  results; PERSONA_BIND governs binding the calling agent to a persona
  profile. An operation whose governing right is absent from the caller's
  table must be denied. (AGENT_MODEL.md, "Capability rights")
- **S7.7 Destruction and reclaim.** Destroying an agent reclaims its object
  table entry and releases both its capability table and its memory budget
  back to the kernel. (AGENT_MODEL.md, "Syscall surface (thox-abi)",
  AgentDestroy row)
- **S7.8 Agent messaging.** A live agent can post a mesh-addressed message to
  another agent through the in-kernel send operation; message transfer is
  kernel-mediated and zero-copy in intent. (AGENT_MODEL.md, "Syscall surface
  (thox-abi)", AgentSend row)
- **S7.9 Blocking inference submit.** Submitting an inference request blocks
  the caller until its inference ticket completes. (AGENT_MODEL.md, "Syscall
  surface (thox-abi)", InferRequest row)
- **S7.10 Non-blocking inference poll.** A caller can poll for a pending
  inference result without blocking. (AGENT_MODEL.md, "Syscall surface
  (thox-abi)", InferPoll row)
- **S7.11 Deterministic inference order.** The inference scheduler issues
  tickets and runs models in FIFO order with fixed time quanta; inference
  execution order is therefore deterministic for a given submission order.
  (AGENT_MODEL.md, "Agent as a kernel object")
- **S7.12 Layout-stable ABI.** The agent and inference data exchanged across
  the kernel/userspace boundary (agent spec, inference request and response,
  rights) uses layout-stable types, stable per major version. (AGENT_MODEL.md,
  "Workspace crates", thox-abi row)

### S8. Invariants restated as behavioural requirements (from INVARIANTS.md)

Source note: INVARIANTS.md states exactly five invariants, I1–I5. Its closing
"Open" section still describes the resume-after-Exit defect as unsolved while
I3 records that defect's root cause; the requirements below treat the
invariants as binding regardless of that status discrepancy, and no
implementation detail from the document is carried over.

- **S8.1 Change control (preamble).** No requirement in this section may be
  removed or reverted without boot-level (QEMU transcript) evidence that the
  alternative works; static validation alone does not discharge an invariant.
  (INVARIANTS.md, preamble)
- **S8.2 I1 — Boot stack sizing.** The early boot stack reservation must be
  4 MiB. Rationale: user images are materialised through this stack during
  early launch, and a small (128 KiB class) reservation silently overflows and
  hangs the boot before any console output appears. (INVARIANTS.md, "I1 - Boot
  stack must be 4 MiB")
- **S8.3 I2 — Synchronised frame allocator.** The frame allocator must be
  held behind a spin lock, never as unsynchronised static mutable state, and
  every access must go through the lock. Rationale: unsynchronised static
  mutable access is unsound once interrupts are live. (INVARIANTS.md, "I2 -
  FRAME_ALLOCATOR must be a SpinLock, not static mut")
- **S8.4 I3 — Images install in place.** A user image must be materialised
  directly into the task slot's own statically owned storage, and clearing a
  slot must clear the image in place; an image must never be constructed or
  moved by value through launch, spawn, or teardown paths. Rationale: an
  image is page-scale (pages × 4 KiB, ~256 KiB class), and a by-value
  temporary overflows the small per-task trap stack at spawn time.
  (INVARIANTS.md, "I3 - Images install in place, never by value")
- **S8.5 CONSTRAINT — Ring-3 resume stack discipline (the Defect 5 lesson).**
  On the exit/teardown and resume path, task-slot reset MUST clear the user
  image in place and MUST NOT place any page-scale temporary on the
  syscall/scheduler stack. Rationale, stated as the source records it: a
  by-value image temporary created during exit teardown overflowed the 64 KiB
  syscall/scheduler stack and zeroed the statics linked directly below it,
  including the interrupt descriptor table; the next timer interrupt then
  failed delivery and escalated to a triple fault when the scheduler resumed
  the surviving task after an exit. Any implementation must be reviewable
  against this constraint without reference to the retired implementation;
  the corrective code itself is re-derived, not carried over. (INVARIANTS.md,
  "I3 - Images install in place, never by value"; "Open: defect 5
  (resume-after-Exit triple fault)")
- **S8.6 I4 — Resume state consistency.** Whenever a task's execution state
  is saved at a syscall boundary, every saved representation of its
  instruction pointer and stack pointer must be updated together, so a
  resumed task continues at the instruction after the syscall rather than
  relaunching at its entry point; the syscall path and the timer-preemption
  path must maintain the same saved-state discipline. (INVARIANTS.md, "I4 -
  Syscall dispatch must sync gprs with pc/sp")
- **S8.7 I5 — Non-interactive Ring-3 smoke paths.** The kernel must provide
  its two named terminal autorun keys as the non-interactive user-mode (CPL3)
  smoke paths; they are the required reproduction and regression vehicle for
  the resume-after-exit behaviour constrained by S8.5. (INVARIANTS.md, "I5 -
  Terminal Ring-3 autorun keys")

### S9. Scheduler behaviour requirements (from DESIGN.md)

- **S9.1 Preemptive, timer-driven production scheduler.** The production
  scheduler is preemptive and driven by the hardware timer; tasks do not run
  until they voluntarily yield as their only means of losing the CPU.
  (DESIGN.md, "Scheduling and process model", thox-scheduler row)
- **S9.2 Wait-queues.** The production scheduler provides wait-queues: a task
  blocked on an event is parked in a wait-queue and made runnable by a wake
  event, rather than spinning. (DESIGN.md, "Scheduling and process model",
  thox-scheduler row)
- **S9.3 Inference-waiter wake policy.** The production scheduler implements
  a defined wake policy for tasks waiting on inference completion. The source
  names this policy but does not define its algorithm; defining it (wake
  order, fairness against non-inference waiters) is an open specification
  task for the implementer, to be recorded in this index before the scheduler
  is implemented. (DESIGN.md, "Scheduling and process model", thox-scheduler
  row)
- **S9.4 Test scheduler is not the production contract.** A separate
  scheduler abstraction with a cooperative round-robin implementation exists
  as the test implementation; production preemption behaviour (S9.1–S9.3)
  must not be inferred from, or degraded to, the cooperative test behaviour.
  (DESIGN.md, "Scheduling and process model", thox-sched row)
- **S9.5 Agent-level cooperative yield.** Alongside the preemptive task
  scheduler, the core agent scheduling layer provides cooperative yield for
  agents and operates the in-kernel agent message bus. (DESIGN.md, "Agentic
  layer", thox-agentic-core row)
- **S9.6 Milestone traceability.** Scheduler primitives, wait-queues, and the
  inference-waiter wake policy are recorded as having landed across the
  MVP-7 to MVP-16 milestone band (versions v1.1.1–v1.1.10); a Lab
  implementation claiming parity with the design must cover the whole band's
  behaviours, not only the final state. (DESIGN.md, "Version History (by MVP
  milestone)")
- **S9.7 Determinism and auditability.** Inference is a scheduled operation
  under kernel control specifically so that it is deterministic and
  auditable; scheduling behaviour that makes inference order or timing
  non-reproducible for a given workload violates the design principle.
  (DESIGN.md, "Design Philosophy", principle 1)

## Provenance of the Round 2 addition

- Documents read (design documents only, from the retired thoxos-kernel
  snapshot): DESIGN.md (Design Philosophy; Workspace Crate Map — Scheduling
  and process model, Agentic layer; Version History), docs/AGENT_MODEL.md
  (Part 1 — Kernel-Level Agent Model, in full), docs/INVARIANTS.md (in full).
- Date: 2026-10-30.
- Extractor: Team C (specification extraction step, Charter section 4.1).
- Attestation: No source files were opened for this extraction — no .rs
  files, no Cargo manifests, no src/ directories, and no implementation
  files of any kind. Where a design document contained code-shaped material
  (interface signatures, named fields, command examples), only the API shape
  or behavioural fact was restated as a requirement; no implementation was
  copied, adapted, or paraphrased.
- Charter section 3 expects PROVENANCE.md per crate, not per folder; the
  specs folder is not a crate, so no specs/PROVENANCE.md is created and this
  note in SPEC-INDEX.md is the provenance record for the extraction step.
