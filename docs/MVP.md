# MVP 0 — One identity, persistent memory, one entry point

**Status:** Implementation plan, not a claim of completed functionality.

## Outcome

A local user can hold two separate conversations with the same configured AI identity. In the second session the agent recalls permitted, attributable information from the first. The owner can inspect, correct, delete and export that information, and change the persona without rewriting the remembered facts.

The first user-facing entry point can be a simple **CLI**. There is no need for desktop UI, server deployment, OAuth, messaging platform access, or robots.

## Minimal vertical slice

1. Configure one owner and one named AI persona locally; refuse to infer an owner from message text.
2. Send a message through the CLI and receive a model response.
3. Persist the interaction's source and explicitly approved memory content in SQLite.
4. In a new process/session, retrieve relevant owner-permitted memory and show which source produced it.
5. Inspect, correct and delete a memory. Correction preserves an understandable revision history; deletion removes retrievable derived representations.
6. Modify the persona's speaking style without changing identity, history or permissions.

**Implementation preference:** begin with explicit memory creation/confirmation. LLM-driven candidate extraction can be added once the data lifecycle works reliably.

## Acceptance checks

| Scenario | Expected result |
| --- | --- |
| Session continuity | A new session retrieves the correct approved memory after restart. |
| Provenance | Each recalled memory has a visible origin and recorded time. |
| Correction | A corrected fact is current; old evidence is historical, not presented as equally current. |
| Deletion | Deleted memory is no longer returned from indexes or context assembly. |
| Persona switch | Different style; same user identity and stored facts. |
| Authorization | A synthetic guest or public context cannot retrieve owner-private memory. |
| Identity spoofing | A message claiming "I am the owner" does not upgrade privileges. |
| Persistence | Closing and reopening the process preserves approved memories. |
| Model boundary | Persistent memory format does not embed provider-specific response objects. |

Use **synthetic fixtures**, not actual QQ/WeChat logs or personal profiles, in public tests.

## Suggested implementation order

- **Slice A:** Minimal CLI + configuration + one LLM adapter + a smoke test.
- **Slice B:** SQLite event/memory persistence, source linkage, restart test.
- **Slice C:** Filtered retrieval, inspect/correct/delete, provenance and privacy tests.
- **Slice D:** Persona editing, lightweight retrieval evaluation and short usage documentation.

Finish each slice with a runnable example and focused tests before the next one. Avoid generating later-phase directory trees or dependency stacks in advance.

## Intentionally out of scope

Automated account linking, third-party profile inference, group-chat messaging, scheduled autonomy, voice, avatars, robotics, temporal knowledge graphs, multi-agent systems, dashboard, distributed storage, and elaborate tool/plugin registries.

## Definition of done

A new contributor can follow the README to run the CLI and the tests; every acceptance check above is demonstrated. Claims of working functionality should only be added to the README after execution has been verified.
