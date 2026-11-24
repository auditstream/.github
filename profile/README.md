# Auditstream

**Open-source audit trail engine for PostgreSQL — written in Rust, with thin TypeScript adapters.**

Auditstream records **who** changed **what** in PostgreSQL 15–19 with full `before` and `after` values, stored in a partitioned append-only history table (`auditstream.changes`) inside your own database and exported on demand using a commit-safe `xid8` cursor.

**No 24/7 consumer daemon. No WAL replication slots. No new infrastructure.**

→ **[auditstream.dev](https://auditstream.dev)** · **[Documentation](https://auditstream.dev/docs)** · **[Spec v0.1](https://auditstream.dev/docs/contract)** · **[Verified Benchmarks](https://auditstream.dev/docs/evidence)**

---

### How it works

1. **Thin adapters attach the actor:** `@auditstream/node`, `@auditstream/prisma` (Prisma 7.1+), and `@auditstream/drizzle` (0.45 & 1.x) propagate identity via `AsyncLocalStorage.run()` and append an unpadded base64url `sqlcommenter` tag (`/*auditstream='...'*/`) — supporting humans, API keys, services, and AI agents acting `onBehalfOf` a user.
2. **PostgreSQL triggers capture atomically:** Generated `SECURITY DEFINER` triggers write `before`, `after`, `changed`, and actor metadata into `auditstream.changes` (statement-level for `INSERT`/`DELETE`, row-level with `IS DISTINCT FROM` for `UPDATE`). Application roles receive `SELECT` only.
3. **One Rust CLI inspects and exports:** `auditstream init`, `verify`, `history`, and `export` (JSON Lines & CSV in v0.1) read committed windows via `xact_id < pg_snapshot_xmin()` so concurrent out-of-order commits are never skipped.

---

### Repositories

| Repository | Description |
| :--- | :--- |
| `auditstream/auditstream` | Rust engine and CLI, normative `spec/`, conformance fixtures, and architecture docs |
| `auditstream/auditstream-js` | ESM-only TypeScript adapters (`@auditstream/node`, `@auditstream/prisma`, `@auditstream/drizzle`) |
| `auditstream/site` | Website and documentation ([auditstream.dev](https://auditstream.dev)) |

> **Status:** **v0.1.0 is live!** Install the CLI via shell (`curl -LsSf https://github.com/auditstream/auditstream/releases/latest/download/auditstream-installer.sh | sh`), Homebrew (`brew install auditstream/tap/auditstream`), npm (`npm i -g auditstream`), or Docker (`ghcr.io/auditstream/auditstream:latest`), and wire TypeScript adapters with `pnpm add @auditstream/node @auditstream/prisma` (or `@auditstream/drizzle`). Read the docs and verified benchmarks at **[auditstream.dev](https://auditstream.dev)**.

---

### License & Security

- **License:** MIT
- **Security disclosures:** `security@auditstream.dev`
