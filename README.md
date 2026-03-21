# Boundary Starter

A project scaffolding wizard for new [Boundary](https://github.com/tcbv/boundary) projects.

## Prerequisites

- [Java 17+](https://adoptium.net/)
- [Clojure CLI](https://clojure.org/guides/install_clojure)
- [Babashka](https://github.com/babashka/babashka#installation) (`bb`)

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/tcbv/boundary-starter/main/scripts/bootstrap.sh | bash
cd boundary-starter
bb setup
```

Or non-interactively:

```bash
bb setup --template web-app --name my-project --db sqlite
```

## Templates

| Template | Libraries included | DB default |
|---|---|---|
| `minimal` | core, observability, platform | SQLite |
| `api-only` | + user | SQLite |
| `microservice` | core, observability, platform | SQLite |
| `web-app` | + user, admin | SQLite |
| `saas` | + user, admin, storage, cache, jobs, email, tenant | PostgreSQL |
| `custom` | Interactive selection | Your choice |

`core`, `observability`, and `platform` are always included.

### Database options

| Value | Notes |
|---|---|
| `sqlite` | Zero-config. Default for all templates except `saas`. |
| `postgres` | Required for multi-tenancy (`tenant` library). |
| `both` | Includes both JDBC drivers; switch via `DATABASE_URL` at runtime. |

## Wizard flags

```bash
bb setup --template web-app --name my-project   # Non-interactive
bb setup --template saas --name my-app --dry-run  # Preview without writing files
bb setup --template minimal --name my-app --yes   # Skip confirmation prompts
```

## Custom templates

```bash
bb setup
# Select: 6 (custom template) → 1 (Create new custom template)
# Choose libraries interactively, then save as saved-templates/my-template.edn
```

Saved templates live in `saved-templates/` (gitignored) and can be reused, edited, duplicated, or renamed via the wizard.

## Generated project structure

```
{project-name}/
├── deps.edn                        ← Boundary libs as git deps
├── bb.edn                          ← bb scaffold / bb gen-agents
├── build.clj
├── AGENTS.md                       ← Project-local dev guide
├── scripts/gen_agents.clj
├── src/boundary/{project-name}/app.clj
├── test/boundary/{project-name}/app_test.clj
├── resources/conf/dev/
│   ├── config.edn
│   └── system.clj
└── .env.example
```

## Running the generated project

```bash
cd my-project
export JWT_SECRET="dev-secret-minimum-32-characters"   # required for auth templates
clojure -M:repl-clj
# (require '[integrant.repl :as ig-repl])
# (ig-repl/go)
# → http://localhost:3000
```

## Troubleshooting

### Invalid project name

Project names must be kebab-case (lowercase letters, digits, hyphens):

```bash
bb setup --template minimal --name my-project   # ✅
bb setup --template minimal --name MyProject    # ❌
bb setup --template minimal --name my_project   # ❌
```

### JWT_SECRET not set

Authentication templates (api-only, web-app, saas) require a secret of at least 32 characters:

```bash
export JWT_SECRET="dev-secret-32-chars-minimum-length"
```

Generate one: `openssl rand -base64 32`

### SQLite database locked

Close other REPL sessions that have the database file open (`lsof dev-database.db`).

### defrecord changes require full restart

`(ig-repl/reset)` is not enough — use `(ig-repl/halt)` then `(ig-repl/go)`.

### Dependency resolution failed

The wizard uses git dependencies pointing to the boundary monorepo. On first use, Clojure downloads them into `~/.gitlibs/`. Subsequent runs work offline from cache.

### Dependency conflict auto-resolved

When a selected library requires another (e.g. `calendar` requires `storage`), the wizard adds it automatically with a warning. No action needed.

## Platform notes

### Windows

- Use **Windows Terminal** or **PowerShell 5.1+** for ANSI color support (old CMD works but shows raw escape codes).
- Enable long path support to avoid `MAX_PATH` errors:
  ```powershell
  # Run as Administrator
  New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
    -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
  ```
- Recommended Git config: `git config --global core.autocrlf input`

### macOS Gatekeeper warning for `bb`

```bash
brew install borkdude/brew/babashka   # Homebrew handles signing
# or: xattr -d com.apple.quarantine $(which bb)
```

## Links

- [Boundary framework](https://github.com/tcbv/boundary)
- [Boundary AGENTS.md](https://github.com/tcbv/boundary/blob/main/AGENTS.md)
- [AGENTS.md](AGENTS.md) — starter development guide
