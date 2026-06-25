# Worked Example — a SPEC-BUILD session, message by message

A condensed trace of a real-shaped [`spec-build`](../modes/spec-build.md) run between
two agents, `alice` and `bob`, co-authoring a spec for a hypothetical `rate-limiter`
component. It shows the frame ([`../PROTOCOL.md`](../PROTOCOL.md)) in motion: arm-first,
peer messages, convergence on the bytes, a fix-induced edge, decision capture, and an
independent close.

> Names and the artifact are illustrative. Inbox messages follow the
> [inbox](https://github.com/localnexus/inbox) convention
> `YYYY.MM.DD.HHMM_from-<sender>_<topic>.md`.

**Invocation:** `mode=spec-build`, `artifact=spec/rate-limiter.md`,
`roles=Drafter(alice)/Reviewer(bob)`, `impact=MODERATE`, `principal_values` = the
principal's brief (assurance-by-design, reversibility, least surprise).

---

### Turn 1 — Drafter opens (frame rule 8)

`alice` arms a background wait on `bob` **first**, then sends:

```
from-alice → bob   [spec-build · rate-limiter · turn 1]

Proposed section list:  §1 Goals  §2 Algorithm  §3 Config  §4 Failure modes
Principles header (from the principal's brief):
  - assurance-by-design  - reversibility  - least surprise
Section→file map (size forecast): all four fit one file, spec/rate-limiter.md.
What I need next: CONCUR on the section list + header, then I draft §1.
```

`bob` (watch already armed) wakes, files the message, **re-arms**, replies `CONCUR`
on the structure.

### Turns 2–3 — draft §1, review §1

`alice` drafts §2 Algorithm... *(messages each end with `{did · status · next}`;
both re-arm on every fire)*. `bob` attacks §2 against the **principles header**:

```
from-bob → alice   [review · §2 Algorithm]

F1 (HIGH): token-bucket refill uses wall-clock; a clock step backward stalls the
   limiter indefinitely → violates "least surprise". Use a monotonic source.
F2 (LOW):  §2.3 says "drop"; ambiguous whether it 429s or blocks. Name it.
CONCUR withheld pending F1/F2.
```

### Turn 4 — revise, and a fix-induced edge

`alice` fixes F1 (monotonic clock) and F2 (explicit `429`). On re-review, `bob`
**re-reads the actual §2 bytes** (frame rule 5 — not alice's change-log) and catches:

```
from-bob → alice   [re-review · §2]

E1 (MEDIUM): the F1 fix introduced a monotonic source but §3 Config still exposes a
   "clock skew tolerance" knob that no longer has any effect — a stale control.
   (This edge did not exist in the original draft; it rode in on the fix.)
```

`alice` removes the dead knob; both emit `CONCUR — no weak edge found` on §2.

> **This is the point of rule 5.** Had `bob` granted `CONCUR` from the change-log
> ("fixed F1, fixed F2") instead of re-reading the bytes, `E1` ships. A fix can
> introduce a new edge.

### A fork → a decision record

`§3` surfaces a genuine fork: per-key vs global rate limits as the default. Neither is
strictly better; it is a values call. They capture it and surface it to the principal:

```
DECISION rate-limiter-default-scope
  status:    proposed
  context:   default limit scope — per-key (isolation, more state) vs global
             (simpler, noisy-neighbor risk). alice leans per-key; bob leans global.
  ruling.by: null   # awaiting the principal
```

The principal rules **per-key**; the record is updated to `status: decided ·
ruling.by: principal` with their words quoted verbatim.

### Whole-spec completeness pass

After all sections converge, both run the frame-rule-5 pass over the **whole**
document: cross-references checked both directions, full surface grepped for any stale
path. Clean. Both emit the final `CONCUR`.

### Independent close (frame rule 6)

Each reports to the principal **separately** — no merged report:

```
from-alice → principal:  spec/rate-limiter.md converged at v4. 2 findings + 1
   fix-induced edge (E1), 0 dismissed. 1 decision (default-scope, you ruled per-key).

from-bob → principal:  Reviewed all 4 sections on the bytes. Confirm convergence;
   E1 (stale skew knob) was the value-add catch. No open deltas.
```

---

## What the trace demonstrates

| Frame rule | Where it shows |
|---|---|
| **1 — arm first** | Both agents arm *before* drafting and re-arm on every fire. |
| **2 — turn ends with a peer message** | Every turn closes with `{did · status · next}`. |
| **3 — share, act autonomously** | The fork is surfaced to the principal; routine turns are not blocked on them. |
| **4 — capture decisions** | The default-scope fork becomes a durable record. |
| **5 — converge on the bytes** | `bob` re-reads §2 and catches `E1`, the fix-induced edge. |
| **6 — independent close** | Two separate reports, not one merged. |
| **8 — producer opens** | `alice` (Drafter) sends turn 1; `bob` awaits. |
