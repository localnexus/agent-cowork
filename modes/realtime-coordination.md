# MODE — REALTIME-COORDINATION

Sustain loose coordination while two agents each work an assigned slice of a **shared
roadmap** *solo*. Periodic status exchange keeps both current and surfaces divergence
early. This is the cheap, **sustained-watch** counterpart to a one-time alignment
handshake.

> Reads on top of the invariant frame in [`../PROTOCOL.md`](../PROTOCOL.md).
> **Scope: LOW-to-MODERATE impact only.** This mode does **no mutual
> work-verification** — that is a review mode's job. It keeps two solo workstreams
> *aligned*, it does not certify each other's output.

---

| Field | Value |
|---|---|
| **goal** | Keep two agents working separate slices of one roadmap in sync, surfacing divergence early, without either verifying the other's work. |
| **roles** | Symmetric **Co-workers** — each owns and drives its assigned slice. (An asymmetric lead/support binding is allowed when frame rule 8 applies.) |
| **opener** | The run opens with the **alignment handshake** below; the requestor sends turn 1. |
| **termination** | Both slices reported complete with a final reconciliation showing no open divergence; **or** a divergence/blocker exceeding LOW-MODERATE → halt → re-align or escalate; **or** the peer goes dark → the give-up valve (escalate + stand by). |
| **output** | The completed roadmap slices + the heartbeat status trail (no standalone document). A decision record iff the opening handshake or a divergence produced a fork. |

## Opening handshake (self-contained)

Before either agent starts, reach mutual `CONCUR` on **five items**. The requestor
opens turn 1 with a proposal covering all five; the responder accepts or counters;
iterate to mutual `CONCUR`. Capture any unresolved fork as a decision record.

| # | Item | What to agree |
|---|---|---|
| 1 | **Ownership** | Who owns which slice. The single most important item — overlapping ownership is the main failure this mode prevents. |
| 2 | **Order** | Sequencing and any cross-slice dependencies (slice B needs slice A's interface first, etc.). |
| 3 | **Interfaces** | The contract at each slice boundary — the shape each side will hand the other. |
| 4 | **Cadence** | How often heartbeats fire (per milestone / per interval) and what a heartbeat must contain. |
| 5 | **Impact ceiling** | The agreed tier (LOW or MODERATE). Crossing it forces a halt (see the divergence seam). |

*(In the full catalog this handshake is its own `alignment-check` mode; the five-item
core is inlined here so this mode is self-contained.)*

## Turns

1. **Open with the handshake** (above): reach mutual `CONCUR` on the five items —
   especially **ownership** and **order**. → handshake messages.
2. **Both work their slice solo**, each with the peer-watch armed (frame rule 1).
3. **Status checkpoint:** at each milestone / blocker / interval, send the peer a
   heartbeat — `{done · now · next · divergence-flag?}` — then **re-arm.** This is
   *not* a blocking request.
4. **On a received heartbeat:** read → file to `processed/` → **re-arm** → integrate
   (adjust your slice for a flagged dependency). **Do not verify the peer's work** —
   out of scope for this mode.
5. **Divergence seam:** if a heartbeat shows the slices have drifted out of alignment,
   **or** the impact would exceed MODERATE → **pause and re-run the handshake** (or
   escalate to the principal). This is the **only** route back up to a verification
   layer — REALTIME-COORDINATION never absorbs that silently.
6. Repeat 2–5 until both slices are complete.
7. **Independent close** — each reports its completed slice + the heartbeat trail
   (frame rule 6).

## Heartbeat carve-out — specializes frame rule 2

Frame rule 2 (every turn ends with a peer message, strict alternation) assumes
turn-taking. After the handshake, status updates are **asynchronous heartbeats fired
at each agent's own checkpoints**, not alternating request-reply turns. **Both watches
stay armed continuously** (rule 1 unchanged — never a silent drop).

*Failure averted:* without this carve-out, each agent blocks awaiting an alternating
reply the peer has **no turn-obligation to send** — deadlocking two solo workstreams.

## Reuse

- The **alignment handshake** above (mandatory opener — this mode composes it).
- The invariant frame (`../PROTOCOL.md` §3) · the background wait · the audible alert
  of rule 9 · the decision-record contract for any fork.

## Note

This mode deliberately trades certainty for cheapness: it is for parallelizing work
that is *already* understood and partitioned, not for jointly figuring out hard or
high-stakes problems. The moment the stakes rise above MODERATE, or the slices start
to interfere, the divergence seam (turn 5) kicks the work back up to a verification
mode. Respect that ceiling.
