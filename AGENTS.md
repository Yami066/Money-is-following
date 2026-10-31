# AGENTS.md — Deception SOC (read this first)

Instructions for every coding agent working in this workspace (Claude Code, Antigravity, Codex, Cursor, …).

## 1. What we are building

A **Deception-Based SOC Platform** for a hackathon (PS 04). Fake services (decoys) attract attackers; every
interaction becomes an event; a Go engine correlates events into attacker profiles, maps them to MITRE ATT&CK,
scores risk, adapts a deception maze, alerts Discord/Slack, and a Next.js dashboard shows it all live.

Chain the demo must show: **Attack → Capture → Normalize → Analyse → Correlate → Classify → Alert → Respond.**

## 2. Workspace layout

```text
Deception-SOC/
├── AGENTS.md                ← this file
├── CLAUDE.md                ← Claude Code entry point (points here)
├── references/              ← READ-ONLY research material, never part of the product
│   ├── cowrie/  wisp/  echidra/  honeywire/  labyrinth/  codeguard/
└── deception-soc/           ← THE PRODUCT (its own git repo)
    ├── README.md
    ├── docs/                ← the specification (start at docs/00-overview.md)
    ├── backend/go/          ← Go: core engine + decoys      (owner: backend, Claude Code)
    ├── frontend/nextjs/     ← Next.js SOC dashboard         (owner: frontend, Antigravity)
    ├── ai-service/fastapi/  ← Python ML service             (owner: ML)
    ├── infrastructure/docker/
    └── docker-compose.yml
```

## 3. Reference Repository Policy

The `references/` directory contains five cloned open-source repositories:

1. Cowrie
2. Wisp
3. EchidraOSS
4. HoneyWire
5. LABYRINTH

These repositories are strictly for research and technical reference.

### Strict Rules

- DO NOT use any reference repository as the project's base.
- DO NOT fork or modify the reference repositories.
- DO NOT copy or vendor their source code into the main project.
- DO NOT import their modules or dependencies as implementation shortcuts.
- DO NOT reproduce their existing dashboards.
- DO NOT execute their deployment stacks as part of the product.

### Expected Usage

Study their:
- System architecture and design decisions.
- Honeypot and decoy implementation logic.
- Honeytoken generation and triggering mechanisms.
- Event schemas and logging mechanisms.
- Session tracking and behavioural fingerprinting.
- Threat classification and detection rules.
- MITRE ATT&CK mapping.
- Distributed sensor communication.
- Adaptive deception and AI-agent interaction techniques.

Use these findings to independently design and implement our own system.

All actual product code must reside inside `deception-soc/`.

The final implementation must use:
- Next.js for frontend.
- Go for the main backend and intelligence engine.
- PostgreSQL for storage.
- FastAPI for optional AI/ML services.

Document architectural decisions and independently developed implementations.

Preserve the separation between reference material and original project code. Review applicable licenses before incorporating any third-party code or substantial code-derived material.

### License notes (verified 30 Oct 2026 from each LICENSE file)

| Repo | License |
| --- | --- |
| Cowrie | BSD-3-Clause |
| Wisp | BSD-3-Clause |
| EchidraOSS | AGPL-3.0 |
| HoneyWire | AGPL-3.0 (its README badge text says GPLv3; the LICENSE file is AGPL-3.0) |
| LABYRINTH | AGPL-3.0 |

### CodeGuard (`references/codeguard`) — separate status

CodeGuard is the team's own earlier project (github.com/ryuma04/codeguard), not third-party code, so the five-repo
rules above do not automatically apply. Default handling, unless the team decides otherwise:

- Do not modify it in `references/` and do not copy its code into `deception-soc/`.
- The ML service may call it as an **unmodified external HTTP service** (`POST /analyze`), optional and switchable
  off with `CODEGUARD_URL=` (empty). See `docs/10-ml-service.md`.
- Our own ML models are trained from scratch inside `deception-soc/ai-service/` (its design may be studied).
- Check the hackathon's rules on prior work, and declare CodeGuard as prior work if it is used.

## 4. Hard rules (do not break)

1. **`references/` is read-only.** See section 3.
2. **License safety.** Reading AGPL code to learn an approach is fine; porting it line by line is not.
3. **Contracts are law.** `docs/02-event-schema.md`, `docs/03-database.md` and `docs/04-api-contract.md` are
   shared by all four people. Change them only by editing the doc first and telling the team.
4. **Decoys never execute attacker input.** No `os/exec`, no shell, no real network fetches, no file writes
   outside a quarantine dir. Every response is simulated.
5. **Never use real secrets as lures.** All honeytokens are generated fakes (`cmd/seed`).
6. **Decoys can only talk to the core ingest port.** No access to Postgres, the dashboard, Docker socket or
   the internet.
7. **Rules first, ML second.** Every detection must work with the ML service switched off.
8. **Correlation is association, not identity.** UI and docs say "likely same operator", never "same person".
9. **Bounded deception.** Every loop, maze and session has hard limits (`docs/08-deception-maze.md`).
10. **Keep scope.** Do not add technologies not listed in `docs/01-architecture.md` without asking.

## 5. Where to start

| You are | Read in this order |
| --- | --- |
| Backend (Go) | 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 12 → 13 |
| Frontend (Next.js) | 00 → 01 → 04 → 11 → `frontend/nextjs/AGENTS.md` |
| ML (Python) | 00 → 01 → 02 → 04 (ML section) → 10 |
| Decoys / infra | 00 → 01 → 02 → 06 → 08 → 12 → 13 |

Milestones and who-does-what: `docs/14-milestones-and-team.md`.
How to use each reference repo: `docs/15-references-guide.md`.

## 6. Conventions

- Times are UTC ISO-8601 in all APIs and storage; the UI converts to local time (team is in IST, UTC+5:30).
- IDs: events = ULID strings; sessions, actors, incidents = UUIDv4.
- Severity enum everywhere: `low | medium | high | critical`.
- Commit style: `area: short summary` (e.g. `ingest: validate envelope size`).
- Each feature lands with a test and a short note in `docs/decisions.md` if a design choice was made.
