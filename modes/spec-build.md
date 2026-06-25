# MODE — SPEC-BUILD

Co-author a spec from known goals + the **principal's values**, section by section,
each section adversarially reviewed before the next one is drafted. The output is a
spec document plus a decision record for every real fork.

> Reads on top of the invariant frame in [`../PROTOCOL.md`](../PROTOCOL.md). Only the
> mode-specific choreography is below.

---

| Field | Value |
|---|---|
| **goal** | A spec, co-authored section by section, each section adversarially reviewed and converged before the next is drafted. |
| **roles** | Two sanctioned configs — see below. |
| **opener** | **Drafter opens turn 1; Reviewer awaits** (frame rule 8). |
| **inputs** | `principal_values` is **mandatory**. |
| **termination** | Every section *and* a whole-spec completeness pass reach mutual `CONCUR`, or halt-and-escalate. |
| **output** | A spec document + a decision record per real fork. **≥1 record is mandatory.** |

## Roles — two sanctioned configurations

- **(a) Alternating Peers** *(default)* — symmetric expertise, shared ownership;
  alternate who drafts per section.
- **(b) Fixed Drafter / Reviewer** — one agent drafts every section, the other reviews
  every section. Use when expertise is asymmetric or authorial continuity matters.
  Under (b), the "alternate who drafts" step does not apply.

## Inputs — the principles header

`principal_values` is mandatory: pull the principal's stated values and working style
(from their values brief) **plus** the goals stated in the ask, and restate them as an
explicit **principles header** at the top of the spec. Every section is then checked
against that header — it is the rubric the spec must satisfy.

## Turns

1. **Drafter opens** (frame rule 8): agree the **section list** + the **principles
   header**, and **forecast each section's rough size → pre-allocate the section→file
   map before drafting.** Put the map in the entry-file index from turn 1. *(This
   averts a mid-draft re-split and the rename-induced stale references that follow it.)*
   → message Reviewer.
2. **Drafter** writes section *k*. → message Reviewer.
3. **Reviewer** attacks section *k* unit-by-unit **and against the principles header**
   (does it honor the principal's stated values — reversibility, scope, substantiation,
   whatever the header names?). → message Drafter.
4. **Drafter** revises; on mutual `CONCUR` for section *k*, advance. Capture any real
   fork as a decision record.
5. Repeat 2–4 across all sections; then a **whole-spec completeness pass** (frame rule
   5 applied over the whole document — including cross-file references in both
   directions on a multi-file spec).
6. **Independent close** — each agent reports to the principal separately; the spec +
   its decision records are the deliverable.

## Reuse

- A **test-design** lens to turn each section's acceptance criteria into a checkable
  test plan.
- A **planning** skill if the spec graduates to a build.
- The decision-record contract (frame §5).

## Notes

- Keep each spec file modest (a soft target around 10 KB, hard around 15 KB); split
  before a file sprawls, and record the split in the entry index so references stay
  live.
- A spec that no section's review ever pushed back on is a smell: either the principles
  header is toothless or the Reviewer is rubber-stamping. Real SPEC-BUILD runs surface
  findings.
