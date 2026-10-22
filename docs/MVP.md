# MVP 0 — One identity, persistent memory, one entry point

**Status:** Implementation plan, not a claim of completed functionality.

## Outcome

A local user can hold two separate conversations with the same configured AI identity. In the second session the agent recalls permitted, attributable information from the first. The owner can inspect, correct, delete and export that information, and change the persona without rewriting the remembered facts.

The first user-facing entry point can be a simple **CLI**. There is no need for desktop UI, server deployment, OAuth, messaging platform access, or robots.

## Minimal vertical slice

1. Configure stable local **owner person ID** and **companion ID** with separately editable persona settings; refuse to infer an owner from message text.
2. Send a message through the CLI and receive a model response.
3. Persist minimum source evidence and **explicitly owner-approved** memory content in SQLite, without saving entire chat transcripts by default.
4. In a new process/session, retrieve relevant owner-permitted memory and show which source produced it.
5. Inspect, correct and delete a memory. Correction preserves an understandable revision history; deletion removes retrievable derived representations.
6. Modify the persona's speaking style without changing identity, history or permissions.

**MVP rule:** memory creation requires explicit owner approval. LLM extraction may be explored only as an **unapproved suggestion** after the basic lifecycle works; autonomous approval is out of scope.

## Acceptance checks

| Scenario | Expected result |
| --- | --- |
| Session continuity | Save an approved memory, exit the process, and restart without passing the original full transcript. Retrieve it, feed the authorized result to a fake or real model, and show memory ID/revision/source. |
| Provenance | Each recalled memory shows revision, approval, recorded time, source ID and source status. Deleted source text is labeled removed, not reconstructed or invented. |
| Correction vs time change | A corrected error is labeled false in history; a later real-world change preserves previously valid times. Conflicting unapproved facts do not silently replace the current assertion. |
| Deletion | After restart and index rebuild, deleted content is absent from retrieval, revision history, source previews, export and retained context; it cannot be recreated by source processing. |
| Persona switch | Different style; same user identity and stored facts. |
| Authorization | Test owner/private, owner/public, guest, missing context and text impersonation. Inspect the **complete input received by the fake model** for absence of private content in denied cases. |
| Identity spoofing | A message claiming "I am the owner" does not upgrade privileges. |
| Persistence | Process restart preserves approved memories and stable owner/companion IDs. Replayed save requests do not produce duplicate active copies. |
| Model boundary | Memory data excludes provider response objects; simulated model failure does not undo a saved fact or create fictional completed actions. |

| Export | JSON export parses with standard tools, carries schema version, stable IDs, revision/source metadata, and excludes deleted content, secrets and provider response objects. Import/restore is deferred. |
| Chinese retrieval | From the first retrieval slice, test short names/places, paraphrases, no-answer, deleted, stale and denied memories. Before MVP completion, aim for >=18/20 answerable Chinese questions retrieving the right current fact in top 5 from about 30 synthetic memories; record misses and timing. |

Use **synthetic fixtures**, not actual QQ/WeChat logs or personal profiles, in public tests. The Chinese target is a proposed project threshold, not an external benchmark. Check retrieval evidence and complete model input separately from generated answers.

## Suggested implementation order

- **Step 0 — bootstrap (Issue #2, not product acceptance):** Minimal CLI, stable owner/companion IDs, editable persona, one LLM adapter, one real call and an offline fake-model smoke test.
- **Step 1 — first product acceptance gate:** Explicit source-linked save → exit process → authorize and retrieve in a new process → send **only authorized** context to the model → show memory revision and source. Test synthetic public/guest denial and initial Chinese queries **in this step**.
- **Step 2 — controlled lifecycle:** Targeted inspect/revise (correction vs time change), delete/source handling, JSON export, index/context non-resurrection checks, and persona independence.

Split into small reviewable PRs if helpful. Every step updates its runnable instructions. Finishing Step 0 alone does **not** verify Continuum's central memory hypothesis.

Finish each slice with a runnable example and focused tests before the next one. Avoid generating later-phase directory trees or dependency stacks in advance.

## Intentionally out of scope

Automated account linking, third-party profile inference, group-chat messaging, scheduled autonomy, voice, avatars, robotics, temporal knowledge graphs, multi-agent systems, dashboard, distributed storage, and elaborate tool/plugin registries.

## Definition of done

A new contributor can follow the README to run the CLI and tests; the **cross-process memory loop** and all acceptance checks are demonstrated. Documentation must identify which approved data is transmitted to any configured cloud model and disclose that local deletion cannot retract past external calls. Claim only capabilities verified by execution.
