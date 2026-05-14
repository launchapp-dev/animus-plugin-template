# animus-plugin-template

Scaffold template for [Animus](https://github.com/launchapp-dev/animus-cli) plugins.

> **Status:** Under construction — landing in Animus v0.4.0.

## What this is

Animus v0.4.0 ships `animus plugin new` — a scaffold command that generates a new plugin project from this template. Three plugin kinds are supported:

- **`subject`** — a backend for a unit-of-work source (Linear, Jira, GitHub Issues, Notion, Asana, Zendesk, ...)
- **`provider`** — a backend for an LLM provider (OpenAI, Anthropic, Gemini, on-prem inference, ...)
- **`trigger`** — a backend for event ingest (Slack, generic webhooks, file watchers, ...)

Once published, you'll be able to:

```bash
animus plugin new --kind subject --name jira
# Creates ./animus-subject-jira/ with a working stdio plugin skeleton
# that compiles, has the SubjectBackend trait stubbed, and runs the
# contract test suite against ao-subject-mock. Implement the API
# integration; ship.
```

## Layout (planned)

```
animus-plugin-template/
├── subject/        # scaffold for subject backends
├── provider/       # scaffold for provider backends
├── trigger/        # scaffold for trigger backends
└── _common/        # shared CI, license, README skeleton
```

Each kind subdirectory contains a complete project skeleton:

- `Cargo.toml` — depends on `ao-plugin-runtime` + the kind's protocol crate
- `plugin.toml` — plugin manifest (kind, name, version, capabilities)
- `src/main.rs` — one-liner: `subject_backend_main(MyBackend::new())`
- `src/backend.rs` — stubbed trait impl with `todo!()` in each method
- `src/config.rs` — env-var-driven config
- `tests/contract.rs` — exercises the protocol contract
- `.github/workflows/{ci,release}.yml` — build + test on PR, release binaries on tag
- `README.md` — auto-filled with plugin name
- `LICENSE` — MIT

## Using the scaffold without `animus plugin new`

You can also use this template directly with [`cargo generate`](https://github.com/cargo-generate/cargo-generate):

```bash
cargo generate --git https://github.com/launchapp-dev/animus-plugin-template subject
```

## Design

- **Protocol design:** [`docs/architecture/subject-backend-plugins.md`](https://github.com/launchapp-dev/animus-cli/blob/main/docs/architecture/subject-backend-plugins.md)
- **Naming contract:** [`docs/architecture/naming-contract.md`](https://github.com/launchapp-dev/animus-cli/blob/main/docs/architecture/naming-contract.md)
- **Repository name:** `animus-plugin-template` (brand prefix for discovery)
- **Generated repo names:** `animus-<kind>-<name>` (e.g. `animus-subject-jira`)
- **Generated binary names:** `ao-<kind>-<name>` (the `ao-*` protocol prefix per the naming contract)

## License

MIT — see [LICENSE](LICENSE).
