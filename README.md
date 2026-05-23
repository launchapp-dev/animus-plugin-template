# animus-plugin-template

Scaffold templates for [Animus](https://github.com/launchapp-dev/animus-cli) plugins.

> **Status:** Templates are usable today via manual scaffold or
> `cargo generate`. A first-party `animus plugin new` scaffold command
> is planned but **not yet shipped** — the Animus v0.4.0 CLI today
> exposes `animus plugin install | uninstall | list | info | call | ping`,
> with no `new` subcommand.

## What this is

A set of scaffold templates for building Animus plugins. Three plugin
kinds are supported:

- **`subject`** — a backend for a unit-of-work source (Linear, Jira,
  GitHub Issues, Notion, Asana, Zendesk, ...)
- **`provider`** — a backend for an LLM provider (OpenAI, Anthropic,
  Gemini, on-prem inference, ...)
- **`trigger`** — a backend for event ingest (Slack, generic webhooks,
  file watchers, cron, ...). Ships a working `FileWatch` example built
  on `animus-trigger-protocol` v0.1.8.

Until `animus plugin new` ships, scaffold a new plugin by cloning this
repo, copying the kind subdirectory you want, and substituting the
template variables (see [Using the scaffold](#using-the-scaffold) below
for the exact commands).

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
└── trigger/                 # full scaffold for trigger backends
    └── (same layout as subject/)
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

### Manually (works today)

Clone the template, copy the kind subdir you want, substitute variables,
and strip the `.tmpl` suffix:

```bash
# Clone the template
git clone https://github.com/launchapp-dev/animus-plugin-template
cp -r animus-plugin-template/subject my-plugin
cd my-plugin

# Replace template variables in every .tmpl file. Pick your own values
# for {{name}}, {{org}}, etc. — the full variable list is in
# template-manifest.toml.
find . -name '*.tmpl' -type f -print0 | while IFS= read -r -d '' f; do
  sed -i.bak \
    -e 's/{{name}}/jira/g' \
    -e 's/{{NAME_UPPER}}/JIRA/g' \
    -e 's/{{NAME_PASCAL}}/Jira/g' \
    -e 's/{{name_snake}}/jira/g' \
    -e 's/{{kind}}/subject/g' \
    -e 's/{{full_name}}/animus-subject-jira/g' \
    -e 's/{{description}}/An Animus subject backend for Jira/g' \
    -e 's|{{org}}|launchapp-dev|g' \
    -e 's/{{year}}/2026/g' \
    "$f" && rm "${f}.bak"
done

# Strip the .tmpl suffix
find . -name '*.tmpl' -print0 | xargs -0 -I {} bash -c 'mv "$1" "${1%.tmpl}"' _ {}

cargo build
animus plugin install .
```

### Via `cargo generate`

You can also use this template with
[`cargo generate`](https://github.com/cargo-generate/cargo-generate):

```bash
cargo generate \
  --git https://github.com/launchapp-dev/animus-plugin-template \
  --branch main \
  subject
```

`cargo generate` will prompt for `name`, `description`, etc. interactively.
Pre-supply values with `--define name=jira --define description="..."`.

### Via `animus plugin new` (planned, not yet shipped)

A first-party scaffold command is on the roadmap:

```bash
# Not implemented yet — the Animus v0.4.0 CLI today exposes only
# `animus plugin install | uninstall | list | info | call | ping`.
animus plugin new --kind subject --name jira \
  --org launchapp-dev \
  --description "Animus subject backend for Jira issues"
```

Once shipped, it will clone this repository, resolve substitution
variables (CLI flags > prompts > git config > defaults), render every
`.tmpl` file, and drop the result at `./<full_name>/` ready to commit.
Until then, use the manual or `cargo generate` paths above.

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

Models a [`SubjectBackend`](https://github.com/launchapp-dev/animus-protocol/blob/main/animus-subject-protocol/src/lib.rs)
implementation. Stubs out `list`, `get`, `update`, `watch`, `schema`,
and `health`. Defaults `supports_watch=false` (polling-only); flip to
`true` when you implement live subscriptions.

Reference: [`animus-subject-linear`](https://github.com/launchapp-dev/animus-subject-linear).

### `provider/`

Models a [`ProviderBackend`](https://github.com/launchapp-dev/animus-protocol/blob/main/animus-provider-protocol/src/lib.rs)
implementation. Stubs out `manifest`, `run_agent`, `resume_agent`,
`cancel_agent`, and `health`. The shared runtime handles JSON-RPC
framing, the initialize handshake, and aggregating the final result
envelope.

References: `animus-provider-claude`, `animus-provider-codex`,
`animus-provider-gemini`, `animus-provider-oai`, `animus-provider-opencode`.

### `trigger/`

Models a [`TriggerBackend`](https://github.com/launchapp-dev/animus-protocol/blob/main/animus-trigger-protocol/src/lib.rs)
implementation. Pinned to `animus-protocol` **v0.1.8** — the first tag
exposing `animus-trigger-protocol`. The shipped example is a
`FileWatch` backend built on the `notify` crate: it watches a directory
and emits one debounced `{{name}}_file_changed` event per file change.
Swap the watcher in `src/backend.rs` for a Slack socket, webhook
listener, cron interval, message queue, or any other event source —
the trait surface (`schema`, `watch`, `ack`, `health`) does not change.

Reference: the trigger scaffold ships with a real `notify`-based
backend that runs as-is; downstream forks customize the event source
rather than starting from `todo!()`.

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
