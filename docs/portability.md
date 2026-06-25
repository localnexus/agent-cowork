# Portability — Operating Systems & Agent Harnesses

`agent-cowork` is mostly **platform-neutral**: the nine invariant rules, the modes,
and the decision-capture discipline are pure convention and care about nothing below
them. Almost all portability concern collapses onto **one seam** and a handful of
**shell/filesystem details** inherited from the companion packages.

This document draws that boundary, then gives concrete adaptation notes for the three
desktop operating systems and for different agent harnesses (Claude Code, GitHub
Copilot CLI, and others).

---

## 1. The one seam that actually varies: the wake

The frame needs exactly one capability from the host platform:

> **Run a background wait on a peer's reply, and bring the agent back when that wait
> finishes** (a reply landed, or it timed out).

Everything else in the frame is platform-neutral. So porting is almost entirely about
**how this one wake is wired** — and there are only two archetypes:

| Archetype | How the wake works | Harness shape |
|---|---|---|
| **A. Native re-invoke** | Launch the wait as a background task; the harness **automatically re-invokes the agent** when the task exits. *Task completion IS the wake.* | Claude Code (`run_in_background: true`) |
| **B. Explicit block-wait** | Launch the wait as a background task, then **explicitly block on it** with a "wait for this task" tool call; the result comes back as normal tool output. | Most other agent/tool-runner harnesses |

The frame is **identical** in both; only the *arming + collecting* differs. A real
internal port of this system to a second harness changed nothing but those two tool
calls — the wait script, the inbox, the rules, and every mode were untouched.

```
        ┌─────────────────────────────────────────────┐
        │  agent-cowork: frame + modes + decisions     │  ← 100% portable
        ├─────────────────────────────────────────────┤
        │  await-reply: wait predicate + timeout       │  ← portable script (see §3 for OS)
        ├─────────────────────────────────────────────┤
        │  THE WAKE  (archetype A or B)                │  ← the only real per-harness work
        └─────────────────────────────────────────────┘
```

---

## 2. Harness adaptations

### Claude Code — archetype A (native)

Native fit. Launch the wait via the Bash tool with `run_in_background: true`; when the
background task exits, the agent is re-invoked once and reads the result. No timer, no
explicit block — task completion is the wake. This is the lineage the frame was first
dogfooded on, so the "arm-first / re-arm-on-fire" rhythm maps directly onto
backgrounded Bash calls at near-zero idle cost.

### GitHub Copilot CLI — archetype B (verify, then block-wait)

Copilot CLI is an agentic terminal tool that can run shell commands, so the
`await-reply` script and the inbox run on it unchanged. The adaptation is the **wake**:

- **Confirm the background model.** Determine whether the version in use can (a) run a
  command detached and later (b) block on its completion and read its output. If it
  can, you are in **archetype B**: launch the wait in the background, then issue the
  block-wait call; process the result on return.
- **If it has no background+wait primitive at all**, fall back to **synchronous**
  arming (§4): run `await-reply` in the foreground. The agent blocks in-process until
  the wait returns. You keep the wait predicate, the timeout, and the exit codes; you
  lose only the "idle at zero model-cost" property.
- **Permissions / approvals.** Harnesses that prompt before running shell commands
  will prompt on each arm — pre-approve the wait command (and the send) so a session
  is not gated on a human clicking "allow" every turn.
- **Don't assume the wrapper tools exist.** The optional send+arm wrapper, watch-guard,
  and audible-alert in the reference system are conveniences, not requirements. On a
  fresh harness, start from the bare loop in [`../PROTOCOL.md`](../PROTOCOL.md) §4 and
  add ergonomics later.

> The exact flag names and the background/blocking API differ by harness and version,
> and they change — **verify the two calls on your harness** rather than copying a
> snippet. The seam to satisfy is fixed (§1); the call names are not.

### Any other harness — the porting recipe

1. Keep the frame, the modes, and the `await-reply` script **as-is**.
2. Identify the harness's background-job + completion-callback (or wait) mechanism.
3. Wire **arming** (archetype A: background-launch and let it re-invoke; archetype B:
   background-launch then block-wait).
4. If there is **no** background-wake at all → run synchronously (§4).

The wait predicate, the timeout, and the exit-code contract never change. Only the
wake does.

---

## 3. Operating-system adaptations

The frame is OS-agnostic; the surfaces that touch the OS are the **inbox** (files +
filenames) and the **wait** (`stat`/`date`/`sleep`). The notes below are where
macOS, Linux, and Windows diverge.

| Concern | macOS | Linux | Windows |
|---|---|---|---|
| **File mtime** (the wait's freshness test) | `stat -f %m` (BSD) | `stat -c %Y` (GNU) | No POSIX `stat`; use PowerShell `(Get-Item x).LastWriteTimeUtc` or a portable runtime |
| **Shell** | `zsh`/`bash` | `bash`/`sh` | PowerShell or WSL/Git-Bash (the reference `bash` runs as-is under WSL/Git-Bash) |
| **`date` / `sleep`** | BSD variants | GNU variants | PowerShell `Start-Sleep`, `Get-Date` |
| **Path separator** | `/` | `/` | `\` (or `/` in most runtimes) — never hard-code `/` in portable code |
| **Filename case** | case-**insensitive** by default | case-**sensitive** | case-**insensitive** |
| **Reserved filename chars** | permissive | permissive | `: \ / * ? " < > \|` forbidden; reserved names (`CON`, `NUL`, …) |
| **Path length** | long | long | classic 260-char `MAX_PATH` limit (opt-in long paths) |

**The practical guidance:**

- **`stat` portability is the #1 item.** The reference `await-reply` auto-detects
  BSD-vs-GNU `stat`. On Windows, either run it under **WSL or Git-Bash** (recommended —
  the script is unmodified) or reimplement the predicate against the spec using a
  runtime with a portable mtime call (any language conforms — the predicate is "a
  top-level file matching `*_from-<sender>_*.md` with mtime newer than arm-time").
- **Sender-name case.** Because macOS/Windows are case-insensitive and Linux is not, a
  participant named `Alice` vs `alice` can collide on one OS and not another. **Normalize
  participant ids to a single case** and keep them distinct beyond case.
- **The timestamp convention dodges Windows's worst trap on purpose.** The inbox
  filename format uses **dots**, `YYYY.MM.DD.HHMM` — *not* `HH:MM` — so it is already
  legal on Windows (where `:` is forbidden in filenames). Preserve that; never
  introduce a colon into a message filename.
- **One clock, applied consistently.** Filename ordering across senders only holds if
  every participant stamps with the **same timezone**. Agree on one (the principal's
  local zone is the usual choice) and apply it everywhere.
- **Atomic delivery.** "Send" should be write-elsewhere-then-move/copy-into-inbox so a
  reader never sees a half-written message. `rename` within a filesystem is atomic on
  all three OSes; a cross-filesystem move is not — keep the staging file on the **same
  volume** as the inbox.
- **Mixed-OS sessions work** as long as the above hold: the wire is just well-formed
  files on a shared filesystem, so one agent on macOS and one on Linux co-work fine.

---

## 4. The universal fallback: run it synchronously

If a platform offers **no** background-wake at all (no archetype A, no archetype B),
the frame still runs. Arm `await-reply` in the **foreground**: the agent blocks
in-process until a reply lands or the timeout fires, then continues on the same
exit-code contract.

You keep:

- the wait predicate (a new message from the peer),
- the bounded timeout and give-up policy,
- the exit codes and every frame rule.

You lose only the **idle-at-zero-cost** property — the session occupies a slot while
it waits instead of sleeping for free. For many uses that is an acceptable trade, and
it makes the frame portable to essentially anything that can run a shell command and
read a directory.

---

## 5. Summary — what to actually do when porting

1. **Wire the wake** (§1–2): pick archetype A or B for your harness, or fall back to
   synchronous (§4). *This is ~all of the harness work.*
2. **Make the wait's `stat` portable** (§3): WSL/Git-Bash on Windows, or reimplement
   the predicate. *This is ~all of the OS work.*
3. **Normalize participant-id case and the clock/timezone** (§3).
4. Leave the frame, the modes, and the decision discipline **untouched** — they do not
   know or care what is under them.
