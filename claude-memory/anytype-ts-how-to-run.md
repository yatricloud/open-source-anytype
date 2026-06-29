---
name: anytype-ts-how-to-run
description: Exact commands to launch anytype-ts (Electron + web mode) and the web access URL
metadata: 
  node_type: memory
  type: project
  originSessionId: 4df01c30-81a9-442c-88d5-93abc6da5869
---

Resume / run anytype-ts (already fully set up). Repo: `/Volumes/Yatri Cloud/open-source/anytype-ts`.

Always export the toolchain PATH first:
```
export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"
cd "/Volumes/Yatri Cloud/open-source/anytype-ts"
```

**Desktop (Electron):** `bun run start:dev` — builds electron bundle, starts Vite :8080, opens the Electron window. Spawns its own `anytypeHelper` middleware. Closing the window stops Vite.

**Browser (web mode):** `WEB_OPEN_BROWSER=1 bun run start:web` — starts `anytypeHelper` (gRPC-web proxy on `127.0.0.1:31008`) + Vite on `:3030`, then opens the browser. Access URL (server+dataPath query params are REQUIRED, bare localhost:3030 won't connect):
`http://127.0.0.1:3030/?server=http%3A%2F%2F127.0.0.1%3A31008&dataPath=%2FUsers%2Fyatharthchauhan%2FLibrary%2FApplication%20Support%2Fanytype%2Fdata`

**Never run start:dev and start:web at the same time** — they share one vault and the SQLite DB will lock.

Other: `bun run build` (prod), `bun run typecheck`, `bun run lint`, `ELECTRON_SKIP_NOTARIZE=1 bun run dist:mac` (packaged .app → `dist/mac-arm64/Anytype.app`). To stop: kill processes `anytypeHelper` and `anytype-ts/node_modules/electron`.

Vault data is on the external disk — see [[anytype-vault-on-external-disk]]. Setup details/gotchas — see [[anytype-ts-setup-toolchain]].
