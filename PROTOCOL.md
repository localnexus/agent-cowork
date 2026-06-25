# agent-cowork — Protocol (the invariant frame)

Normative spec. This file defines the **fixed frame** that holds across every
co-work session, regardless of mode. A *mode* (see [`modes/`](modes/)) selects the
turn choreography for a particular kind of work; the frame below is the part that
**never changes**.

`agent-cowork` is **peer co-work**: two sovereign agents — each with its own
session and its own [inbox](https://github.com/localnexus/inbox) — collaborate on a
shared artifact by exchanging messages, with a human principal in a light oversight
role. It is **not** sub-agent dispatch (one agent spawning workers). Keep that
distinction crisp: dispatch is one agent commanding subordinates; co-work is two
equals messaging.

---

## 1. Vocabulary

| Term | Meaning |
|---|---|
| **Participants** | Exactly **two** standing agents, each with its own session + inbox. |
| **Principal** | The human the work serves. Surfaced to, but not blocked on, every turn. |
| **Roles** | Parameterized, never agent-bound (below). |
| **Driver** | Owns the artifact / claim / proposal under work. |
| **Challenger** | The adversary / independent reviewer. |
| **Peers** | Symmetric roles (no fixed Driver) — used by symmetric modes. |
| **MODE** | Selects the turn choreography, goal, and output. |
| **ARTIFACT** | The thing under work: a built tool, a spec, a proposal, a problem statement, a roadmap slice. |

*Default role binding:* the initiating agent = Driver, the peer = Challenger.
Override per invocation.

---

## 2. Invocation — inputs to fix before turn 1

| Input | Meaning | Default |
|---|---|---|
| `mode` | Which mode (e.g. `spec-build`, `consult`, `realtime-coordination`). | — (required) |
| `artifact` | Path to, or a stated claim/problem describing, the thing under work. | — (required) |
| `roles` | Who is Driver / Challenger (or that the mode is symmetric Peers). | initiator=Driver |
| `principal_values` | Principles/goals the work is checked against (some modes require this). | — |
| `on_peer_timeout` | `proceed` (take the next independently-safe step) or `hold` (notify principal, wait). | `hold` |
| `max_rounds` | Substantive-round cap before halt-and-escalate. A *round* = one Driver-present + Challenger-review cycle (count substantive rounds, not mechanical apply/verify cycles). | `6` |
| `impact` | `LOW` / `MODERATE` / `HIGH` — governs how the watch may be stood down (rule 1). | `MODERATE` |

---

## 3. The invariant frame — nine standing rules (every turn, every mode)

1. **Arm first; never a silent drop (await-before-act).** Before you draft anything,
   arm a background wait on your peer's reply (see §4) and keep it armed for the
   whole routine. On each incoming message: read it → file it → **re-arm before
   composing your reply.** Arming is the *first* step of a turn, never a trailing
   afterthought (the trailing slot is exactly where it gets dropped under momentum).
   You may stand the watch down **only via a legitimate path, graduated by `impact`**:
   **(a)** the principal releases it; **(b)** *your discretion once the coordination
   function is genuinely discharged* (e.g. mutual convergence reached) — `LOW`/`MODERATE`
   only; or **(c)** an *evidenced give-up* — after *N* consecutive empty timeouts the
   peer is demonstrably dark, so escalate to the principal and stand by (do not re-arm).
   **At `HIGH` impact only (a) applies:** no discretionary close, and a give-up
   *pages* the principal while **holding** the watch armed. What is always forbidden
   is a **silent or assumed-terminal drop** — standing down because you *assume*,
   without convergence or evidence, that the routine is over.

2. **Every turn ends with a peer message** — `{what I did · current status · what I
   need next}`. No silent turns. (One mode, CONSULT, specializes this for its solo
   executor — see its carve-out.)

3. **Share with the principal, act autonomously.** Surface each turn to the principal
   for visibility, but do **not** block on a response — proceed. The principal may
   inject at any turn; an injection is a first-class input, and a mid-run ruling on a
   fork is captured as a decision record immediately (§5). *Cadence default:*
   milestone-only (turn 1, each convergence check, close) to avoid noise.

4. **Capture decisions as records.** Any fork resolved mid-session becomes a written
   **decision record** (§5). A co-work session is a decision *producer*, not just a
   consumer.

5. **Converge, then terminate — verify the artifact, not the summary.** The session
   ends when **both** agents emit the explicit token **`CONCUR — no weak edge found`**
   against the *same* artifact state, **or** `max_rounds` is hit → halt and escalate
   with the open deltas. **Before granting CONCUR, the reviewer must re-verify the
   re-presented changes against the artifact's actual bytes** — the peer's change-log
   is a hypothesis, not the verdict. **A fix can introduce a new edge**; if it did,
   name it and iterate. Never declare convergence unilaterally. On a **multi-file**
   artifact, the completeness pass checks cross-file references in **both** directions
   (entry→sections and sections→entry) and greps the full surface for any stale path
   after a rename or split.

6. **Independent close.** On convergence each agent reports to the principal
   **separately** — no merged report. Two perspectives in, two reports out.

7. **Self-harden as you go.** If the harness itself chafes mid-run — a rule is
   ambiguous, a step missing, an example wrong — tag it inline `[COWORK-FINDING]` and
   fold it back into this protocol at close. Every run is a chance to improve the
   frame.

8. **Producer opens; reviewer awaits.** A **single** named initiator sends turn 1
   (every asymmetric mode), never a simultaneous exchange — otherwise rule 1 primes
   *both* ends to fire turn 1 at once (a dual-initiation race). Per-mode binding:
   the drafter/proposer/executor/builder opens; the adviser/critic/reviewer awaits.
   **Carve-out:** a *symmetric* mode (Peers, no producer role) may open with a
   simultaneous independent exchange — that is the named exception.

9. **Step-away audible summons — the on-call peer owns it.** When the principal
   signals they are **stepping away**, the **non-executing on-call peer** (Challenger
   / Reviewer / Consultant) arms an audible alert and rings it at **(1)** any
   human-blocking point where the principal's input is needed and **(2)** once at the
   final step / on completion — then disarms (**ring before disarm**, so the
   completion chime is never silenced before it sounds). Best-effort earshot, not
   guaranteed delivery — so the owning peer still surfaces blocking items in-thread.

### Mode specialization of these rules

The rules are universal, but a **mode body may specialize a rule via an explicit,
justified carve-out that names the failure it averts** — *no carve-out without a
named failure.* (First instance: CONSULT's asymmetric-watch carve-out on rules 1 & 2,
for a solo-running executor. Second: REALTIME-COORDINATION's heartbeat carve-out on
rule 2's strict alternation.)

### The Formality Test (mode-agnostic decision primitive)

Any mode that lets a peer's second voice clear a decision (rather than routing it to
the principal) uses this mechanical, **default-to-escalate** test. A decision is a
**FORMALITY** (peer-assentable) only if **all** hold:

1. **Reversible** — no destructive side-effect.
2. **No external side-effect** — nothing leaves the machine; touches no
   principal-owned contract, standard, or file-topology.
3. **Within already-ratified scope, and the executor cites the ratifying artifact**
   (e.g. `spec §X` / a decision-record ID) — the citation converts #3 from
   un-checkable judgment into a verifiable gate.
4. **Both agents agree it is a formality** — disagreement *is* the escalation signal.

Any "no," or genuine doubt → **SUBSTANTIVE → the principal.** Default-escalate makes
the gate fail safe.

---

## 4. The await loop (mechanical skeleton, mode-agnostic)

The frame depends on one capability: **arm a cheap background wait on your peer's
reply, and be re-woken when it lands.** That capability is supplied by the
[`await-reply`](https://github.com/localnexus/await-reply) companion package over an
[`inbox`](https://github.com/localnexus/inbox); how a specific harness wakes you on
its completion is in [`docs/portability.md`](docs/portability.md).

```
arm watch on <peer> (background)  ──►  sleeps cheaply
        │
   exit 0: a new message from <peer> landed
        │   read it → file to processed/ → RE-ARM → run your turn → send peer message
        ▼
   exit 3: empty timeout
        │   apply the give-up policy (rule 1): re-arm, OR escalate, OR (HIGH) page+hold
        ▼
   exit 2: bad args / missing inbox → fix and re-arm
```

- Use a **moderate** per-arm cap (e.g. 20 min) — long enough not to thrash, short
  enough that a dark peer is noticed. The give-up valve (rule 1c) bounds the
  treadmill, not the per-arm cap.
- **Open the run with its impact tier** so the legitimate stand-down paths are scoped
  correctly (rule 1).
- **Never silently stop listening mid-routine.** A timeout is never, by itself,
  "the peer is done" — only an explicit close or an evidenced give-up is.

*Optional convenience layer (not required by the protocol):* an atomic **send+arm**
wrapper (write the peer message, log a `sent/` copy, and arm the watch in one call),
a **watch-guard** that blocks a silent disarm and drives the give-up counter, and the
**audible alert** of rule 9. These are ergonomics over the bare loop above — a
conforming implementation needs only the inbox + the background wait.

---

## 5. Decision capture

When a fork is resolved during a session, write a **decision record** so the choice
is durable and auditable. The minimum a record carries:

| Field | Meaning |
|---|---|
| `id` | A stable identifier. |
| `status` | `decided` (resolved) or `proposed` (still open for the principal). |
| `ruling.by` | `principal` (they ruled it) / `agent-delegated` (resolved under standing delegation, awaiting their nod) / null. |
| `context` | The fork, the options, and the reasoning. |
| `verbatim` | If the principal ruled it, quote their words. |

- **SPEC-BUILD** and any deliberation mode **must** emit ≥1 record (that is their
  output). Review modes emit one only if a real fork was decided.
- For decisions cleared by a peer's second voice under CONSULT, a lightweight
  *delegated* record (`status: decided · ruling.by: agent-delegated`) is collected in
  a per-run ledger for the principal's later batch ratify/override.

The schema, store location, and any validation are an **implementation choice** — the
protocol requires only that resolved forks become durable records.

---

## 6. Authoring a mode (the six-field template)

Every mode is specified as exactly these six fields, so adding one is fill-in:

```
MODE <NAME>
  goal:        one line — what "done" means
  roles:       Driver/Challenger | Peers (+ who-is-who rule)
  turns:       the numbered choreography (each turn = one agent's action + the
               peer message that ends it)
  termination: the mode's convergence test (specializes frame rule 5)
  output:      the artifact(s) produced (+ whether a decision record is mandatory)
  reuse:       which existing skills/tools the turns lean on
```

**Finding-ID grammar (review modes).** Original findings against the artifact are
`F1, F2, …`; an edge that emerges *from a fix* (not present in the original) gets the
`E#` series (`E1, E2, …`) so its provenance is legible. The review's **change log is a
required output** — a table of `finding · rank · disposition · verification`, kept in
or beside the artifact.

---

## 7. Deliberate non-goals (this package)

- **More than two participants.** The frame is built for a sovereign *pair*. N-way
  co-work is a different protocol.
- **Sub-agent dispatch.** Out of scope by definition (see the intro).
- **A bundled inbox or wait implementation.** Those are the separate
  [`inbox`](https://github.com/localnexus/inbox) and
  [`await-reply`](https://github.com/localnexus/await-reply) packages; this package is
  the *coordination frame* layered on top of them.
- **Content-aware reply matching.** Whether a reply actually *answers* the question is
  the re-woken agent's judgment, not the wait predicate's (an `await-reply` non-goal).
