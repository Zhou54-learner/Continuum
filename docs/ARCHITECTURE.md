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
- **Memory:** source-linked approved claim/episode, with stable memory ID, revision and reason, subject, recorded time, optional fact-validity interval, approval/dispute/deletion state, visibility and allowed processing destinations. Approval authorizes use, not objective truth or unrestricted cloud transmission.
- **Source excerpt:** minimum user-selected evidence, owned exclusively by one memory revision. A source ID identifies this retained excerpt; a separate content-free origin reference may identify the original message. It is not a shared transcript container.
- **Persona revision:** versioned editable settings, stored separately from factual memories and operational permissions. MVP uses one authoritative local configuration.

Relations, procedural memories, embeddings and time-aware graphs are future representations **only when queries justify them**. The persistent database is the source of truth; search indexes are rebuildable derivatives.

The default runtime root is `.continuum/` in the workspace, containing local configuration, `continuum.sqlite3` (including journal/WAL/SHM sidecars), caches and `exports/`. It is ignored as a whole. Public synthetic fixtures live separately under `tests/fixtures/`; custom runtime paths must remain outside version control. Do not persist full source messages in event metadata, origin references or diagnostic logs as an alternative transcript store.

## 4. Memory lifecycle

1. Receive an authenticated envelope with source metadata; store only what retention settings permit.
2. Resolve the speaker to a known person/account binding or a separate unverified identity; never trust text saying "I am the owner".
3. Create an **explicitly approved** memory from a user action. Automatic extraction/approval is not part of MVP; a later extractor may only propose candidates for review.
4. Compare with current memories. Update validity/status and provenance; do not erase contradictory historical evidence without a correction policy.
5. Memory filters all reads by trusted requester, audience, purpose and processing destination **before** exposing content to Agent or the model; the same check applies to history, source preview and export.
6. Allow inspection, correction, export and deletion. Deletion must propagate to derived indexes and summaries; backups follow an explicit retention policy.

### Revision and truth-status rules

An **Event** proves that a statement was observed, not that it is objectively true. Keep approval, time validity, dispute and deletion separate rather than encoding them as one overloaded status. Revisions have a stable memory ID, revision ID, reason, recorded time and optional factual validity interval.

- **Fact changes over time:** “I lived in Hangzhou in June and moved to Shanghai in September.” The old claim remains valid for June, but Shanghai is current.
- **Correction of an error:** “I was mistaken: I lived in Shanghai even in June.” The old Hangzhou claim was an **incorrect assertion**, not a valid historical residence.
- **Unresolved conflict:** “A says Li lives in Hangzhou; B says Shanghai.” Preserve source-attributed disputed claims; do not let latest write, similarity or model confidence silently pick the truth.

MVP corrections target a specific memory ID; no automatic semantic contradiction detector is promised. Default recall selects approved, nondeleted, currently applicable revisions. Authorized history can show corrected assertions with clear error labels.

### Withdrawal, deletion and provenance

For MVP, each revision retains one exclusive source excerpt. Two independently deletable facts from one message require separate, sufficient excerpts that do not contain each other's fact; only content-free origin metadata may be shared. For example, from “I live in Shanghai. I like tea.” retain the two sentences separately. Do not copy the whole message into both records. If evidence cannot be separated without losing its meaning, save the facts as one explicitly identified deletion unit, or ask for separate statements. Do not invent a paraphrase and label it an original quote.

- **Withdraw:** cease treating an assertion as current, while keeping clearly labeled owner-visible history.
- **Delete memory:** remove memory and revision **content** from current reads, history, exports, derived indexes/summaries and retained context caches. Retained source text must not recreate the deleted memory in later rebuilds. A content-free tombstone (ID, time, deletion status) is permissible for non-resurrection.
- **Delete source:** in MVP, deleting a retained source excerpt cascades to its owning memory, all revisions and their excerpts. A memory deletion removes all of its exclusive source text in the same database transaction. Other memories that share only an origin reference remain intact. Content-free metadata may honestly say “source content removed” without pretending the text remains auditable.
- **Deletion scope:** deletion targets a memory ID and all of its owned/derived content, not automatic semantic erasure of every independently saved assertion about the same topic. The command must make this scope clear. Replaying the original save request after deletion must not resurrect the memory; retain a content-free request identifier with the deletion marker. A genuinely new explicit save is a separate user action.
- **Guarantee limits:** no app-managed backups in MVP; user exports and copies already transmitted to model providers are outside local deletion guarantees. SQLite logical deletion is not forensic secure erasure. Future restore/backup logic must respect deletion markers.

Source content is **data**, not an instruction capable of overriding identity, persona, policy or tools.

## 5. Access and agency boundaries

Application constructs a **trusted request context** from actual entry-point authentication/configuration: requesting actor, session, **output audience/visibility**, purpose, and permitted model processing destination. The person discussed in a memory is not necessarily the requester or audience. In local MVP, control of the configured local account is the only owner trust source. Synthetic owner/public, guest and missing-context cases may not read owner-private content; do not build a general-purpose PolicyEngine.

**Memory owns the single authorization rule** for retrieval, inspection, revision history, source previews and exports. Agent only receives authorized content and cannot broaden access. The rule must also cover stored history snippets and future summaries; prompts alone cannot enforce this boundary.

For MVP, new memories default to local processing only. The owner can explicitly grant a memory access to the one configured cloud destination, identified by provider and endpoint; selecting a provider or approving a fact alone does not grant that access. Application supplies the actual destination from trusted adapter configuration, never from chat text or a caller-supplied label. Missing/unknown destinations deny memory access; a provider/endpoint change requires a matching grant. No destination registry or general-purpose policy engine is required.

Local inspection/export uses an explicit local destination and still checks requester, audience and purpose. Model-bound reads additionally require a matching processing grant, including source excerpts and history. Context authorized for local display must not be reused as a cloud prompt. Rebuild/recheck model context when the destination changes, including prior assistant messages containing recalled facts; no fallback provider may inherit another destination's grant. Any future external embedding or reranking call must obey the same boundary before receiving content.

A local SQLite database does not imply local inference. If configured to use a cloud model, only context permitted for that destination is sent to it. The application must document the destination and transmitted categories of data, including the current user message and persona settings as applicable; memory restrictions do not make the whole conversation local. Local deletion cannot retract requests already sent externally.

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
