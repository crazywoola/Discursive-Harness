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

## Example run

A real deliberation on the prompt *"I need a doc to claim why this is better than loop engineering"*. The harness seated three stances, ran three challenge/response rounds, closed the graph with **two theses surviving equally**, and returned `UNDERDETERMINED` — handing the normative fork back to a human rather than forcing a winner.

```ansi
[2m⊢ [0m[1mPhase 0[0m — identifying stakeholders…
[2m⊢ [0m  seat: [36mNew Approach Proponent[0m — Argues the alternative model reduces complexity, improves composability, and eliminates the failure modes inherent to hand-rolled loop logic
[2m⊢ [0m  seat: [36mLoop Engineering Defender[0m — Loop constructs are explicit, deterministic, and universally understood — novelty introduces risk without proven benefit
[2m⊢ [0m  seat: [36mOps / On-Call Engineer[0m — Cares about debuggability, observability, and what happens at 3am — whichever approach fails more silently is the enemy
[2m⊢ [0m[1mPhase 1[0m — claim round (theses + reasons)…
[2m⊢ [0m  3 theses, 9 reasons on the graph
[2m⊢ [0m[1mPhase 2 · round 1[0m — challenge & response…
[2m⊢ [0m  [31m⚔[0m New Approach Proponent undermine → r5
[2m⊢ [0m  [31m⚔[0m Loop Engineering Defender undermine → r7
[2m⊢ [0m  [31m⚔[0m Ops / On-Call Engineer undermine → r8
[2m⊢ [0m  → graph state: [35mCONTINUE[0m (open challenges: 3)
[2m⊢ [0m[1mPhase 2 · round 2[0m — challenge & response…
[2m⊢ [0m  [32m🛡[0m New Approach Proponent rebut → ch1
[2m⊢ [0m  [32m🛡[0m Loop Engineering Defender rebut → ch1
[2m⊢ [0m  [32m🛡[0m Ops / On-Call Engineer rebut → ch2
[2m⊢ [0m  → graph state: [35mCONTINUE[0m (open challenges: 1)
[2m⊢ [0m[1mPhase 2 · round 3[0m — challenge & response…
[2m⊢ [0m  [32m🛡[0m New Approach Proponent rebut → ch3
[2m⊢ [0m  [32m🛡[0m Loop Engineering Defender rebut → ch3
[2m⊢ [0m  [32m🛡[0m Ops / On-Call Engineer rebut → ch3
[2m⊢ [0m  → graph state: [35mUNDERDETERMINED[0m (open challenges: 0)
[2m⊢ [0m[1mPhase 4[0m — synthesizing verdict ledger…

[33m══════════════════════════════════════════════════════════════════════[0m
[1m[33m  VERDICT: UNDERDETERMINED[0m
[2m  Multiple theses survived equally — selection handed to humans.[0m
[33m══════════════════════════════════════════════════════════════════════[0m

[1mTASK[0m  I need a doc to claim why this is better than loop engnierring

[1mSUMMARY[0m
  The deliberation settled one point clearly: structured frameworks
  externalize decision logic in ways that make systems auditable and
  incident-recoverable — loop engineering cannot match this without
  heavy boilerplate (c3, uncontested on auditability). What remains
  genuinely contested is the core replacement question. The pro-
  alternative side (c1) argues correctness-by-construction beats
  correctness-by-discipline; the pro-loop side (c2) argues proven
  reliability and zero abstraction risk outweigh theoretical gains.
  Both positions hold defended empirical and normative grounds that
  the argument graph could not resolve. The fork is normative: does
  your team weight 'reducing the ceiling of possible errors' over
  'minimizing the floor of onboarding risk'? That is a values
  question, not a technical one.

[1m[32mSURVIVING CLAIMS[0m
  • [c1] [32mThe alternative model should replace hand-rolled loop engineering because it encodes control flow at the framework level, making iteration correct by construction rather than correct by discipline.[0m
[2m      ↳ (empirical) Hand-rolled loops accumulate failure modes—off-by-one errors, missed break conditions, state leakage between iterations—that empirically account for a disproportionate share of production bugs in long-running agent and pipeline systems.[0m
[2m      ↳ (inferential) When loop logic is lifted into a composable primitive (pipeline/parallel/phase), each stage is a pure function of its inputs; this eliminates shared mutable state and makes the control graph inspectable, resumable, and testable in isolation—properties hand-rolled loops cannot provide without significant boilerplate.[0m
[2m      ↳ (normative) Correctness should be the default, not the reward for careful engineering; a model that makes the right behavior the path of least resistance is normatively superior to one that demands discipline from every implementer.[0m
  • [c2] [32mLoop engineering should be retained as the standard approach because its explicit, deterministic control flow is universally understood, battle-tested, and introduces zero additional abstraction risk.[0m
[2m      ↳ (empirical) Decades of production software built on loop constructs demonstrates their reliability; failure modes are well-documented and engineers at all levels can reason about them without specialized knowledge.[0m
[2m      ↳ (inferential) Introducing a novel alternative imposes a learning curve and new failure modes whose edge cases are not yet understood, meaning the transition cost is certain while the benefit remains speculative.[0m
[2m      ↳ (normative) Engineering systems should prefer proven, maintainable solutions over novelty; correctness and debuggability are higher-order values than theoretical elegance.[0m
  • [c3] [32mStructured deliberation with assigned-stance agents produces more debuggable, auditable decision records than loop engineering because every reasoning step is externalized and attributable.[0m
[2m      ↳ (empirical) Loop engineering embeds decision logic in iterative runtime state that is opaque at failure time — when something goes wrong at 3am, the on-call engineer cannot reconstruct which iteration produced which side effect or why a loop exited early.[0m
[2m      ↳ (inferential) Structured discourse with explicit stances and typed grounds (empirical/inferential/normative/interpretive) creates a permanent, queryable artifact: you can grep for the normative assumption that was wrong or the empirical claim that was stale, without re-running anything.[0m
[2m      ↳ (normative) Observability is not a post-hoc concern — decision processes that cannot be inspected after the fact are operationally irresponsible because they make incident retrospectives guesswork rather than evidence-based.[0m

[1m[31mDEFEATED ALTERNATIVES[0m[2m  (reopenable)[0m
[2m  (none)[0m

[1m[35mUNRESOLVED CHALLENGES[0m
[2m  (none — graph closed)[0m

[1m[33m⟶ DECISION POINT FOR HUMAN[0m
[33m  Which risk does your organization treat as primary: (A) the failure[0m
[33m  modes that skilled engineers introduce when loop logic is hand-[0m
[33m  rolled at scale, or (B) the adoption and abstraction risk of[0m
[33m  migrating to an unfamiliar control-flow framework? The answer[0m
[33m  determines which model is 'better.'[0m
```

> The live phase log (`⊢ …`) is written to **stderr**; the verdict ledger is written to **stdout**.

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
