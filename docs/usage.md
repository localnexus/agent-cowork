# Using agent-cowork

How to run a session end-to-end, and the discipline that keeps it trustworthy. Reads
on top of [`../PROTOCOL.md`](../PROTOCOL.md).

## 1. Set up the run

Fix the invocation inputs before turn 1 (`PROTOCOL.md` §2):

```
mode      = spec-build | consult | realtime-coordination
artifact  = <path or stated claim/problem>
roles     = Driver/Challenger | Drafter/Reviewer | Executor/Consultant | Peers
impact    = LOW | MODERATE | HIGH
max_rounds, on_peer_timeout, principal_values (as the mode requires)
```

Decide **who opens** from the mode (frame rule 8): the drafter / executor / requestor
sends turn 1; the reviewer / consultant / responder awaits.

## 2. The per-turn loop

Every turn, in this order:

```
1. ARM the watch on your peer (background)        ← FIRST, before drafting anything
2. do your work (draft / review / run a step)
3. send the peer message: {what I did · status · what I need next}
4. surface the milestone to the principal (non-blocking)
```

On a reply landing:

```
read it → file it to processed/ → RE-ARM → run your next turn
```

The arm-first ordering is the single most important habit. The natural pull is to
draft your reply and arm "after" — that trailing slot is exactly where the watch gets
dropped and the session silently stalls.

## 3. On re-wake — act by the wait's exit code

The background wait ([`await-reply`](https://github.com/localnexus/await-reply))
returns a code; branch on it:

| Exit | Meaning | Do |
|---|---|---|
| `0` | A new message from the peer landed | Read it, file it, re-arm, take your turn. |
| `3` | Timeout — no reply within the cap | Apply the give-up policy below. |
| `2` | Error (bad args, missing inbox) | Fix and re-arm. **Do not** silently proceed. |

## 4. The give-up policy (graduated by impact)

A timeout is **never**, by itself, "the peer is done." Count consecutive empty
timeouts and act by tier:

| Impact | On *N* consecutive empty timeouts |
|---|---|
| **LOW / MODERATE** | The peer is demonstrably dark → **escalate to the principal and stand by.** Do **not** re-arm. *(You may also stand down early at your discretion once the coordination function is genuinely discharged — e.g. mutual `CONCUR`.)* |
| **HIGH** | **Page the principal and HOLD the watch armed.** No discretionary close. Only the principal's explicit release stands a HIGH run down. |

Pick a moderate per-arm cap (≈20 min) — long enough not to thrash, short enough that a
dark peer is noticed quickly. The give-up counter bounds the *treadmill*; the per-arm
cap does not.

## 5. Convergence — verify the artifact, not the summary

The session ends only when **both** agents emit `CONCUR — no weak edge found` against
the **same** artifact state.

- Before granting `CONCUR`, the reviewer **re-reads the actual artifact bytes** for
  the changes the peer claims. The peer's change-log is a *hypothesis*, not the
  verdict.
- **A fix can introduce a new edge.** If re-reading surfaces one, name it (`E1, E2,
  …`) and iterate — do not wave it through because "we were almost done."
- On a **multi-file** artifact, check cross-file references in **both** directions
  (entry→sections and sections→entry) and grep the whole surface for any stale path
  after a rename or split.
- If `max_rounds` is hit without convergence → **halt and escalate** with the open
  deltas. Never declare convergence unilaterally.

## 6. Discipline (non-negotiable)

- **Arm before you draft.** Always. The watch is the first step of a turn.
- **No silent turns.** Every action-sequence ends with a peer message (the one mode
  carve-out, CONSULT's solo executor, is explicit in its file).
- **Bounded, never unbounded.** Every wait has a finite cap and a give-up valve. A
  wait is a *wait*, not a hang.
- **Cancel when mooted.** If the awaited work becomes irrelevant, stop the background
  wait — never leave an orphan poller running.
- **Capture forks immediately.** A decision resolved mid-run becomes a record *now*,
  while the context is hot — not "later."
- **Two closes, never one merged report.** Two perspectives in, two reports out.
- **Self-harden.** Tag harness friction `[COWORK-FINDING]` inline and fold it back at
  close.

## 7. Stepping away (frame rule 9)

When the principal says they are stepping away, the **non-executing on-call peer**
(Challenger / Reviewer / Consultant) owns an audible alert:

- **arm** it when the principal steps away;
- **ring** at (1) any point that needs the principal's input and (2) once at the final
  step / on completion;
- **disarm** only *after* the ring (ring-before-disarm — the completion chime must be
  allowed to sound).

It is best-effort earshot, not guaranteed delivery — so still surface blocking items
in-thread.
