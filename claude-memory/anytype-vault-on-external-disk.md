---
name: anytype-vault-on-external-disk
description: anytype-ts vault data is symlinked from the default macOS path to the Yatri Cloud external disk
metadata: 
  node_type: memory
  type: project
  originSessionId: 4df01c30-81a9-442c-88d5-93abc6da5869
---

The anytype-ts (Anytype desktop) vault/userData was relocated off the nearly-full internal disk onto the Yatri Cloud external APFS disk.

`~/Library/Application Support/anytype` is a **symlink** → `/Volumes/Yatri Cloud/anytype`.

Both the Electron app (`bun run start:dev`) and web mode (`bun run start:web`) compute that default path, so they transparently use the external disk with no flags. The account dir inside is `data/A9Ztuk93wnJQyxR7V1HvGfagVwcL74XG8wmygQzawJvMYaYr/` (objectstore = SQLite index, flatfs = encrypted CRDT blocks, fts_tantivy = search index, device.key = device key).

**Why:** internal disk `/` was 100% full (sqlite "disk is full" errors); external disk has ~80 GB free and is local Apple-Fabric APFS (safe for SQLite locking/fsync).

**How to apply:** If the Yatri Cloud disk is unplugged, the symlink dangles and Anytype fails to start — remount it or repoint the symlink. Repo itself lives at `/Volumes/Yatri Cloud/open-source/anytype-ts`. See [[anytype-ts-setup-toolchain]].
