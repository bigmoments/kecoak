# 🪳 kecoak

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen.svg)](#)

**Kill bugs like you kill cockroaches: get to the nest, not just the one you can see.**

*kecoak* is Indonesian for "cockroach" — the pest that never dies. Just like that bug
you "already fixed" last week. This is a discipline for AI coding agents to fix bugs
**permanently**, so they don't come back.

---

## The problem

"Already fixed" bugs keep reappearing. Almost always for one of five reasons:

1. The **symptom** was treated, not the **root cause**
2. There was **no reproduction** before the fix — so the fix was never proven
3. The fix wasn't **locked in with a regression test** — it silently regressed later
4. The **blast radius** wasn't mapped — the fix broke something else, or sibling symptoms were left untreated
5. The bug is **non-deterministic** — the "fix" only made it rarer

kecoak is a playbook that closes all five. Every time.

## Two tracks

Every session starts at **Phase 0 — Classification**, because the fix and the
verification differ between a broken feature and a security hole. When in doubt,
treat it as security until proven otherwise.

| | **Track A — Bug Rusak** (broken functionality) | **Track B — Security Hardening** |
|---|---|---|
| Mindset | Think like a *doctor* | Think like an *attacker* |
| Flow | Tight repro loop → ranked falsifiable hypotheses → root cause → sibling sweep → blast radius → minimal fix → regression test → verify | Threat model (5 min) → fix at the trust boundary → never treat symptoms → codebase-wide pattern sweep → adversarial verification |
| Verification | "The feature works normally again" | "Prove the exploit *fails*" (malicious PoC test) |
| Traps | Null-check without asking why null flows there | Hiding the admin button ≠ fixing IDOR; try/catch ≠ fixing injection |

## 🕸️ bugraph

After diagnosis, the analysis is rendered as an **interactive HTML graph** — not a wall of text:

- **Cause chain**: symptom → intermediate causes → **root cause** (highlighted)
- **Blast radius**: changed function → affected callers, each with a verification check
- **Attack path** (Track B): attacker input → trust boundary → impact
- Solid lines = verified facts · dashed lines = still hypotheses

Rendered **once per session, then delta-updated** (token-efficient), bound to the
workflow phases (not optional), and automatically saved as both an in-chat widget
**and** a standalone file: `~/workspace/bugraph/<case>/bugraph-<timestamp>.html`.

## Do & Don't (excerpt)

| ✅ DO | ❌ DON'T |
|---|---|
| Reproduce first — no repro, no fix | Fix from a guess |
| Chase the root cause; prove it with a revert-test | Add a null check without asking why it's null |
| Sweep the same pattern across the codebase | One fix for one symptom of a five-symptom root cause |
| Lock it with a regression test (red → green) | Trust a test that doesn't capture the real bug pattern |
| Security: allowlist, parameterized queries, server-side authz | Security: incomplete denylists, client-side-only validation |
| Security: prove the exploit fails | Security: only prove the happy path works |

The full table, anti-patterns, and phase checklists live in [`SKILL.md`](SKILL.md).

## Usage

**Chat triggers:** `bug muncul lagi` · `sudah di-fix tapi` · `kecoak` · `fix permanen` ·
`regression` · `ada celah` · `security hardening` · `bisa di-bypass`

**Automatic:** the agent starts at Phase 0 on any bug-fix request, and runs the full
Track B (never the shortcut) on any security finding.

## Install

Copy the `kecoak/` folder into your harness's skills directory:

| Harness | Skills directory |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Cursor | `.cursor/skills/` (project) or `~/.cursor/skills/` |
| Antigravity / Gemini CLI | `~/.gemini/skills/` |
| Codex | `~/.codex/skills/` |

Pure markdown. No hooks, plugins, MCP servers, or scripts required.

## Design principles

- **Standalone.** Zero dependencies on other skills. Everything the workflow needs is inside.
- **Phases, not advice.** Each phase has entry criteria, exit criteria, and a checklist. A session isn't done until the checklists are green.
- **YAGNI.** No ceremony without a bug. No architecture astronautics.
- **Measured, not felt.** "p95 1200ms → 320ms", not "feels faster". Red → green, not "looks fixed".

## Works well with (optional)

- [`vulture`](https://github.com/bigmoments/vulture) — call graph for blast-radius mapping (A4) and pattern sweeps (B4)
- [`app-optimizer`](https://github.com/bigmoments/app-optimizer) — performance work, *after* the bug is properly fixed

## Project structure

```
kecoak/
├── SKILL.md        # The full discipline: Phase 0, Track A (7 phases),
│                   #   Track B (6 phases), bugraph, Do & Don't, anti-patterns
├── README.md       # You are here
├── CHANGELOG.md    # Version history (Keep a Changelog)
└── LICENSE         # MIT
```

## Contributing

Bug reports and improvements welcome — preferably proven in a real debugging session
first (this skill eats its own dogfood: no repro, no fix). See [CHANGELOG](CHANGELOG.md)
for the format.

## License

MIT — see [LICENSE](LICENSE).
