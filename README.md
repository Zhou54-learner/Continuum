# Continuum · 连续体

**One Identity. Many Forms. Continuous Memory.**

A modular, memory-centered runtime for persistent AI companions across platforms, tools, and devices.

Continuum explores a practical engineering problem: how can an AI companion keep a coherent, user-controlled identity and an inspectable memory across conversations, models, applications, and eventually physical devices?

**Status:** Foundation design / pre-implementation. The repository does not yet contain a working agent. Features described below are goals, not current capabilities.

## Vision

- **Continuity:** memories and identity belong to the system, not to a specific LLM provider or conversation.
- **User control:** persona, stored memories, account bindings, and permissions are inspectable and editable.
- **Multiple forms:** messaging platforms, desktop avatars, tools, and robots are optional adapters around the same core.
- **Context boundaries:** knowledge learned in one place is not automatically disclosed in another.

## Engineering stance

1. Start with a **modular monolith** and one complete vertical slice.
2. Keep responsibilities clear; do not create abstractions for hypothetical integrations.
3. Prefer a small number of well-understood dependencies over a large orchestration stack.
4. Preserve source provenance, identity verification, and authorization at real trust boundaries.
5. Make behavior and data migration testable before adding platforms or autonomy.

## First milestone

A single-owner local prototype: converse through one entry point, persist source-linked memories, retrieve permitted context in later sessions, correct or delete memories, and change persona settings without losing memory.

**Not in the first milestone:** QQ/WeChat/Bilibili integrations, group-chat autonomy, desktop pet, robotics, distributed services, graph databases, or multi-agent orchestration.

## Project notes

- [Architecture and boundaries](docs/ARCHITECTURE.md)
- [First MVP and acceptance checks](docs/MVP.md)
- [Architecture decisions](docs/DECISIONS.md)
- [Guidance for coding agents](AGENTS.md)

The repository is public: **never commit private conversation archives, API credentials, access tokens, or real identity mappings.** Local runtime data will remain untracked.

## License

[MIT](LICENSE). Dependencies and reused code retain their own license obligations.
