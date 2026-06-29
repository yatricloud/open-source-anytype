---
name: anytype-cannot-host-on-vercel
description: Why Anytype (anytype-ts) cannot run as a normal hosted web app on Vercel
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4df01c30-81a9-442c-88d5-93abc6da5869
---

Anytype is local-first. The browser UI is a thin shell; the real work is done by `anytypeHelper`, a ~149 MB persistent Go middleware that serves the gRPC-web API and stores the **encrypted vault on the local filesystem** (flatfs + SQLite objectstore). It must stay running.

**Vercel = static files + ephemeral serverless functions** → cannot run a long-lived stateful Go gRPC server with persistent local storage. So you can deploy the `bun run build:web` static bundle to Vercel and the UI loads, but it's non-functional without a middleware to connect to. The only way it works is each visitor running their OWN local `anytypeHelper` and pointing the page's `?server=` at `http://127.0.0.1:31008` (and even then browsers fight HTTPS→localhost via mixed-content / Private Network Access). It is never a shared cloud Anytype — data stays on each machine.

For real always-on browser access, host the middleware on a **persistent server with a mounted volume** (Fly.io / Railway / VPS / Docker), not Vercel. Anytype's own self-hosting story is running an any-sync node, not middleware-as-web-backend.

The user asked about this for the repo at [[anytype-ts-how-to-run]]. Also: they once pasted a Vercel token in chat — advised to revoke it.
