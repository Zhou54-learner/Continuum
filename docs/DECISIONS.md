# Continuum — Lightweight Decision Log

A concise log of choices that affect the architecture. **These are proposed initial baselines**, not commitments to keep every implementation detail forever. Record a new entry only for a consequential choice.

## D-001 · Modular monolith first
**Choice:** One Python application with logical identity, memory, persona and agent boundaries. No microservices or empty extension scaffolding.

**Why:** The first goal is a reliable end-to-end memory loop, not distributed operations.

**Revisit:** Real performance/isolation evidence or independent deployment requirements.

## D-002 · SQLite as initial source of truth
**Choice:** Local SQLite for event provenance and memory records. Derived search indexes can be rebuilt. Avoid choosing a graph/vector service before an evaluation shows it is needed.

**Why:** Fewer moving parts, simple inspectability, low migration cost at MVP scale.

**Revisit:** Improve tokenization, ranking or **derived indexes** for poor retrieval (notably Chinese). Reconsider SQLite as primary storage only for observed write contention or deployment/concurrency requirements.

## D-003 · Persona ≠ Memory ≠ Permissions
**Choice:** Editable personality is stored separately from user facts and authorization decisions.

**Why:** A change in style must not change the user's history or tool privileges.

**Revisit:** Preserve the separation even if representation changes.

## D-004 · One verified owner and CLI before channels
**Choice:** First verify cross-session memory for one configured owner. Social adapters, multi-account mapping and robots come later.

**Why:** Platform integrations would obscure memory design defects.

**Revisit:** After MVP acceptance checks pass.

## D-005 · Evidence before abstraction
**Choice:** Add abstraction layers or dependencies only for an observed need. The second real provider/channel is a natural moment to discover a shared interface.

**Why:** Premature frameworks increase maintenance without improving the first vertical slice.

**Revisit:** Whenever concrete duplication or testability problems emerge.

## D-006 · Exclusive source excerpts in MVP
**Choice:** Each memory revision owns one minimum source excerpt; only content-free origin references may be shared. Independently deletable facts need separable evidence; otherwise save one disclosed deletion unit. Deleting an excerpt cascades to its owning memory and all revisions/excerpts.

**Why:** Sharing source text creates ambiguous cascading deletion, while copying full messages can retain deleted facts in another memory's evidence.

**Revisit:** A real need for shared evidence or multiple independent sources, together with explicit redaction and deletion semantics.

## D-007 · Local read permission is separate from model processing permission
**Choice:** Memory defaults to local processing; cloud use requires an explicit per-memory grant for the configured provider/endpoint. Trusted request context carries the actual destination. All model-bound context, including sources and history, must satisfy that grant as well as requester/audience/purpose rules.

**Why:** A private conversation with the owner does not by itself authorize transmission to any provider. A changed endpoint must not silently inherit access.

**Revisit:** A second concrete destination or batch-grant workflow, preserving explicit permission and complete-input denial tests.
