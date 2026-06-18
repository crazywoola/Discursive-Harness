# Discursive Harness

**A multi-agent deliberation harness built on Habermasian discourse theory — not reflexive self-critique.**

Most agent harnesses are *reflexive*: a single generator wrapped in a control loop that forces it to self-reflect (one subject reasoning with itself — monologue). The Discursive Harness is *discursive*: it stages a **dialogue** among multiple, opposed party-agents whose legitimacy lives in **reasons exchanged under power-free conditions**, recorded in a reopenable **reason ledger**.

The single load-bearing test applied to every design decision:

> Does this move locate the legitimacy of a decision in **reasons exchanged between affected parties under power-free conditions**, rather than in the **internal reflexive structure of one system**?

The reference implementation is **`jurgen`** (named for Jürgen Habermas), a CLI in which the "fundamental agent" is **Claude Code itself** — every party-agent, stakeholder-finder, and synthesizer is a headless `claude -p` invocation. The **referee is deliberately *not* an agent**: it is deterministic, content-blind Python. That separation is the whole game.

---

## Why this is different

| Dimension | Reflexive harness (baseline) | Discursive Harness |
|---|---|---|
| Output | One answer | Answer **+ reason ledger + recorded dissent** |
| Decision mechanism | Goal reached / max steps | **Argument-graph closure** |
| Multi-agent rationale | Context isolation, efficiency | **Collision of opposed reasons** |
| Termination | Timeout / target met | **Consensus, underdetermination, or reasonable disagreement** — never resource exhaustion deciding the winner |
| Referee role | (n/a) | Judges **form only**, never substance |

It is **not** a fast single-answer system, **not** a voting/scoring ensemble, and **not** a replacement for human deliberation. It trades latency and decisiveness for traceability and honesty — "slow" and "honestly undetermined" are *features*.

---

## Install / requirements

- Python 3.9+
- [Claude Code](https://claude.com/claude-code) CLI (`claude`) installed and authenticated — it is the agent backbone.

```bash
git clone https://github.com/crazywoola/Discursive-Harness.git
cd Discursive-Harness
chmod +x jurgen
```

## Usage

```bash
# Simplest form — pass the question as one quoted argument
./jurgen "Should our team adopt a 4-day work week?"

# Deeper debate: more rounds + more stakeholders
./jurgen "Should we open-source our core engine?" --max-rounds 3 --max-stakeholders 4

# Save the machine-readable ledger and see the pretty output
./jurgen "Should we move to usage-based pricing?" --out ledger.json

# Raw JSON only (pipe into other tools)
./jurgen "Should we require RTO 3 days a week?" --json > ledger.json

# Use Opus for sharper arguments (slower, pricier)
./jurgen "Should we acquire competitor Y?" --model claude-opus-4-8
```

The `deliberate` keyword is optional: `./jurgen deliberate "<task>"` works identically.

### Flags

| Flag | Default | Meaning |
|---|---|---|
| `--model` | `claude-sonnet-4-6` | Model for the party-agents |
| `--max-rounds` | `3` | Max challenge/response rounds |
| `--stall-rounds` | `2` | Rounds with no progress → reasonable disagreement |
| `--max-stakeholders` | `4` | Cap on seated parties |
| `--json` | off | Emit raw ledger JSON only |
| `--out FILE` | — | Write ledger JSON to a file |
| `--quiet` | off | Suppress phase logging |

### What you'll see

- **stderr (live):** seated parties → theses → `⚔` challenges and `🛡` responses per round → graph state.
- **stdout (final):** the ledger — `SURVIVING CLAIMS` with reason chains, `DEFEATED ALTERNATIVES` (reopenable, with what killed them), `UNRESOLVED CHALLENGES`, and — for `UNDERDETERMINED` / `REASONABLE DISAGREEMENT` outcomes — the `DECISION POINT FOR HUMAN`.

---

## How it works

```
 Phase 0          Phase 1          Phase 2             Phase 4
┌──────────┐    ┌──────────┐    ┌─────────────┐     ┌──────────┐
│ Framing /│    │  Claim   │    │  Challenge  │     │ Verdict  │
│Stakeholder│──▶│  Round   │──▶ │ & Response  │ ──▶ │ + Reason │
│   ID      │    │          │    │ (core loop) │     │  Ledger  │
└──────────┘    └──────────┘    └─────────────┘     └──────────┘
                         │              │
                         ▼              ▼
                ┌────────────────────────────────┐
                │   ARGUMENT GRAPH (shared state) │
                │ Claims·Reasons·Challenges·Resp. │
                └────────────────────────────────┘
                              ▲
                              │ form-only state updates
                       ┌─────────────┐
                       │   REFEREE    │  (deterministic, no substance)
                       └─────────────┘
```

- **Phase 0 — Stakeholder identification.** Names the distinct stances/affected parties, then runs an adversarial "who did we leave out?" pass; recovered parties are logged as `excluded_then_included`.
- **Phase 1 — Claim round.** Each party proposes a thesis and its reasons (every reason carries a `groundType`: empirical / inferential / normative / interpretive).
- **Phase 2 — Challenge & response (core loop).** Agents see an **author-anonymized** graph view and either challenge (with a *required reason* — no bare veto), respond, or pass. A submission gate drops bare negations, dead targets, and duplicate challenges (anti-filibuster).
- **Referee.** Deterministic, content-blind status propagation. It may only formally `dismiss` a challenge for an enumerated defect (non-responsiveness, circularity, settled repetition) — **never** for being "substantively weak."
- **Termination.** One of three legitimate states — never "time ran out, leader wins":

  | State | Meaning |
  |---|---|
  | `RATIONAL_CONSENSUS` | One thesis survives, graph closed |
  | `UNDERDETERMINED` | Multiple theses survive equally — selection handed to humans |
  | `REASONABLE_DISAGREEMENT` | Open challenge persists — a structured map of *why* consensus is impossible |

- **Phase 4 — Reason ledger.** The deliverable is **not an answer**; the answer is one field inside a ledger that retains defeated alternatives and is *reopenable*: anyone who can answer the challenge that killed an alternative can overturn the conclusion.

See [`discursive_harness_implementation_plan.md`](discursive_harness_implementation_plan.md) for the full design.

---

## Honest limits

The agents are **not persons** — they simulate parties holding reasons; they hold no real interests and bear no normative responsibility. This harness therefore produces a **simulation of discourse**: valuable as a *decision procedure* (multi-perspective, reasons made explicit, blind spots surfaced) but **not a source of normative legitimacy**. Its most honest endpoint hands the final normative judgment back to actually-affected humans.

---

## License

[MIT](LICENSE) © 2026 Crazywoola
