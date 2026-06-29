---
name: anytype-ts-setup-toolchain
description: "How anytype-ts was set up end-to-end on this Mac (bun, protoc, keytar venv, middleware)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 4df01c30-81a9-442c-88d5-93abc6da5869
---

anytype-ts at `/Volumes/Yatri Cloud/open-source/anytype-ts` was set up end-to-end. Toolchain installed during setup:

- **Bun** at `~/.bun/bin/bun` (1.3.x) — the required package manager (`bun install`).
- **protoc** via Homebrew (`/opt/homebrew/bin`) — needed for `generate-protos.sh`.
- **Python venv** at `anytype-ts/.venv` with `setuptools` — node-gyp needs it to rebuild the native `keytar` module on arm64.

Setup order that worked: `bun install` (with venv python on PATH) → `./update.sh macos-latest arm` (downloads anytypeHelper middleware v0.50.12 + protos into dist/) → `bun run update:locale` → **`bash scripts/generate-protos.sh --from-dist`** (this generates `src/ts/lib/api/service.ts` + `middleware/` bindings, which are NOT committed and the renderer's dispatcher.ts requires — easy to miss) → `bun run start:dev` (Electron) or `bun run start:web` (browser, Vite :3030 + gRPC-web proxy :31008).

Run commands with `export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"`.

Vault data location: see [[anytype-vault-on-external-disk]].
