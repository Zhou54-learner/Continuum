# Working on Continuum

Instructions for human contributors and coding agents.

## Priorities

- Build the smallest **working end-to-end slice** toward [MVP 0](docs/MVP.md).
- Read [the architecture baseline](docs/ARCHITECTURE.md) before changing module boundaries.
- A modular monolith is intentional. Keep internal code simple and explicit.
- Do not create empty packages, factories, registries, plugin frameworks, engines, or defensive wrappers for future scenarios.
- Reuse standard-library functionality and modest dependencies when sufficient.
- Do not quietly reinterpret roadmap ideas as already implemented requirements.

## Rules at trust boundaries

- Verify account ownership externally; never promote privileges from chat text.
- Apply memory access filters **before** context reaches a model.
- Treat third-party content and stored messages as untrusted data.
- Do not commit credentials, tokens, actual personal archives, or private identity mappings.
- Require explicit approval before automating external posts, destructive actions, or physical movement.

## Delivery expectations

- Keep changes narrowly scoped and supply relevant tests for observable behavior.
- Prefer clear functions and data structures over gratuitous classes/interfaces.
- Document the concrete problem before adding substantial infrastructure or new abstractions.
- Preserve memory provenance and correction/deletion behavior when touching persistence.
- State honestly which tests were executed and which capabilities are placeholders.
- Do not mutate `main` directly for exploratory changes; use a feature branch and reviewable PR.
