# Auditstream

[![CLI Release][badge-release]][link-release]
[![Spec Release][badge-spec]][link-spec]
[![SonarCloud][badge-sonar]][link-sonar]
[![MIT License][badge-license]][link-license]

**Open-source database audit trail engine — written in Rust, with thin TypeScript adapters.**

Auditstream records **who** changed **what** at the database level with full `before` and `after` values, stored in a partitioned append-only history table (`auditstream.changes`) inside your own database and exported on demand using a commit-safe `xid8` cursor.

**No 24/7 consumer daemon. No WAL replication slots. No new infrastructure.**

`+3.2% (+5.1 µs/row)` durable PG 19 UPDATE · `2.4M` PgBouncer tx with `0` actor leaks · [Benchmarks](https://auditstream.dev/docs/evidence/)

→ **[auditstream.dev](https://auditstream.dev)** · **[Docs](https://auditstream.dev/docs/)** · **[Quickstart](https://auditstream.dev/docs/getting-started/)** · **[TypeScript SDK](https://auditstream.dev/docs/typescript/)** · **[Evidence](https://auditstream.dev/docs/evidence/)** · **[Spec v0.1.1](https://auditstream.dev/docs/contract/)**

---

## How it works

1. **Thin adapters attach the actor:** `@auditstream/node`, `@auditstream/prisma` (Prisma 7.1+), and `@auditstream/drizzle` (0.45 & 1.x) propagate identity via `AsyncLocalStorage.run()` and append an unpadded base64url `sqlcommenter` tag (`/*auditstream='...'*/`) — supporting humans, API keys, services, and AI agents acting `onBehalfOf` a user.
2. **Database triggers capture atomically:** Generated `SECURITY DEFINER` triggers write `before`, `after`, `changed`, and actor metadata into `auditstream.changes` (statement-level for `INSERT`/`DELETE`, row-level with `IS DISTINCT FROM` for `UPDATE`). Application roles receive `SELECT` only.
3. **One Rust CLI inspects and exports:** `auditstream init`, `verify`, `status`, `history`, and `export` (JSON Lines & CSV in v0.1) read committed windows via `xact_id < pg_snapshot_xmin()` so concurrent out-of-order commits are never skipped.

---

## Support status

| Target | Status | Details |
| :--- | :--- | :--- |
| **PostgreSQL** | **Supported & tested** | PostgreSQL 15–19 (verified 15.20–19.0), PgBouncer 1.26, Neon, and Supabase |
| **MySQL** | Evaluating | Not supported yet — PostgreSQL is the only engine in v0.1 |
| **TypeScript SDK** | **Supported** | `@auditstream/*` v0.2.0 for Prisma 7, Drizzle, Next.js 16, Hono, and Express |
| **Python SDK** | Planned | No native SDK yet — works today via [`psycopg` and SQL comments](https://auditstream.dev/docs/python/) |
| **Plain SQL** | **Supported** | Any language or driver (Go, Ruby, Java, `psql`) via [SQL comments or `set_config`](https://auditstream.dev/docs/plain-sql/) |

---

## Repositories

| Repository | Description |
| :--- | :--- |
| [`auditstream/auditstream`](https://github.com/auditstream/auditstream) | Rust engine & CLI (`v0.1.0`), normative `spec/` (`v0.1.1`), conformance fixtures, handbook |
| [`auditstream/auditstream-js`](https://github.com/auditstream/auditstream-js) | TypeScript SDKs (`v0.2.0`): [`@auditstream/node`](https://www.npmjs.com/package/@auditstream/node), [`@auditstream/prisma`](https://www.npmjs.com/package/@auditstream/prisma), [`@auditstream/drizzle`](https://www.npmjs.com/package/@auditstream/drizzle) |
| [`auditstream/site`](https://github.com/auditstream/site) | Documentation & site ([auditstream.dev](https://auditstream.dev)) |
| [`auditstream/homebrew-tap`](https://github.com/auditstream/homebrew-tap) | Official Homebrew formula (`brew install auditstream/tap/auditstream`) |

---

## Quick install

CLI `v0.1.0` · TypeScript SDKs `v0.2.0` · Spec `v0.1.1`

```sh
# Shell (macOS & Linux)
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/auditstream/auditstream/releases/latest/download/auditstream-installer.sh | sh

# Homebrew (macOS & Linux)
brew install auditstream/tap/auditstream

# npm
npm i -g auditstream

# Docker (multi-arch linux/amd64 + linux/arm64)
docker run --rm ghcr.io/auditstream/auditstream:0.1.0 --version

# TypeScript SDKs (or replace @auditstream/prisma with @auditstream/drizzle)
pnpm add @auditstream/node @auditstream/prisma
```

**Windows (PowerShell):** `powershell -ExecutionPolicy Bypass -c "irm https://github.com/auditstream/auditstream/releases/latest/download/auditstream-installer.ps1 | iex"`

---

## License & Security

- **License:** [MIT License](https://github.com/auditstream/auditstream/blob/main/LICENSE)
- **Security disclosures:** `security@auditstream.dev` · [SECURITY.md](https://github.com/auditstream/auditstream/security/policy)
- **Supply chain:** [SLSA Build Attestations](https://github.com/auditstream/auditstream/attestations) · [GHCR container image](https://github.com/auditstream/auditstream/pkgs/container/auditstream)

[badge-release]: https://img.shields.io/github/v/release/auditstream/auditstream?filter=auditstream-*&style=flat&label=release
[link-release]: https://github.com/auditstream/auditstream/releases/latest
[badge-spec]: https://img.shields.io/github/v/release/auditstream/auditstream?filter=spec-*&style=flat&label=spec&color=2f6fed
[link-spec]: https://github.com/auditstream/auditstream/tree/spec-v0.1.1/spec
[badge-sonar]: https://img.shields.io/sonar/quality_gate/auditstream_auditstream?server=https%3A%2F%2Fsonarcloud.io&style=flat&label=SonarCloud
[link-sonar]: https://sonarcloud.io/project/overview?id=auditstream_auditstream
[badge-license]: https://img.shields.io/github/license/auditstream/auditstream?style=flat
[link-license]: https://github.com/auditstream/auditstream/blob/main/LICENSE
