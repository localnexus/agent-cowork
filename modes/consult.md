# MODE — CONSULT

On-call second opinion / **dual control.** One agent (the **Executor**) runs a
primary skill *solo*; the other (the **Consultant**) supplies a *documented* second
voice **only at gated decision points** — so formality-class decisions clear with
second-voice assurance instead of routing to the principal, and only truly
substantive calls escalate. The discipline is lifted from key-management
separation-of-duties: no single operator authorizes a formality-class step alone; the
principal is reserved for ceremony-grade calls.

> Reads on top of the invariant frame in [`../PROTOCOL.md`](../PROTOCOL.md). This is
> the catalog's first **asymmetric** mode — it specializes frame rules 1 & 2 (see the
> carve-out at the end). The Formality Test it relies on is defined in
> [`../PROTOCOL.md`](../PROTOCOL.md) §3.

---

| Field | Value |
|---|---|
| **goal** | Let the Executor run a primary skill solo while the Consultant supplies a documented second voice only at gated decision points. |
| **roles** | **Executor** — runs the primary skill and **owns the final call on every decision.** **Consultant** — on-call; **advises / assents / escalates; never seizes.** Principal = escalation authority. |
| **opener** | **Executor opens turn 1** (frame rule 8). |
| **termination** | The primary-skill run completes with every decision (a) resolved-by-observation, (b) formality-cleared with a delegated record, or (c) escalated. **No open decision proceeds unsigned.** |
| **output** | The primary skill's deliverable + a **delegated-decision ledger** (one row per formality-cleared decision) for the principal's async batch ratify/override. |

## Turns

0. **Entry — readiness + role-disjointness ACK.** Before any byte is mutated, both
   agents confirm the run is ready (preconditions met *and* design inputs converged)
   over a *wired* channel, and exchange an explicit **role-disjointness ACK**: both
   confirm who-owns-what **and that neither holds pending edits to the shared
   artifact.** *(A role collision — both primed to mutate the same files — is the
   failure this ACK averts: a "watch is up" signal can read healthy while the roles
   are silently crossed.)*
1. **Executor** runs the primary skill solo to its natural decision points.
2. At each decision, apply the **Formality Test** ([`../PROTOCOL.md`](../PROTOCOL.md)
   §3). **SUBSTANTIVE** → escalate to the principal; do not proceed on that fork.
   **FORMALITY** → message the Consultant: `{decision · finding/evidence · my call ·
   sub-shape}`. → message Consultant.
3. **Consultant re-derives on the ground truth / artifact** (never the Executor's
   prose) → **ASSENT / ADVISE / ESCALATE.** It **must pull the cited ratifying
   artifact** and confirm it covers *this* decision — a missing, uncheckable, or
   non-covering citation is an **automatic ESCALATE** (verify the tooth, not that one
   was cited). It may also **contest the sub-shape** ("you marked this ADVISORY; it
   needs ASSENT" → escalate, never a silent downgrade). → message Executor.
4. **Executor finalizes:** ASSENT-gated + **ASSENT** → proceed, emit a delegated
   record. **ADVISORY** → Executor decides + owns it; advice and any dissent recorded
   (non-blocking). **ESCALATE** on either → route to the principal.
5. Loop to run completion; **independent close** — Executor reports the run + the
   delegated-decision ledger; Consultant reports its assents / escalations.

## The two sub-shapes (engine invariant — not host-optional)

- **ASSENT-gated** — a true two-signature formality; the Consultant's assent is
  **required** to proceed; withheld → the principal.
- **ADVISORY** — a reversible either/or; the Executor decides + owns it; dissent
  documented, non-blocking.

The decision's **class** sets the sub-shape — the Executor does **not** self-select.
Genuine class-ambiguity is itself a Formality-Test failure (#4) → **ESCALATE.**
*Rule of thumb:* **ASSENT blocks; ADVISORY proceeds.**

## Anti-rubber-stamp

A Consultant who only ever says "proceed" is worthless. On a genuine edge the default
is **ESCALATE, not assent.** Always **re-derive on the artifact** — never nod to the
Executor's framing, or two layers compound a *shared* error. Track the
**assent : escalate ratio** as a per-run health smell.

## Asymmetric-watch carve-out — specializes frame rules 1 & 2

Rules 1 (always-armed) and 2 (every turn ends with a peer message) assume symmetric
turn-taking; CONSULT's Executor runs solo through many internal turns. The carve-out:

- The **Consultant stays armed on-call for the whole run** (generous wait; re-arm on
  each timeout). **A timeout never means "Executor done"** — only an explicit close
  does.
- The **Executor** at an **ASSENT-gated** point **blocks** on the Consultant (or does
  `on_peer_timeout`-safe work), and proceeds only on ASSENT; at an **ADVISORY** point
  it **proceeds** and folds advice when it lands.
- The Executor's solo internal steps are **not** rule-2 "silent turn" violations — the
  peer-message obligation attaches at gate points and at close, not to every internal
  step.

*Failure averted:* without this carve-out, a solo executor either spams the peer with
non-decisions (rule 2 literalism) or drops the watch between gates (rule 1 violation).

## Reuse

- The **Formality Test** (frame §3) · the decision-record contract (frame §5) · the
  background wait (Consultant's on-call pull is near-free).
- The **host skill supplies the per-run decision taxonomy** — a table mapping that
  skill's decision points to the FORMALITY / SUBSTANTIVE and ASSENT / ADVISORY
  classes. The taxonomy supplies *instances*; it never overrides the sub-shape rule.

## Note — single-reviewer hosts

If CONSULT wraps a single-reviewer audit skill, the Consultant is **not a second blind
reviewer** (that would breach the host's single-reviewer invariant) — it seconds the
Executor's **disposition** of findings, not the attack on the artifact. A *new* issue
tripped over while auditing a disposition does not log a fresh finding — it
**ESCALATEs** (the disposition is unsafe / scope exceeded). The high-value case: a
second signature on a **dismissed high-severity finding** (dismiss-low = formality;
dismiss-high = substantive → the principal).
