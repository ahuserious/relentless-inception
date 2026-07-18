# plugin-dev handoff

Paste-able context for the dedicated **plugin-dev** chat. Scope of that chat: evolve this skill + its fusion machinery. Neuro-centrifuge harness dev stays in its own chat.

## Current state (2026-07-18, v0.2.1)

- Fusion gates shipped and battle-proven: gate-1 (rung-3 claude-panel) and gate-2 (rung-2 codex panel sol/luna/terra + Claude fuser) both ran for real on the neuro-centrifuge P1b run; gate-2's panel caught two donor determinism bugs (HashMap-ordered Pareto selection + minibatch padding) that the dev worker's green tests masked.
- Panel default per owner preference: 1–2× fable-5 + 1–2× gpt-5.6-sol xhigh + 1–2× opus-4.8 xhigh; judge default fable-5; fuser = Claude Code session model. luna/terra out of defaults.
- Config: `~/.claude/relentless-inception/fusion.config.json`; secrets: `~/.claude/relentless-inception/secrets.env` (OPENROUTER_API_KEY).
- codex plugin (`codex@openai-codex` v1.0.3) is an unmodified upstream prerequisite; all customization lives skill-side (alias map, any-effort, panel wiring). codex CLI must be ≥0.144 for gpt-5.6 (0.142 gets "requires a newer version of Codex").

## Backlog (from the 9-agent grading + run experience, priority order)

1. Rung-1 revival test once a valid OPENROUTER_API_KEY exists (current key 401s) — end-to-end `openrouter/fusion` gate + pricing ledger reconciliation (fusion bills N panel + 2× judge).
2. Vendor the claude-fusion plugin's verbatim panelist/judge/fuser prompts (local repo `~/hyperfrequency/claude-fusion`) with its two known bugs fixed: codex `--effort` parsed-but-dropped; `__openrouter` sentinel leaking into HTTP headers. Gate-specific preamble over rewrite.
3. Run-scoped stall watchdog with a real scheduler (the v0.1 watchdog never fired; Stop-hook state is polluted by unrelated sessions).
4. Budget ledger enforcement: sum `ledger.jsonl` into a standing phase-gate input; hard caps from prose → counters.
5. Task-granular dependency encoding persisted outside session state (edges.json).
6. Degraded summarize gate wired to fire on every compaction + rescue resume.
7. Mixed-transport driver: `adversarial_review.sh` currently executes codex/openrouter seats and defers claude seats to the orchestrator (exit 42) — consider a companion mjs that also drives claude seats headlessly via `claude -p`.
8. Worktree-isolation quirk: subagent worktrees key off the orchestrator's CWD repo — first dispatch keyed the wrong repo. Guard: every dev-worker prompt must include a "verify your worktree remote" preamble (already standard practice; consider making it a script).

## Distribution

Public home: https://github.com/ahuserious/relentless-inception (this repo). Install instructions in README.md. Owner's private canonical copies: `~/.claude/skills/relentless-inception/` (live) and `~/meta_skill/skills/relentless-inception/` (curated tier) — keep the three in sync when landing changes.
