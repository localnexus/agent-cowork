# agent-cowork — A Coordination Frame for Two-Agent Peer Co-Work

`agent-cowork` lets **two sovereign agents** — each with its own session and its own
[inbox](https://github.com/localnexus/inbox) — collaborate on a shared artifact as
**peers**, with a human principal in a light oversight role. It is the discipline that
turns "two agents messaging a file back and forth" into a reliable, convergent,
auditable working session.

It is **not** sub-agent dispatch (one agent spawning and commanding workers). It is
two equals — a Driver and a Challenger, or two symmetric Peers — holding each other to
a standard the work has to clear before it ships.

```
            ┌──────────── the invariant frame (PROTOCOL.md) ────────────┐
            │  arm-first · turn ends with a peer message · converge on   │
            │  the bytes · capture decisions · independent close · …     │
            └───────────────────────────────────────────────────────────┘
                      ▲                                   ▲
            ┌─────────┴─────────┐               ┌─────────┴─────────┐
            │   Agent A (Driver)│◄── inbox ────►│ Agent B (Challenger)
            └─────────┬─────────┘   messages    └─────────┬─────────┘
                      │                                   │
                      └───────────► principal ◄───────────┘
                            (surfaced to, not blocking)
```

## What problem it solves

Two agents can already message each other through a file inbox. What they *lack* is a
**frame** that makes the collaboration trustworthy:

| Without a frame | With `agent-cowork` |
|---|---|
| An agent ends a turn and forgets to listen for the reply | **Arm-first:** the wait is the first step of every turn |
| "Looks done to me" ships an unreviewed change | **Converge on the bytes:** both must verify the *actual artifact*, not each other's summary, before `CONCUR` |
| A resolved disagreement evaporates | **Decisions become durable records** |
| One agent unilaterally declares victory | **Two independent closes**, or halt-and-escalate |
| A wait hangs forever on a dark peer | **Graduated give-up:** bounded retries, then escalate or page |

## The shape of it: a fixed frame + a swappable mode

The **frame** never changes — it is the nine standing rules in
[`PROTOCOL.md`](PROTOCOL.md). The **mode** is the head that changes per job: it sets
the turn choreography, the goal, and the output. This package ships three:

| Mode | Use it when | Output |
|---|---|---|
| [**spec-build**](modes/spec-build.md) | Two agents co-author a spec section by section, each section adversarially reviewed before the next | A spec + a decision record per fork |
| [**consult**](modes/consult.md) | One agent runs a task solo; the other is an on-call second voice (dual control) at gated decisions only | The task's deliverable + a delegated-decision ledger |
| [**realtime-coordination**](modes/realtime-coordination.md) | Two agents each work a slice of one roadmap solo, exchanging heartbeats to stay aligned (LOW–MODERATE stakes) | The completed slices + a heartbeat trail |

Adding a mode is fill-in-the-blanks against a six-field template (`PROTOCOL.md` §6).

## Prerequisites

| Requirement | Detail |
|---|---|
| **Two agents, each with an inbox** | Following the [inbox protocol](https://github.com/localnexus/inbox) (`YYYY.MM.DD.HHMM_from-<sender>_<topic>.md`). |
| **A background reply-wait** | The [`await-reply`](https://github.com/localnexus/await-reply) companion — arm a cheap wait on a peer's reply and be re-woken when it lands. |
| **A host that re-invokes an agent on background-task completion** | The one *platform* dependency. Native on some harnesses; emulated on others. See [`docs/portability.md`](docs/portability.md). |
| **A place to write decision records** | Any durable store. The schema is your choice (`PROTOCOL.md` §5). |

> **Not required:** any server, broker, database, or shared runtime. Two inboxes on a
> shared filesystem and a background wait are the whole substrate.

## Quickstart

1. **Fix the invocation** (`PROTOCOL.md` §2): pick the `mode`, name the `artifact`,
   assign `roles`, set the `impact` tier.
2. **The producer opens** (frame rule 8): the drafter / executor / requestor sends
   turn 1; the reviewer awaits. **Arm your watch first** (frame rule 1).
3. **Run the mode's choreography.** Every turn ends with a peer message
   `{what I did · status · what I need next}` and re-arms the watch.
4. **Surface milestones to the principal** without blocking (frame rule 3). Capture
   every resolved fork as a decision record (§5).
5. **Converge on the bytes:** both agents re-verify the artifact and emit
   `CONCUR — no weak edge found`, **or** halt and escalate.
6. **Independent close:** each agent reports to the principal separately.

The operational detail — arming, the give-up policy, the step-away alert — is in
[`docs/usage.md`](docs/usage.md).

## Contents

| Path | What it is |
|---|---|
| [`PROTOCOL.md`](PROTOCOL.md) | Normative spec: vocabulary, invocation, the nine invariant rules, the await loop, decision capture, the mode template. |
| [`modes/`](modes/) | The three mode bodies (spec-build · consult · realtime-coordination). |
| [`docs/usage.md`](docs/usage.md) | Running a session end-to-end + the non-negotiable discipline. |
| [`docs/portability.md`](docs/portability.md) | Adapting across operating systems (macOS / Windows / Linux) and agent harnesses (Claude Code, Copilot CLI, others). |
| [`examples/session-trace.md`](examples/session-trace.md) | A worked spec-build run, message by message. |

## Companions

`agent-cowork` is the top layer of a small stack. Each layer is a separate package so
you can adopt only what you need:

```
  agent-cowork   ── coordination frame + modes        (this package)
       │
  await-reply    ── cheap background wait on a reply   github.com/localnexus/await-reply
       │
  inbox          ── file-based message protocol        github.com/localnexus/inbox
       │
  filesystem     ── two directories you can read/write
```

## Status

The frame and the **spec-build** and **consult** modes are field-proven across
multiple real two-agent runs. **realtime-coordination** is a v0 prototype — sound in
design, tighten on first use. This repository is a **generalized, light proof of
concept** extracted from a working internal system; paths, names, and
environment-specific tooling have been stripped or replaced with generic seams.

## License

See [`LICENSE`](LICENSE) (MIT).
