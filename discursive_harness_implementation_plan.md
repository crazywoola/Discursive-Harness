# Discursive Agent Harness — Full Implementation Plan

**A multi-agent harness built on Habermasian discourse theory, not reflexive self-critique.**

---

## 0. Orienting principle (read this first)

This system is a deliberate departure from the current generation of agent harnesses. The current paradigm is **reflexive** (Teubner): a single generator is wrapped in a control loop that forces it to self-reflect. This plan builds something different — a **discursive** harness (Habermas): legitimacy is relocated from *internal reflection structure* to *unforced argumentation among multiple, opposed parties*.

The single load-bearing test, applied to every design decision:

> Does this move locate the legitimacy of a decision in **reasons exchanged between affected parties under power-free conditions**, rather than in the **internal reflexive structure of one system**?

If a feature only adds more self-critique to one agent, it is still reflexive law in disguise. Self-critique is *one subject reasoning with itself* (monologue). This system requires *multiple subjects reasoning with each other* (dialogue). That distinction is the watershed and governs the entire build.

**A standing caveat carried through the whole plan:** the agents are not persons. They simulate parties holding reasons; they do not actually hold interests, cannot truly be persuaded, and cannot bear normative responsibility. Therefore this harness produces a **simulation of discourse**, valuable as a *decision procedure* (multi-perspective, reasons made explicit, blind spots surfaced) but **not a source of normative legitimacy**. The most Habermasian endpoint of the system hands the final normative judgment back to real affected humans. Build accordingly.

---

## 1. Scope, goals, and non-goals

### 1.1 Goals

1. Produce decisions whose **justification is fully traceable** — every surviving claim, every defeated alternative, every unresolved disagreement recorded with its reasons.
2. Force **genuine collision of opposed reasons**, not aggregation of independent answers.
3. Eliminate, as far as engineering allows, the **distortions** Habermas names: status, rhetoric/manipulation, exclusion, and premature termination by resource exhaustion.
4. **Surface and hand back** to humans the decision points the system cannot legitimately settle.

### 1.2 Non-goals

- **Not** a fast single-answer system. It trades latency and decisiveness for traceability and honesty. "Slow" and "honestly undetermined" are *features*.
- **Not** a voting/scoring ensemble. Aggregation of preferences is explicitly forbidden as a decision mechanism.
- **Not** a replacement for human deliberation. It prepares, clears, and de-occludes the ground for human judgment.
- **Not** a general orchestrator. Orchestrator-worker (functional division of labor) is the *opposite* architecture; this is stance-based division.

### 1.3 Success criteria

| Dimension | Reflexive harness (baseline) | This system (target) |
|---|---|---|
| Output | One answer | Answer **+ reason ledger + recorded dissent** |
| Decision mechanism | Goal reached / max steps | **Argument graph closure** |
| Multi-agent rationale | Context isolation, efficiency | **Collision of opposed reasons** |
| Termination | Timeout / target met | **Consensus, underdetermination, or reasonable disagreement** — never resource exhaustion deciding the winner |
| Referee role | (n/a) | Judges **form only**, never substance |

---

## 2. Architecture overview

```
                    ┌─────────────────────────────────────────────┐
                    │              DISCURSIVE HARNESS              │
                    └─────────────────────────────────────────────┘

   Phase 0            Phase 1           Phase 2            Phase 3          Phase 4
 ┌──────────┐      ┌──────────┐     ┌─────────────┐    ┌──────────┐    ┌──────────┐
 │ Framing/ │      │  Claim   │     │  Challenge  │    │ Closure  │    │ Verdict  │
 │Stakeholder│ ──▶ │  Round   │ ──▶ │  & Response │──▶ │ Judgment │──▶ │ + Reason │
 │  ID       │      │          │     │  (core loop)│    │          │    │  Ledger  │
 └──────────┘      └──────────┘     └─────────────┘    └──────────┘    └──────────┘
      │                  │                  │                 │               │
      │                  ▼                  ▼                 ▼               │
      │            ┌────────────────────────────────────────────┐           │
      │            │           ARGUMENT GRAPH (shared state)     │           │
      │            │   Claims · Reasons · Challenges · Responses  │           │
      │            └────────────────────────────────────────────┘           │
      │                                  ▲                                    │
      │                                  │ form-only state updates            │
      │                          ┌───────────────┐                           │
      └─────── seats ───────────▶│    REFEREE    │◀──── anonymized views ────┘
                                 │ (no substance) │
                                 └───────────────┘
```

Five components:

1. **Stakeholder Identifier** (Phase 0) — determines who is in the room.
2. **Party Agents** (Phases 1–2) — each represents a stance/value/affected party, at parity.
3. **Argument Graph** — the shared, append-mostly state object; the承重墙 (load-bearing wall).
4. **Referee** — maintains *formal* graph state only; never adjudicates substance.
5. **Verdict Synthesizer** (Phase 4) — emits the reason ledger.

---

## 3. The Argument Graph (load-bearing data structure)

### 3.1 Design commitments

- **Challenges must carry reasons.** A bare "I disagree" is rejected at submission. There is no naked veto in discourse — only reasoned challenge. This is the first gate against degeneration into voting.
- **`challenge_universalizability` is a first-class attack type** reserved for normative reasons, encoding Habermas's principle (U): a norm is valid only if its consequences are acceptable to all affected.
- **`undercut` is separated from `undermine`/`rebut`** (Pollock): attacking the *inference* ("the reason doesn't license the conclusion") is distinct from attacking the reason's truth or the conclusion's truth. Omitting it lets agents spin on facts while smuggling bad inferences past.
- **Status is computed, never declared.** An agent cannot assert its own claim is "defended." Only the referee's formal propagation assigns status.

### 3.2 Schema (TypeScript reference)

```typescript
type AgentId = string;

// ── CLAIM ──────────────────────────────────────────
interface Claim {
  id: string;
  content: string;
  type: "thesis" | "sub_claim";   // thesis = candidate decision; sub_claim = supporting
  proposedBy: AgentId;            // HIDDEN from referee & from peers during attack phase
  status: ClaimStatus;           // computed by referee, never self-asserted
  createdRound: number;
}

type ClaimStatus =
  | "undefended"   // proposed, no supporting reason yet
  | "defended"     // has reasons, no open valid challenge
  | "challenged"   // >=1 open (unanswered) valid challenge
  | "withdrawn"    // retracted or superseded by amendment
  | "settled";     // survived after graph closure

// ── REASON ─────────────────────────────────────────
interface Reason {
  id: string;
  content: string;
  supports: string;              // Claim.id OR Reason.id (reasons chain)
  proposedBy: AgentId;
  groundType: GroundType;
}

type GroundType =
  | "empirical"     // fact claim      → attackable by 'undermine'
  | "inferential"   // reasoning step  → attackable by 'undercut'
  | "normative"     // value claim     → attackable by 'challenge_universalizability'
  | "interpretive"; // task/semantic understanding → attackable by 'undermine'

// ── CHALLENGE ──────────────────────────────────────
interface Challenge {
  id: string;
  target: string;                // Claim.id or Reason.id
  challengeType: ChallengeType;
  content: string;               // REQUIRED: the reason for the challenge (validated non-empty)
  proposedBy: AgentId;
  status: ChallengeStatus;
  createdRound: number;
}

type ChallengeType =
  | "rebuttal"      // even if reason holds, conclusion fails (counterexample/counter-reason)
  | "undercut"      // attacks the reason→conclusion link itself
  | "undermine"     // attacks the fact/premise the reason rests on
  | "challenge_universalizability"; // normative-only: acceptable to all affected?

type ChallengeStatus =
  | "open"          // unanswered ← the state that blocks convergence
  | "answered"      // a Response exists and is not itself successfully challenged
  | "conceded"      // accepted by target's author; target withdrawn/amended
  | "dismissed";    // FORMAL defect only (see §5.3) — never "substantively weak"

// ── RESPONSE ───────────────────────────────────────
interface Response {
  id: string;
  respondsTo: string;            // Challenge.id
  responseType: "rebut" | "amend" | "concede";
  content: string;               // rebut→counter-reason; amend→new claim; concede→capitulation
  newClaimId?: string;           // present iff responseType === "amend"
  proposedBy: AgentId;
  createdRound: number;
}

// ── GRAPH ──────────────────────────────────────────
interface ArgumentGraph {
  claims: Map<string, Claim>;
  reasons: Map<string, Reason>;
  challenges: Map<string, Challenge>;
  responses: Map<string, Response>;
  // edges are implicit via supports / target / respondsTo / newClaimId
}
```

### 3.3 Invariants (enforce in code, test continuously)

- I1. Every `Challenge.content` is non-empty and topically bound to its `target` (else rejected at submission, not dismissed later).
- I2. No `Claim.status` is ever written by a party agent — only by `refereeUpdate`.
- I3. Author identity is stripped from any graph view passed to an agent during Phase 2.
- I4. The graph is append-mostly: nodes are added or status-transitioned; content is never silently rewritten. `amend` creates a *new* claim and `withdraw`s the old — history is preserved for the ledger.
- I5. A `dismissed` challenge must reference exactly one enumerated formal defect (§5.3).

---

## 4. The deliberation protocol (state machine)

### 4.1 Phases

```python
def deliberate(task, config):
    graph = ArgumentGraph()

    # ── Phase 0: Framing / Stakeholder identification ──
    stakeholders = identify_stakeholders(task)        # see §6
    assert_no_empty_seat(stakeholders, task)          # anti-domination gate #1
    agents = [instantiate_party_agent(s) for s in stakeholders]

    # ── Phase 1: Claim round ──
    for agent in shuffled(agents):                    # randomize order: no序位 advantage
        claim = agent.propose_thesis(task)
        reasons = agent.give_reasons(claim)           # protocol-forced: claim needs reasons
        graph.add_claim(claim, reasons)
    referee_update(graph)

    # ── Phase 2: Challenge & Response (core loop) ──
    for round_i in range(config.max_rounds):
        view = graph.view(anonymize_authorship=True)  # anti-domination gate #2
        actions = []
        for agent in shuffled(agents):
            action = agent.act(view, task)            # challenge | respond | pass
            if action.kind == "challenge":
                if not validate_challenge(action, graph):  # gate #3: reasoned, novel, live
                    continue
            actions.append(action)
        graph.apply(actions)
        referee_update(graph)                         # form-only state propagation

        state = convergence_state(graph, config)
        if state != ConvergenceState.CONTINUE:
            break

    # ── Phase 4: Verdict ──
    return synthesize_verdict(graph, state)
```

### 4.2 `validate_challenge` — the submission gate

Rejects (silently drops, does **not** record as dismissed):

- **Bare negation:** empty or contentless `content`.
- **Dead target:** target already `conceded` / `withdrawn`.
- **Substantive duplicate:** semantically equivalent to an existing live challenge — prevents *filibuster by repetition*, which is the **reverse** domination move (manufacturing fake "open challenges" to block closure forever).

> Note the symmetry: the protocol must resist domination from **both** directions — a powerful party forcing premature closure, *and* an obstructionist party preventing any closure through车轮战 (wheel-war repetition).

---

## 5. The Referee (and why it must stay weak)

### 5.1 The central danger

If the referee adjudicates **substance**, "power-free discourse" is a lie — you have merely moved the king from the lead agent to the judge. The entire system's integrity rests on the referee being **formally constrained**: it propagates graph status by deterministic rules and never rules on whether a reason is *good*.

### 5.2 Referee logic (deterministic, content-blind)

```python
def referee_update(graph):
    # 1. Challenge status from presence/quality-of-form of responses
    for ch in graph.challenges.values():
        if ch.status in ("conceded", "dismissed"):
            continue
        resp = responses_to(graph, ch.id)
        if resp and not response_itself_open_challenged(graph, resp):
            ch.status = "answered"
        else:
            ch.status = "open"

    # 2. Claim status from open challenges + reason presence
    for cl in graph.claims.values():
        if cl.status in ("withdrawn",):
            continue
        open_ch = [c for c in attacks_on(graph, cl.id) if c.status == "open"]
        if open_ch:
            cl.status = "challenged"
        elif has_reasons(graph, cl.id):
            cl.status = "defended"
        else:
            cl.status = "undefended"

    # 3. Propagate along support edges:
    #    a reason whose ground was undermined drags its supported claim down
    propagate_along_support_edges(graph)
```

### 5.3 The ONLY content the referee may touch: formal `dismissed`

A challenge may be `dismissed` **only** for an enumerated formal defect:

1. **Non-responsiveness** — `content` does not address `target` (topic mismatch).
2. **Circularity** — the challenge's reason presupposes its own conclusion.
3. **Settled repetition** — re-raising a challenge already `answered`/`conceded` with no new content.

**Forbidden:** dismissing a challenge because the referee judges its substantive force weak. Substantive weakness may only manifest as *being successfully answered*. This rule is the difference between the system holding and collapsing. Encode it as a hard assertion with a dedicated test suite (§9).

---

## 6. Phase 0 — Stakeholder identification (second load-bearing wall)

The whole deliberation's legitimacy is capped by *who was let into the room*. A flawless argument graph over a quietly amputated stance-set is a polished injustice. This phase is not a preamble — it is structural.

### 6.1 Procedure

```python
def identify_stakeholders(task):
    # a) First-pass: directly named/obvious affected parties & competing values
    direct = extract_direct_stakeholders(task)

    # b) Second-pass: WHO IS AFFECTED BUT UNREPRESENTED?
    #    Explicit search for silenced/absent positions — the差序格局 pathology
    #    is precisely that the weak affected party is simply not present.
    latent = search_absent_affected(task, direct)

    # c) Adversarial completeness check: an independent agent whose ONLY job
    #    is to name a stakeholder the set has missed. Run until it can name none.
    while True:
        missing = devils_advocate_missing_party(direct + latent, task)
        if missing is None:
            break
        latent.append(missing)

    # d) Assign a party agent to every identified stance, including absent ones
    return dedupe(direct + latent)
```

### 6.2 Anti-domination notes

- The default failure mode is representing **only the commissioning party's** viewpoint. Phase 0 exists specifically to break that.
- Absent/weak parties get an **assigned advocate agent** so their reasons enter the graph even though they are not "in the room" in reality.
- The completeness check (`devils_advocate_missing_party`) is itself recorded in the ledger — *who we realized we'd left out, and when* — because that is auditable evidence of de-occlusion.

---

## 7. Convergence & termination

```python
class ConvergenceState(Enum):
    CONTINUE = auto()
    RATIONAL_CONSENSUS = auto()
    UNDERDETERMINED = auto()
    REASONABLE_DISAGREEMENT = auto()

def convergence_state(graph, config):
    open_ch = [c for c in graph.challenges.values() if c.status == "open"]
    theses = [c for c in graph.claims.values() if c.type == "thesis"]
    surviving = [c for c in theses if c.status == "defended"]

    if not open_ch:                                  # graph closed
        if len(surviving) == 1:
            return ConvergenceState.RATIONAL_CONSENSUS
        if len(surviving) > 1:
            return ConvergenceState.UNDERDETERMINED  # equally defensible → DO NOT force-pick
    if no_new_arguments_for_n_rounds(graph, n=config.stall_rounds):
        return ConvergenceState.REASONABLE_DISAGREEMENT
    return ConvergenceState.CONTINUE
```

**Three legitimate terminations — none is "time ran out, leader wins":**

| State | Meaning | Output behavior |
|---|---|---|
| `RATIONAL_CONSENSUS` | One thesis survives, graph closed | Single decision + reason chain |
| `UNDERDETERMINED` | Multiple theses survive equally | Return **all**, with their reasons; hand selection to human |
| `REASONABLE_DISAGREEMENT` | Open challenge persists, debate stalled | **Not a failure** — structured map of *why* consensus is impossible |

Termination is **never** bound to budget/timeout deciding a winner. Resource limits may *halt* the process, but a halt under unresolved challenges yields `REASONABLE_DISAGREEMENT`, not a victory for whoever was ahead.

---

## 8. Phase 4 — Verdict & the Reason Ledger

The deliverable is **not an answer**. It is a ledger; the answer is one field inside it.

```python
def synthesize_verdict(graph, state):
    return {
        "outcome_type": state.name,                       # consensus | underdetermined | reasonable_disagreement
        "surviving_claims": serialize_with_reason_chains(graph),
        "defeated_alternatives": [                        # MUST be preserved
            {"claim": c, "killed_by": ch_id, "why": explanation}
            for c, ch_id, explanation in defeated(graph)
        ],
        "unresolved_challenges": serialize_open(graph),   # honestly exposed
        "excluded_then_included": phase0_recovered_parties(graph),
        "decision_point_for_human": human_handback(graph, state),
    }
```

- **`defeated_alternatives` must be retained** — this is the spine of traceable justification. Anyone may later ask "why not B?" and get "B was killed by challenge #17 (a counterexample), and no one could answer it." The ledger can be *reopened*: a decision's legitimacy lives in the ledger, not the conclusion. The conclusion is overturnable by anyone who can answer the challenge that killed the alternative.
- For `UNDERDETERMINED` / `REASONABLE_DISAGREEMENT`, `decision_point_for_human` names the precise normative fork handed back to real affected people.

---

## 9. Testing & validation strategy

The correctness that matters here is **procedural integrity**, not answer accuracy. Test the *process*, especially its anti-domination properties.

### 9.1 Referee neutrality suite (highest priority)

- **Substance-blindness:** feed the referee challenges of varying substantive quality but identical form; assert status assignments are *identical*. The referee must not favor "stronger" reasons.
- **Dismissal restriction:** property test asserting no challenge is ever `dismissed` except for the three enumerated formal defects. Any other dismissal path = build-breaking failure.

### 9.2 Anti-domination suite

- **Anonymity leak test:** assert no agent-facing graph view contains `proposedBy` during Phase 2. Fuzz for indirect leaks (e.g., stylistic fingerprints in `content`).
- **Status-suppression test:** inject a "lead agent" with high-confidence assertions; assert its claims gain no status advantage absent surviving reasons.
- **Filibuster test:** inject an obstructionist repeating equivalent challenges; assert `validate_challenge` collapses them and closure remains reachable.
- **Premature-closure test:** exhaust budget with open challenges live; assert outcome is `REASONABLE_DISAGREEMENT`, never a winner.

### 9.3 Stakeholder-completeness suite

- Seed tasks with a known-omitted affected party; assert Phase 0's adversarial check recovers it and logs it in `excluded_then_included`.

### 9.4 Ledger-integrity suite

- Assert every defeated alternative carries a non-empty `killed_by` + `why`.
- Assert the ledger is *reopenable*: reconstruct the graph from the ledger and verify the kill-chain replays.

### 9.5 Anti-degeneration ("is this still discourse?") red-line checks

Continuous assertions encoding §0:

1. **Medium check:** agents exchange *reasons*, not scored answers. Flag any path that selects by aggregation.
2. **Symmetry check:** no agent's output is structurally privileged.
3. **Output check:** delivered artifact is answer **+ ledger + recorded dissent**, never the bare answer.

---

## 10. Build sequence (phased roadmap)

| Phase | Deliverable | Exit criterion |
|---|---|---|
| **M0 — Skeleton** | Argument Graph data structures + invariants I1–I5; referee `refereeUpdate` (form-only) | Referee neutrality suite (§9.1) green |
| **M1 — Core loop** | Phase 1–2 protocol, `validate_challenge`, anonymized views | Filibuster + premature-closure tests (§9.2) green |
| **M2 — Termination** | `convergence_state`, three terminal states | Underdetermined & reasonable-disagreement paths emit correctly |
| **M3 — Ledger** | `synthesize_verdict`, defeated-alternative retention, reopenability | Ledger-integrity suite (§9.4) green |
| **M4 — Stakeholders** | Phase 0 identification + adversarial completeness check | Stakeholder-completeness suite (§9.3) green |
| **M5 — Human handback** | `decision_point_for_human`, escalation interface to real affected parties | Underdetermined/disagreement outcomes route to humans with the precise fork |
| **M6 — Hardening** | Full anti-degeneration red-lines (§9.5) as CI gates; performance/latency budgets that *halt* without *deciding* | All §9 suites green in CI |

**Sequencing rationale:** the referee (M0) is built and locked *before* any party agents exist, because everything downstream trusts its neutrality. Stakeholder identification (M4) comes relatively late in code but is conceptually the second承重墙 — it is sequenced after the graph/loop are proven so that completeness failures are visible against a working deliberation, not hidden inside an unproven loop.

---

## 11. Known tensions & honest limits

1. **Simulation, not legitimacy.** Agents simulate parties; they hold no real interests and bear no responsibility. As a *decision procedure* this can yield more robust, traceable, less blind-spotted outputs — a real engineering gain. As a *source of normative legitimacy* it cannot substitute for the assent of actually affected people. The system's most honest endpoint requests human judgment at the normative fork.

2. **Efficiency optimizations are the chief threat to pluralism.** Every "merge similar agents," "prune weak claims early," "cap rounds to converge faster" silently shrinks the argumentative space toward the single-subject reflection you set out to escape. **Rule:** before adding any efficiency optimization, ask whether it removes *redundancy* (allowed) or removes a *genuine distinct stance* (self-destruction). The former is fine; the latter is forbidden.

3. **Slowness and honest no-answer are intended.** A discursive harness that frequently returns `UNDERDETERMINED` or `REASONABLE_DISAGREEMENT` is functioning correctly, not failing. The value of the procedure is not to replace judgment but to **force silenced stances into the open and expose discretion to accountable light.**

4. **The referee/stakeholder pair is the whole game.** Two failure modes collapse everything: a referee that adjudicates substance (re-introduces a sovereign), and a stakeholder phase that omits affected parties (deliberation over a quietly amputated set). Guard both with the dedicated suites in §9 and treat any regression as build-breaking.
