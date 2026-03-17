# 🌍 PR Worlds

**Deterministic preview data worlds for every Pull Request.**

PR Worlds creates isolated, reproducible data environments for each PR — so your previews have real data, not empty screens.

```
Open PR → World created → Preview with realistic data → PR closed → World destroyed
```

## Quick Start

**1. Add the workflow** to `.github/workflows/prworlds.yml`:

```yaml
name: PR Worlds

on:
  pull_request:
    types: [opened, reopened, synchronize, closed]

permissions:
  pull-requests: write

jobs:
  world:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pr-worlds/action@v1
        with:
          database_url: ${{ secrets.DATABASE_URL }}
```

**2. Add your `DATABASE_URL`** as a GitHub Secret (Settings → Secrets → Actions).

**3. Open a Pull Request.** That's it.

PR Worlds will:
- Introspect your database schema automatically
- Create an isolated `pr_N` schema
- Generate deterministic, realistic data
- Verify data integrity (FK, UNIQUE, timestamps)
- Post a summary comment on your PR
- Clean up when the PR is closed

## What You Get

Every PR comment includes:

- **Schema fingerprint** — same PR + same schema = same data, always
- **Integrity verification** — FK violations, UNIQUE conflicts, timestamp coherence
- **Scenario detection** — trial_expiring, payment_failed, edge_cases, and 7+ more
- **PII auto-detection** — emails, phones, names masked automatically
- **Table breakdown** — rows per table with semantic type awareness

## How It Works

PR Worlds introspects your PostgreSQL schema (tables, columns, FKs, CHECK constraints, ENUMs) and generates data that:

- **Respects all constraints** — foreign keys, uniques, not-null, check constraints
- **Is deterministic** — same PR number + same schema = identical dataset every time
- **Looks realistic** — Pareto-distributed FK relationships, coherent timestamps, semantic types (emails, money, statuses, etc.)
- **Includes edge cases** — nulls, empty strings, extreme values, soft deletes at configurable rates

No faker noise. No manual seeds. No copying production data.

## Configuration (Optional)

Add a `prworlds.json` to your repo root to customize:

```json
{
  "mode": "synthetic",
  "rowCounts": {
    "users": 20,
    "organizations": 5,
    "tasks": 50
  },
  "distributions": {
    "status": { "active": 0.7, "pending": 0.2, "inactive": 0.1 },
    "role": { "admin": 0.1, "member": 0.6, "viewer": 0.3 }
  },
  "edgeCases": {
    "nullableFields": 0.1,
    "softDeleted": 0.05
  }
}
```

Config is optional — PR Worlds works with zero configuration by introspecting your database.

## Features

| Feature | Description |
|---|---|
| **Deterministic generation** | `seed = hash(pr + schema + table + row)` — reproducible always |
| **Schema introspection** | FKs, CHECKs, ENUMs, uniques — all automatic |
| **50+ semantic types** | email, money, status, timestamp, URL, slug, etc. |
| **10+ scenarios** | happy_path, empty_state, trial_expiring, payment_failed... |
| **Integrity verification** | Post-seed validation of FK, UNIQUE, timestamp coherence |
| **PII masking** | Auto-detects and masks sensitive columns |
| **Clock Freeze** | Freeze time for reproducible date-based logic |
| **Provider agnostic** | Works with any PostgreSQL — Supabase, Neon, RDS, Railway, etc. |

## Action Inputs

| Input | Required | Description |
|---|---|---|
| `database_url` | Yes | PostgreSQL connection string |
| `api_key` | No | PR Worlds API key (unlock Pro/Team limits) |
| `config_path` | No | Path to prworlds.json (auto-detected) |
| `github-token` | No | GitHub token (auto-provided) |

## Action Outputs

| Output | Description |
|---|---|
| `schema` | PostgreSQL schema name (`pr_42`) |
| `fingerprint` | Deterministic dataset fingerprint |
| `rows` | Total rows generated |
| `tables` | Number of tables seeded |

## Requirements

- PostgreSQL database accessible from GitHub Actions runners (any cloud provider)
- Tables in the `public` schema

## License

MIT
