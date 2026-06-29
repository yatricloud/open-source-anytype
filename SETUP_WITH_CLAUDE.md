# Setting up anytype-ts with Claude Code

This guide gets the **Anytype desktop** client (`anytype-ts`) running from a fresh
clone, and explains how to drive the whole process with
[Claude Code](https://claude.com/claude-code). It is written so you can either
follow the steps by hand, or paste them at Claude Code and let it do the work.

> Anytype is **local-first**. There is no server to deploy — the app talks to a
> local Go middleware binary (`anytypeHelper`) over gRPC, and all your data lives
> in an encrypted vault on disk. See [Vault / data location](#vault--data-location).

---

## TL;DR

```bash
# 0. Toolchain (once per machine) — Bun + protoc + a Python venv for native modules
curl -fsSL https://bun.sh/install | bash          # Bun (package manager)
brew install protobuf                              # protoc (for generate-protos.sh)
python3 -m venv .venv && ./.venv/bin/pip install setuptools   # node-gyp needs setuptools on arm64

# Always put the toolchain on PATH first:
export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"

# 1. Install + build native deps (run with the venv python on PATH)
PATH="$PWD/.venv/bin:$PATH" bun install

# 2. Download the middleware (anytypeHelper) + protobuf definitions into dist/
./update.sh macos-latest arm                       # use your platform/arch

# 3. Bundle locales, then generate the gRPC bindings from the downloaded protos
bun run update:locale
bash scripts/generate-protos.sh --from-dist        # EASY TO MISS — required, see below

# 4. Run it
bun run start:dev                                  # Electron desktop app
```

---

## Prerequisites

| Tool | Why | Install |
|------|-----|---------|
| **Bun** `1.3.x+` | The package manager this repo uses (do **not** use npm/yarn) | `curl -fsSL https://bun.sh/install \| bash` |
| **protoc** | `generate-protos.sh` compiles the `.proto` files | `brew install protobuf` (macOS) |
| **Python 3 + venv** | `node-gyp` rebuilds the native `keytar` module; needs `setuptools` on Apple Silicon | `python3 -m venv .venv && ./.venv/bin/pip install setuptools` |
| **Git** | clone / version control | preinstalled / `brew install git` |

> **PATH:** every command below assumes the toolchain is on PATH. Export it once
> per shell:
> ```bash
> export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"
> ```

---

## Setup, step by step

The order matters — each step produces files the next one needs.

1. **`bun install`** — installs JS deps and rebuilds native modules. Run it with
   the venv's Python first on PATH so `node-gyp` can build `keytar`:
   ```bash
   PATH="$PWD/.venv/bin:$PATH" bun install
   ```

2. **`./update.sh <platform> <arch>`** — downloads the prebuilt `anytypeHelper`
   middleware binary **and** the matching protobuf definitions into `dist/`.
   ```bash
   ./update.sh macos-latest arm     # Apple Silicon
   # ./update.sh macos-latest amd   # Intel mac
   # ./update.sh ubuntu-latest amd  # Linux
   ```

3. **`bun run update:locale`** — bundles the translation/locale files.

4. **`bash scripts/generate-protos.sh --from-dist`** — ⚠️ **the easy-to-miss
   step.** It generates `src/ts/lib/api/service.ts` and the `middleware/`
   bindings from the protos downloaded in step 2. These files are **not**
   committed to the repo, and the renderer's `dispatcher.ts` imports them — skip
   this and the app fails to start with missing-module errors.
   The `--from-dist` flag tells it to use the protos `update.sh` already
   downloaded (instead of fetching its own).

---

## Running

> **Never run `start:dev` and `start:web` at the same time** — they share one
> vault and the SQLite index will lock.

### Desktop (Electron) — recommended

```bash
export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"
bun run start:dev
```

Builds the Electron bundle, starts Vite on **:8080**, opens the desktop window,
and spawns its own `anytypeHelper` middleware. Closing the window stops Vite.

### Browser (web mode)

```bash
export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"
WEB_OPEN_BROWSER=1 bun run start:web
```

Starts `anytypeHelper` (a gRPC-web proxy on `127.0.0.1:31008`) **+** Vite on
**:3030**, then opens the browser. The `server` and `dataPath` query params are
**required** — a bare `localhost:3030` will not connect:

```
http://127.0.0.1:3030/?server=http%3A%2F%2F127.0.0.1%3A31008&dataPath=<URL-encoded path to your vault data dir>
```

### Other useful commands

```bash
bun run build       # production renderer build
bun run typecheck   # TypeScript check
bun run lint        # lint
ELECTRON_SKIP_NOTARIZE=1 bun run dist:mac   # package a .app → dist/mac-arm64/Anytype.app
```

To stop everything, kill the `anytypeHelper` and the Electron processes.

---

## Vault / data location

Anytype stores its encrypted vault under the OS default app-data path:

- **macOS:** `~/Library/Application Support/anytype`
- **Linux:** `~/.config/anytype`
- **Windows:** `%APPDATA%\anytype`

Inside, `data/<accountId>/` holds:
- `objectstore` — SQLite index
- `flatfs` — encrypted CRDT blocks
- `fts_tantivy` — full-text search index
- `device.key` — device key

Both run modes compute this default path automatically, so no flags are needed.

> **Tip — relocating the vault (e.g. to an external disk):** the app always reads
> the default path, so the clean way to move data is to make that path a
> **symlink** to wherever you want the bytes to live, e.g.
> `~/Library/Application Support/anytype` → `/Volumes/<external>/anytype`.
> If that external disk is ever unmounted the symlink dangles and Anytype won't
> start — remount it or repoint the symlink. Use a **local** disk (SQLite needs
> reliable locking/fsync; avoid network drives).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| App won't start, missing `service.ts` / `middleware/*` imports | You skipped step 4 — run `bash scripts/generate-protos.sh --from-dist`. |
| `bun install` fails building `keytar` / node-gyp errors on arm64 | Make sure the `.venv` python (with `setuptools`) is first on PATH for the install. |
| `protoc: command not found` | `brew install protobuf`, then re-export PATH. |
| SQLite "disk is full" or "database is locked" | Free space on / move the vault to a disk with room; ensure only one run mode is active. |
| Anytype hangs at startup after moving the vault | The symlinked external disk is unmounted — remount it. |
| Commands "not found" (`bun`, `protoc`) | `export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"`. |

---

## Driving this with Claude Code

[Claude Code](https://claude.com/claude-code) can run this whole setup for you.
Useful prompts once you're in the repo:

- *"Set up anytype-ts from scratch on this machine"* — installs the toolchain and
  runs steps 1–4.
- *"Start anytype in desktop mode"* / *"…in the browser"* — runs `start:dev` /
  `start:web` and hands you the URL.
- *"Anytype won't start, debug it"* — Claude reads the error, checks whether the
  protos were generated and the middleware downloaded, and fixes it.
- *"Package a macOS .app"* — runs the `dist:mac` build.

This repo also ships a `CLAUDE.md` and `AGENTS.md` at the root with deeper
architecture notes that Claude Code reads automatically.

---

## Reference

- Upstream project: <https://github.com/anyproto/anytype-ts>
- Anytype docs: <https://doc.anytype.io>
- This repo's `CLAUDE.md` / `AGENTS.md` — architecture & conventions
