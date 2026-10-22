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
| Application | Trusted request context (requester, audience, purpose, model destination), execution sequence and errors | Memory semantics or SQL internals |
| Identity | Internal person IDs and explicitly verified platform-account bindings | Guessing cross-platform identity from message content |
| Memory | Event provenance, revisions/deletion, and authorization-filtered reads including sources/history/export | Direct platform I/O or persona instructions |
| Persona | Editable identity expression and context-dependent presentation | Authorization policy or user-fact storage |
| Agent | Assemble **already-authorized** context, call LLM, add authorized tools if needed | SQL access, requester identity decisions, or widening permissions |
| Adapters | SQLite, model APIs, CLI, later social/device protocols | Business rules about who may read what |

These are **logical boundaries**. Introduce code folders only when they contain useful working code.

## 3. Minimal data concepts (not a frozen schema)

- **Person:** stable internal identifier. An owner and third parties are distinct people.
- **Companion:** stable AI instance ID, distinct from the owner person ID, chosen model, and editable persona revision. MVP has one companion.
- **Account binding:** `platform + external_account_id -> person_id`, with verification method, status and timestamps. Session/channel IDs are not person IDs.
- **Event:** observed interaction or action, with actor, source channel, timestamp and provenance. Retention can be limited; raw transcripts must not be kept forever by default.
- **Memory:** source-linked approved claim/episode, with stable memory ID, revision and reason, subject, recorded time, optional fact-validity interval, approval/dispute/deletion state and visibility. Approval authorizes use, not objective truth.
- **Persona revision:** versioned editable settings, stored separately from factual memories and operational permissions. MVP uses one authoritative local configuration.

Relations, procedural memories, embeddings and time-aware graphs are future representations **only when queries justify them**. The persistent database is the source of truth; search indexes are rebuildable derivatives.

## 4. Memory lifecycle

1. Receive an authenticated envelope with source metadata; store only what retention settings permit.
2. Resolve the speaker to a known person/account binding or a separate unverified identity; never trust text saying "I am the owner".
3. Create an **explicitly approved** memory from a user action. Automatic extraction/approval is not part of MVP; a later extractor may only propose candidates for review.
4. Compare with current memories. Update validity/status and provenance; do not erase contradictory historical evidence without a correction policy.
5. Memory filters all reads by trusted requester, audience and purpose **before** exposing content to Agent or the model; the same check applies to history, source preview and export.
6. Allow inspection, correction, export and deletion. Deletion must propagate to derived indexes and summaries; backups follow an explicit retention policy.

### Revision and truth-status rules

An **Event** proves that a statement was observed, not that it is objectively true. Keep approval, time validity, dispute and deletion separate rather than encoding them as one overloaded status. Revisions have a stable memory ID, revision ID, reason, recorded time and optional factual validity interval.

- **Fact changes over time:** “I lived in Hangzhou in June and moved to Shanghai in September.” The old claim remains valid for June, but Shanghai is current.
- **Correction of an error:** “I was mistaken: I lived in Shanghai even in June.” The old Hangzhou claim was an **incorrect assertion**, not a valid historical residence.
- **Unresolved conflict:** “A says Li lives in Hangzhou; B says Shanghai.” Preserve source-attributed disputed claims; do not let latest write, similarity or model confidence silently pick the truth.

MVP corrections target a specific memory ID; no automatic semantic contradiction detector is promised. Default recall selects approved, nondeleted, currently applicable revisions. Authorized history can show corrected assertions with clear error labels.

### Withdrawal, deletion and provenance

- **Withdraw:** cease treating an assertion as current, while keeping clearly labeled owner-visible history.
- **Delete memory:** remove memory and revision **content** from current reads, history, exports, derived indexes/summaries and retained context caches. Retained source text must not recreate the deleted memory in later rebuilds. A content-free tombstone (ID, time, deletion status) is permissible for non-resurrection.
- **Delete source:** in MVP, deleting the sole source cascades to its dependent memory. Keep only minimum source evidence for each explicit save, not full transcripts by default. A memory deletion also removes the corresponding source text; metadata may honestly say “source content removed” without pretending the text remains auditable.
- **Guarantee limits:** no app-managed backups in MVP; user exports and copies already transmitted to model providers are outside local deletion guarantees. SQLite logical deletion is not forensic secure erasure. Future restore/backup logic must respect deletion markers.

Source content is **data**, not an instruction capable of overriding identity, persona, policy or tools.

## 5. Access and agency boundaries

Application constructs a **trusted request context** from actual entry-point authentication/configuration: requesting actor, session, **output audience/visibility**, purpose, and permitted model processing destination. The person discussed in a memory is not necessarily the requester or audience. In local MVP, control of the configured local account is the only owner trust source. Synthetic owner/public, guest and missing-context cases may not read owner-private content; do not build a general-purpose PolicyEngine.

**Memory owns the single authorization rule** for retrieval, inspection, revision history, source previews and exports. Agent only receives authorized content and cannot broaden access. The rule must also cover stored history snippets and future summaries; prompts alone cannot enforce this boundary.

A local SQLite database does not imply local inference. If configured to use a cloud model, approved context is sent to that provider. The application must document the destination and transmitted categories of data; local deletion cannot retract requests already sent externally.

- Owner-only knowledge is private by default. A known owner account does not grant every channel permission to reveal owner-only memory.
- Third-party interactions may retain only necessary, purpose-limited context. Do not silently merge third-party accounts across platforms.
- Persona changes affect expression, **not** read permissions, stored facts, or executable privileges.
- Secrets belong in a credential store/environment, never in memory records, prompts, or version control.
- Outbound public messages, destructive system commands and physical-device actions require explicit permission gates. Add the implementation when such actions exist; do not build an empty general-purpose policy engine now.
- A future physical controller must enforce safe limits and emergency stop **outside** the LLM loop.

## 6. Extension seams

A future channel maps inbound messages into a small envelope (source namespace/message ID, authenticated actor, session, **audience**, timestamp, content) and sends responses back. Actor/audience must come from the trusted adapter, not from a sender's text. The core must never import QQ/WeChat-specific objects.

Start with a single model adapter. Unify provider APIs **after** a second real provider needs the same operation. Start with SQLite and simple keyword/structured retrieval; add embeddings or temporal graphs only when measured retrieval failures warrant them.

## 7. Deferred by design

No preemptive Kafka/Redis/Postgres split, Graphiti integration, workflow engine, dynamic plugin registry, elaborate event bus, abstract factories, or autonomous social posting. Any future addition should name the current failure it addresses, the simplest alternative considered, and a small regression test.

## 8. Review triggers

Revisit this baseline when: (a) a second channel creates concrete duplication or a new trust boundary; (b) a second model requires common provider semantics; (c) retrieval failures justify better segmentation/ranking or a derived vector/graph index; or (d) measured write contention/deployment needs justify reconsidering SQLite itself. Poor Chinese retrieval alone is not a reason to migrate the primary database.
