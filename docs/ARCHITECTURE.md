# Continuum — Architecture Baseline

**Status:** Proposed foundation (2026-10-22). Revise when implementation supplies evidence, not to anticipate hypothetical scale.

## 1. Purpose and boundaries

Continuum is a **user-owned persistent AI runtime**, not just a chat bot or a vector database. An LLM is a replaceable inference component; identity, memory, persona configuration, and tool permissions are managed by the application.

The initial architecture is a **modular monolith**: one deployable application, clear internal responsibilities, and adapters at external boundaries. There is no need for microservices, a message broker, or a plugin platform at the outset.

```text
Local CLI (first) / future channel adapters
                |
        Application workflow
          /    |    |    \
     Identity Memory Persona Agent
          \    |    |    /
     Storage / LLM / tool adapters
```

This is a responsibility map, **not** an instruction to generate empty packages or a hierarchy of managers.

## 2. Responsibilities

| Area | Owns | Must not own |
| --- | --- | --- |
| Application | Input-to-output sequence and error presentation | Memory semantics or SQL internals |
| Identity | Internal person IDs and explicitly verified platform-account bindings | Guessing cross-platform identity from message content |
| Memory | Event provenance, candidate facts, versioning, retrieval, correction and deletion | Direct platform I/O or persona instructions |
| Persona | Editable identity expression and context-dependent presentation | Authorization policy or user-fact storage |
| Agent | Context assembly, LLM call, small authorized tool loop when needed | Raw persistence details or unguarded disclosures |
| Adapters | SQLite, model APIs, CLI, later social/device protocols | Business rules about who may read what |

These are **logical boundaries**. Introduce code folders only when they contain useful working code.

## 3. Minimal data concepts (not a frozen schema)

- **Person:** stable internal identifier. An owner and third parties are distinct people.
- **Account binding:** `platform + external_account_id -> person_id`, with verification method, status and timestamps. Session/channel IDs are not person IDs.
- **Event:** observed interaction or action, with actor, source channel, timestamp and provenance. Retention can be limited; raw transcripts must not be kept forever by default.
- **Memory:** a derived or explicitly entered claim/episode linked back to source event(s), with subject, type, confidence/status, validity period and visibility/authorization metadata.
- **Persona revision:** versioned editable settings, stored separately from factual memories and operational permissions.

Relations, procedural memories, embeddings and time-aware graphs are future representations **only when queries justify them**. The persistent database is the source of truth; search indexes are rebuildable derivatives.

## 4. Memory lifecycle

1. Receive an authenticated envelope with source metadata; store only what retention settings permit.
2. Resolve the speaker to a known person/account binding or a separate unverified identity; never trust text saying "I am the owner".
3. Produce memory candidates (initially an explicit user action or a small extractor, not an autonomous rewriting system).
4. Compare with current memories. Update validity/status and provenance; do not erase contradictory historical evidence without a correction policy.
5. On a later request, retrieve relevant memories **after filtering by the current audience and purpose**, before adding any content to the LLM prompt.
6. Allow inspection, correction, export and deletion. Deletion must propagate to derived indexes and summaries; backups follow an explicit retention policy.

Source content is **data**, not an instruction capable of overriding identity, persona, policy or tools.

## 5. Access and agency boundaries

- Owner-only knowledge is private by default. A known owner account does not grant every channel permission to reveal owner-only memory.
- Third-party interactions may retain only necessary, purpose-limited context. Do not silently merge third-party accounts across platforms.
- Persona changes affect expression, **not** read permissions, stored facts, or executable privileges.
- Secrets belong in a credential store/environment, never in memory records, prompts, or version control.
- Outbound public messages, destructive system commands and physical-device actions require explicit permission gates. Add the implementation when such actions exist; do not build an empty general-purpose policy engine now.
- A future physical controller must enforce safe limits and emergency stop **outside** the LLM loop.

## 6. Extension seams

A future channel maps inbound platform messages into a small envelope (source, authenticated actor, conversation/session, timestamp, content) and sends generated responses back. The core must never import QQ/WeChat-specific objects.

Start with a single model adapter. Unify provider APIs **after** a second real provider needs the same operation. Start with SQLite and simple keyword/structured retrieval; add embeddings or temporal graphs only when measured retrieval failures warrant them.

## 7. Deferred by design

No preemptive Kafka/Redis/Postgres split, Graphiti integration, workflow engine, dynamic plugin registry, elaborate event bus, abstract factories, or autonomous social posting. Any future addition should name the current failure it addresses, the simplest alternative considered, and a small regression test.

## 8. Review triggers

Revisit this baseline when: (a) a second channel creates concrete adapter duplication; (b) a second model forces common provider semantics; (c) retrieval evaluation demonstrates need for vector/graph search; or (d) observed throughput/reliability exceeds a single-process SQLite design.
