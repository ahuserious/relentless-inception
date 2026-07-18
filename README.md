# relentless-inception

A long-running autonomous orchestrator **skill for [Claude Code](https://claude.com/claude-code)** — plans, builds, adversarially reviews, ships, and resurrects multi-day coding runs without babysitting. Its signature feature is **fusion deliberation gates**: every plan/phase/summarize checkpoint is reviewed by a multi-model panel (Claude + OpenAI seats) whose reviews are synthesized by a judge + fuser into a structured, provenance-stamped verdict. Fail-closed, always-runs, with a recorded degradation ladder.

> v0.2.1 — panel/judge/fuser fully user-configurable; codex (ChatGPT subscription) and OpenRouter backends, either or both.

## Prerequisites

1. **Claude Code** (CLI or desktop).
2. **OpenAI Codex CLI ≥ 0.144** — `npm i -g @openai/codex` — signed in with your ChatGPT account: `codex login`. This powers the gpt-5.6 panel seats through your subscription (no API key).
3. **The official codex plugin for Claude Code** (prerequisite, unmodified — this skill drives it, it is NOT forked or bundled here):
   ```
   /plugin marketplace add openai/codex-plugin-cc
   /plugin install codex@openai-codex
   ```
4. *(Optional)* An **OpenRouter API key** — enables the `openrouter` backend: direct panel seats and the one-call `openrouter/fusion` server-side fan-out.
5. `jq`, `git`, `docker`, `python3`, `perl`, `curl` on PATH (`scripts/check_prereqs.sh` verifies everything and live-probes your model seats).

## Install

```bash
git clone https://github.com/ahuserious/relentless-inception ~/.claude/skills/relentless-inception

# user config (edit to taste — this copy survives skill upgrades)
mkdir -p ~/.claude/relentless-inception
cp ~/.claude/skills/relentless-inception/assets/fusion.config.default.json \
   ~/.claude/relentless-inception/fusion.config.json

# secrets (only needed for the openrouter backend)
cp ~/.claude/skills/relentless-inception/assets/secrets.env.example \
   ~/.claude/relentless-inception/secrets.env
chmod 600 ~/.claude/relentless-inception/secrets.env
# then put your key on the OPENROUTER_API_KEY= line

# preflight — probes codex login, your panel seats, and the openrouter rung
~/.claude/skills/relentless-inception/scripts/check_prereqs.sh
```

Invoke from any Claude Code chat: `/relentless-inception <your multi-day build task>`.

## Configure your fusion panel

`~/.claude/relentless-inception/fusion.config.json`:

```jsonc
{
  "backends": { "codex_plugin": true, "openrouter": false },  // either or both
  "panel": [
    { "transport": "claude", "model": "fable-5",  "count": 1, "effort": "xhigh" },
    { "transport": "codex",  "model": "sol",      "count": 1, "effort": "xhigh" },
    { "transport": "claude", "model": "opus-4.8", "count": 1, "effort": "xhigh" }
  ],
  "judge": { "transport": "claude", "model": "fable-5", "effort": "low" },
  "fuser": { "transport": "claude-code-session", "effort": "xhigh" }
}
```

- **Transports**: `claude` = subagents inside Claude Code (no key) · `codex` = codex CLI/plugin via your ChatGPT subscription · `openrouter` = direct HTTP (needs the key in `secrets.env`).
- **codex model aliases** the skill resolves: `sol`→`gpt-5.6-sol`, `luna`→`gpt-5.6-luna`, `terra`→`gpt-5.6-terra`; any catalog slug works, **any reasoning effort** `minimal…xhigh` (`ultra` via raw CLI only).
- **Fuser = the model currently selected in your Claude Code session**, always a fresh instance. Empirically the fuser is the lever (~18-pt quality swing); the judge barely matters, so a cheap judge is always safe.
- **Temperature** applies only to openrouter-direct calls (panel 1.0, judge pinned 0). Don't temp-tune reasoning models — OpenAI o-series/gpt-5.x reasoning ignore or reject it, Claude extended thinking requires temp=1, DeepSeek-R1/Gemini thinking manage sampling internally. Temperature is not a diversity mechanism; diversity comes from distinct models and personas.

## What's inside

| Path | What |
|------|------|
| `SKILL.md` | Router: modes, team, triple gate, rescue, safety rails |
| `references/adversarial-gates.md` | **Normative fusion-gate spec**: roles, provider ladder, per-gate config, verdict schema, amendment protocol |
| `references/rescue-mode.md` | Stall/kill recovery incl. provider-capacity fast path |
| `scripts/check_prereqs.sh` | Preflight: tool checks + live ladder probes → `gate_capability.json` |
| `scripts/adversarial_review.sh` | Gate driver: runs codex/openrouter seats, hands the pre-fusion bundle to the orchestrator (exit 42) |
| `assets/verdict.schema.json` | Structured verdict with full provenance (`panel`, `judge_model`, `fuser_model`, `inputs_sha256`, ladder position, degradation flags) |
| `assets/fusion.config.default.json` | The shipped default panel config |
| `agents/` | Per-role prompt templates (panelist / judge / fuser roles included) |

## Design provenance

Gate empirics from the TrustedRouter fusion research artifact ([ahuserious/trustedrouter-fusion-artifact](https://github.com/ahuserious/trustedrouter-fusion-artifact)): generative synthesis over voting, preserve lone-correct minority findings, strongest-model fuser, cheap judge. Battle-tested building the neuro-centrifuge trading-research harness, where the panel caught donor-library determinism bugs that both the implementing agent and the orchestrator's own green test run had missed.
