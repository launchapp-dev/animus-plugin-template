# animus-plugin-template

Scaffold templates for [Animus](https://github.com/launchapp-dev/animus-cli) plugins.

> **Status:** Under construction — landing in Animus v0.4.0.

## What this is

Animus v0.4.0 ships `animus plugin new` — a scaffold command that
generates a new plugin project from this repository. Three plugin kinds
are supported:

- **`subject`** — a backend for a unit-of-work source (Linear, Jira,
  GitHub Issues, Notion, Asana, Zendesk, ...)
- **`provider`** — a backend for an LLM provider (OpenAI, Anthropic,
  Gemini, on-prem inference, ...)
- **`trigger`** — a backend for event ingest (Slack, generic webhooks,
  file watchers, ...) — landing in v0.4.x.

```bash
animus plugin new --kind subject --name jira
# Creates ./animus-subject-jira/ with a working stdio plugin skeleton
# that compiles, has the SubjectBackend trait stubbed, and runs the
# contract test suite against animus-subject-mock. Implement the API
# integration; ship.
```

## Layout

```
animus-plugin-template/
├── template-manifest.toml   # describes kinds + substitution variables
├── README.md                # this file
├── LICENSE                  # MIT
├── .gitignore
├── _common/                 # shared snippets
│   ├── LICENSE.tmpl         # MIT boilerplate (interpolated into each kind)
│   └── README-header.tmpl   # shared README intro
├── subject/                 # full scaffold for subject backends
│   ├── Cargo.toml.tmpl
│   ├── plugin.toml.tmpl
│   ├── src/
│   │   ├── lib.rs.tmpl
│   │   ├── main.rs.tmpl
│   │   ├── backend.rs.tmpl
│   │   └── config.rs.tmpl
│   ├── tests/contract.rs.tmpl
│   ├── .github/workflows/
│   │   ├── ci.yml.tmpl
│   │   └── release.yml.tmpl
│   ├── README.md.tmpl
│   ├── LICENSE.tmpl
│   └── .gitignore
├── provider/                # full scaffold for provider backends
│   └── (same layout as subject/)
└── trigger/                 # placeholder; full skeleton lands in v0.4.x
    └── README.md.tmpl
```

Files with a `.tmpl` suffix are processed by the substitution engine.
Files without the suffix are copied verbatim. The `.tmpl` suffix is
stripped from the destination path.

## Substitution variables

| Variable          | Source       | Example for `--kind subject --name jira`   |
|-------------------|--------------|--------------------------------------------|
| `{{name}}`        | required     | `jira`                                     |
| `{{NAME_UPPER}}`  | derived      | `JIRA`                                     |
| `{{NAME_PASCAL}}` | derived      | `Jira`                                     |
| `{{name_snake}}`  | derived      | `jira`                                     |
| `{{kind}}`        | required     | `subject` (or `provider` / `trigger`)      |
| `{{full_name}}`   | derived      | `animus-subject-jira`                      |
| `{{description}}` | prompted     | "An Animus subject backend for Jira"       |
| `{{org}}`         | prompted     | `launchapp-dev`                            |
| `{{author}}`      | git config   | (from `git config user.name`)              |
| `{{author_email}}`| git config   | (from `git config user.email`)             |
| `{{year}}`        | derived      | `2026`                                     |

See [`template-manifest.toml`](./template-manifest.toml) for the
authoritative list, including validation patterns and per-variable
defaults.

## Using the scaffold

### Via `animus plugin new` (recommended)

```bash
animus plugin new --kind subject --name jira \
  --org launchapp-dev \
  --description "Animus subject backend for Jira issues"

cd animus-subject-jira
cargo build
animus plugin install .
```

`animus plugin new` clones this repository, resolves substitution
variables (CLI flags > prompts > git config > defaults), renders every
`.tmpl` file, and drops the result at `./<full_name>/` ready to commit.

### Via `cargo generate`

You can also use this template directly with
[`cargo generate`](https://github.com/cargo-generate/cargo-generate):

```bash
cargo generate \
  --git https://github.com/launchapp-dev/animus-plugin-template \
  --branch main \
  subject
```

`cargo generate` will prompt for `name`, `description`, etc. interactively.
Pre-supply values with `--define name=jira --define description="..."`.

### Manually

Clone, copy the kind subdir you want, and substitute by hand:

```bash
git clone https://github.com/launchapp-dev/animus-plugin-template
cp -R animus-plugin-template/subject animus-subject-jira
cd animus-subject-jira
# Rename .tmpl files and replace {{name}}, {{kind}}, {{full_name}}, etc.
```

## Generated project anatomy

Each rendered project ships with:

- **`Cargo.toml`** — depends on the matching `animus-*-protocol` crate
  and on `animus-plugin-runtime`, with `tokio`, `serde`, `reqwest`,
  `tracing`, `anyhow`, `thiserror`, `async-trait`, `chrono`, `mockito`
  wired up.
- **`plugin.toml`** — manifest matching `PluginManifest` (name, version,
  kind, description, capabilities, declared env vars).
- **`src/main.rs`** — stdio entrypoint that builds the backend and hands
  off to the shared runtime.
- **`src/backend.rs`** — trait impl with `todo!()` bodies that describe
  exactly what to implement.
- **`src/config.rs`** — env-var-driven `*Config` struct + `from_env()`.
- **`src/lib.rs`** — library surface so contract tests can reach the
  backend without going through the binary.
- **`tests/contract.rs`** — `#[ignore]`d contract tests scaffolded
  against mockito; un-ignore as each method comes online.
- **`.github/workflows/ci.yml`** — fmt + clippy + test on every push
  and PR.
- **`.github/workflows/release.yml`** — tag-driven multi-platform binary
  release (Linux x86_64, macOS aarch64, macOS x86_64) with SHA256
  sidecars that `animus plugin install` verifies.
- **`README.md`** — install/configure/run/roadmap with `{{name}}`,
  `{{NAME_UPPER}}`, and `{{full_name}}` already substituted.
- **`LICENSE`** — MIT, copyright `{{year}} {{author}}`.

## Per-kind reference

### `subject/`

Models a [`SubjectBackend`](https://github.com/launchapp-dev/animus-cli/blob/main/crates/animus-subject-protocol/src/lib.rs)
implementation. Stubs out `list`, `get`, `update`, `watch`, `schema`,
and `health`. Defaults `supports_watch=false` (polling-only); flip to
`true` when you implement live subscriptions.

Reference: [`animus-subject-linear`](https://github.com/launchapp-dev/animus-subject-linear).

### `provider/`

Models a [`ProviderBackend`](https://github.com/launchapp-dev/animus-cli/blob/main/crates/animus-plugin-runtime/src/lib.rs)
implementation. Stubs out `start` (open a session, stream tokens) and
`cancel` (abort in-flight). The shared runtime handles streaming
notifications, JSON-RPC framing, and aggregating the final result.

References: `animus-provider-claude`, `animus-provider-codex`,
`animus-provider-gemini`, `animus-provider-oai`, `animus-provider-opencode`.

### `trigger/`

**Placeholder.** Full scaffold lands once `TriggerBackend` is defined in
v0.4.x. For now the directory only contains a README explaining the
plan.

## Design pointers

- **Protocol design:** [`docs/architecture/subject-backend-plugins.md`](https://github.com/launchapp-dev/animus-cli/blob/main/docs/architecture/subject-backend-plugins.md)
- **Naming contract:** [`docs/architecture/naming-contract.md`](https://github.com/launchapp-dev/animus-cli/blob/main/docs/architecture/naming-contract.md)
- **Repository name:** `animus-plugin-template`
- **Generated repo names:** `animus-<kind>-<name>` (e.g. `animus-subject-jira`)
- **Generated crate / binary names:** `animus-<kind>-<name>` (matches the repo name)

Per the v0.4.0 naming convention: repo, crate, and binary all share the
same `animus-{kind}-{name}` name. There is no longer an `ao-` prefix
anywhere.

## License

MIT — see [LICENSE](LICENSE).
