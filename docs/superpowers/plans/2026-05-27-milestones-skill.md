# Milestones Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Author a tool- and domain-agnostic `milestones` skill that turns a requirements doc into a repo-authoritative `MILESTONES.md` a fresh agent reads to orient itself, with an anti-staleness update/verification protocol as its core.

**Architecture:** A slim `SKILL.md` orchestrator plus three on-demand `references/` files, mirroring the existing `requirements-elicitation` skill. Deliverables are markdown only — no code. Each task authors one file and verifies it structurally; the final task wires it into the repo README and runs whole-skill checks.

**Tech Stack:** Markdown. YAML frontmatter (`name`, `description: >`). Verification via `grep`, `wc`, and link-resolution checks in bash.

---

## Spec reference

Approved spec: `docs/superpowers/specs/2026-05-27-milestones-skill-design.md`. Read it before starting. Non-negotiables drawn from it:

- **Repo-authoritative, tracker-agnostic.** `MILESTONES.md` is the source of truth for status; assume no external tracker; link out only if one exists.
- **Milestone-only granularity.** Tasks are handed off to the existing flow / tickets / `TaskList`. No task-level tracking, no WBS, no estimation/scheduling.
- **Anti-staleness is the core value:** (1) update-in-the-same-change ritual; (2) fresh-agent verify-before-trust.
- **Two operations:** `generate` and `checkpoint`.
- **Full generalisation:** no company, programme, product, ticket-prefix, or repo names. British/Commonwealth spelling (optimise, behaviour, prioritise, organise).
- **House style:** match `requirements-elicitation` — slim `SKILL.md` with `see [file](references/…)` pointers and a `## References` section; references cross-link with bare sibling filenames.

## File structure

```
agents/skills/milestones/
  SKILL.md                       # orchestrator: 2 operations, the one rule (anti-staleness), workflow, pointers
  references/
    milestone-design.md          # what a good milestone is; exit criteria; deriving milestones from requirements
    progress-protocol.md         # the update + verification discipline (the anti-staleness core)
    templates.md                 # MILESTONES.md skeleton + cold-start block + decisions log
```

Also modified:
```
README.md                        # add `milestones` to the per-skill list
```

Reference model to match (read first): `agents/skills/requirements-elicitation/SKILL.md` and its `references/`.

---

### Task 1: SKILL.md orchestrator

**Files:**
- Create: `agents/skills/milestones/SKILL.md`
- Reference (read, do not modify): `agents/skills/requirements-elicitation/SKILL.md`

- [ ] **Step 1: Read the reference skill for house style**

Run: `cat agents/skills/requirements-elicitation/SKILL.md`
Note: frontmatter shape, the "## The one rule" section, the plain-ASCII workflow block, the `## References` annotated list at the end.

- [ ] **Step 2: Write `agents/skills/milestones/SKILL.md`**

Use this exact frontmatter and heading (verbatim):

```markdown
---
name: milestones
description: >
  Turn a requirements doc into a repo-authoritative MILESTONES.md that any fresh agent reads to
  orient itself and continue the work, and keep that file true over time. Use when: (1) A project
  has a requirements doc and needs a phased plan a cold agent can pick up, (2) Onboarding onto an
  unfamiliar repo and asking "what's done, what's next, what's blocked?", (3) Recording that a
  milestone is complete after merging work, (4) Re-orienting at the start of a session before
  continuing a multi-session project. Tracks milestones only — hands task detail to the project's
  ticket flow or in-session task list.
---

# Milestones

Turn requirements into a small set of milestones a fresh agent can act on, captured in a single
committed `MILESTONES.md` that stays true to the repo's actual state.
```

Then author the body with these sections (prose is yours; keep SKILL.md ≈ 90–110 lines, depth in references):

1. **One-paragraph framing** — the artifact orients a cold agent; the skill's real job is keeping status true, not generating a list.
2. **`## The one rule`** — *Status changes only as part of the change that earns it; and never trust `MILESTONES.md` before verifying it against the repo.* One short paragraph. Point to `progress-protocol.md`.
3. **`## Two operations`** — `generate` (requirements → milestones → write the file) and `checkpoint` (update status against reality). 2–3 lines each, each pointing to the relevant reference.
4. **`## Workflow`** — a plain fenced block (NOT graphviz) with ASCII arrows, e.g.:
   ```
   generate:   read requirements → propose milestones (with user) → write MILESTONES.md → commit
   checkpoint: verify file vs repo → update status + evidence → log decisions → commit with the change
   ```
5. **`## Operating rules`** — short bullets: milestones only (hand off tasks); repo is source of truth (link a tracker, don't mirror it); decisions spin out an ADR; keep the cold-start block honest; British spelling.
6. **`## References`** — annotated list pointing to the three reference files (exact pattern below).

Use these exact pointer lines in the References section:

```markdown
## References

- **[milestone-design.md](references/milestone-design.md)** — what makes a good milestone, exit criteria, and how to derive milestones from a requirements doc.
- **[progress-protocol.md](references/progress-protocol.md)** — the update-and-verify discipline that keeps status true (the anti-staleness core).
- **[templates.md](references/templates.md)** — the `MILESTONES.md` skeleton, cold-start block, and decisions log.
```

- [ ] **Step 3: Verify structure and generalisation**

Run:
```bash
test -f agents/skills/milestones/SKILL.md && \
head -1 agents/skills/milestones/SKILL.md | grep -qx -- '---' && \
grep -q '^name: milestones$' agents/skills/milestones/SKILL.md && \
grep -q '## References' agents/skills/milestones/SKILL.md && \
! grep -qi 'digraph' agents/skills/milestones/SKILL.md && \
echo OK-STRUCTURE
```
Expected: `OK-STRUCTURE`

- [ ] **Step 4: Commit**

```bash
git add agents/skills/milestones/SKILL.md
git commit -m "Add milestones skill orchestrator"
```

---

### Task 2: references/milestone-design.md

**Files:**
- Create: `agents/skills/milestones/references/milestone-design.md`

- [ ] **Step 1: Author the file**

Required sections (prose yours; concrete, opinionated, British spelling):

1. **`# Milestone Design`** + one-line intro.
2. **What makes a good milestone** — independently valuable, completable, has a *verifiable* exit, sized so a phase is meaningful (aim 3–8 per project). Contrast with tasks (smaller, volatile, owned elsewhere).
3. **Exit criteria** — must be pass/fail and trace back to acceptance criteria in the requirements doc. Give one worked example of a vague exit rewritten as a verifiable one (domain-neutral — e.g. an auth or import feature, NOT tax/finance).
4. **Deriving milestones from requirements** — read the requirements doc, group by outcome/dependency order, sequence so each milestone unblocks the next; surface ordering risks. Note: prefer outcome-based slices over technical-layer slices.
5. **How many / how big** — guidance and the smell of too-fine (looks like a task list) or too-coarse (one giant "build it" milestone).
6. **Status vocabulary** — `✅ done | 🚧 in progress | ⬜ not started | ⛔ blocked`, and what each means (blocked must name the blocker).
7. **Related** — link `progress-protocol.md` and `templates.md` (bare sibling filenames).

- [ ] **Step 2: Verify**

Run:
```bash
test -f agents/skills/milestones/references/milestone-design.md && \
grep -q '# Milestone Design' agents/skills/milestones/references/milestone-design.md && \
grep -q 'progress-protocol.md' agents/skills/milestones/references/milestone-design.md && \
echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add agents/skills/milestones/references/milestone-design.md
git commit -m "Add milestone-design reference"
```

---

### Task 3: references/progress-protocol.md

**Files:**
- Create: `agents/skills/milestones/references/progress-protocol.md`

- [ ] **Step 1: Author the file**

This is the anti-staleness core — make it the strongest reference. Required sections:

1. **`# Progress Protocol`** + one-line intro: generating milestones is easy; keeping status true is the job.
2. **Update-in-the-same-change** — milestone status changes only in the same commit/PR that achieves the change, never as a separate "remember to update" step. Describe the checkpoint ritual at PR/merge time. Show a concrete before/after of the cold-start block being updated alongside the change.
3. **Verify before trust (cold start)** — before acting on `MILESTONES.md`, run a cheap reality check. Give the actual commands an agent runs:
   ```bash
   git log --oneline "$(grep -m1 'Last verified' MILESTONES.md | grep -oE '[0-9a-f]{7,40}')"..HEAD
   ```
   and a prose checklist: does claimed-done work exist? does "Start here" still make sense? If drifted, correct the file first. Treat the file as a **strong prior, not gospel**.
4. **Keeping "Last verified" honest** — update the SHA/date whenever the file is verified or changed.
5. **What NOT to track here** — task-level churn (handed off), anything a tracker owns (link instead).
6. **Decisions & scope changes** — when a milestone's scope shifts, log it with date + why; if architectural, spin an ADR (link the `adr-documentation` skill by name) and reference it.
7. **Related** — link `milestone-design.md` and `templates.md`.

- [ ] **Step 2: Verify**

Run:
```bash
test -f agents/skills/milestones/references/progress-protocol.md && \
grep -q '# Progress Protocol' agents/skills/milestones/references/progress-protocol.md && \
grep -q 'Last verified' agents/skills/milestones/references/progress-protocol.md && \
grep -q 'git log' agents/skills/milestones/references/progress-protocol.md && \
echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add agents/skills/milestones/references/progress-protocol.md
git commit -m "Add progress-protocol reference (anti-staleness core)"
```

---

### Task 4: references/templates.md

**Files:**
- Create: `agents/skills/milestones/references/templates.md`

- [ ] **Step 1: Author the file**

Open with `# Templates` + one line ("adapt names to the project; drop sections that don't apply, but say why for a major omission"). Then include this **exact** `MILESTONES.md` skeleton in a fenced block (must match the spec verbatim):

```markdown
# <Project> — Milestones

> Last verified: <commit SHA / date>. Authoritative record of milestone status.

## Cold start (read this first)
- Current phase: <milestone N — name>
- Done: <list>   In flight: <list>   Next: <list>   Blocked: <list + why>
- Start here: <the single next action a fresh agent should take>

## Milestones
### M1 — <name>  [✅ done | 🚧 in progress | ⬜ not started | ⛔ blocked]
- Goal: <one line; traces to a requirement>
- Exit criteria: <verifiable; mapped to acceptance criteria in the requirements doc>
- Evidence: <PR/commit links proving exit met>   Tracker: <link if one exists>

## Decisions & scope changes
- <date> — <what changed and why> (link an ADR if the change is architectural)

## Links
- Requirements: <path to requirements doc>   Tracker: <optional>
```

Then add a short **filled example** (a domain-neutral project, e.g. "CSV import feature" or "user auth") showing one done milestone with evidence links and one in-progress milestone — so the implementer sees the intended fidelity. Keep it generic; no company/tax/finance specifics.

- [ ] **Step 2: Verify the skeleton is present and intact**

Run:
```bash
test -f agents/skills/milestones/references/templates.md && \
grep -q '## Cold start (read this first)' agents/skills/milestones/references/templates.md && \
grep -q 'Last verified' agents/skills/milestones/references/templates.md && \
grep -q '## Decisions & scope changes' agents/skills/milestones/references/templates.md && \
echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add agents/skills/milestones/references/templates.md
git commit -m "Add MILESTONES.md templates reference"
```

---

### Task 5: Wire into README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Inspect the current skill listing**

Run: `grep -n 'agents/skills/' README.md`
Note the line listing task-specific skills (it enumerates `agent-safety`, `code-quality`, `skill-creator`, `adr-documentation`, and per-language skills).

- [ ] **Step 2: Add `milestones` to that list**

Edit the `agents/skills/` table row to include the two authoring/process skills. The current text contains:

```
`adr-documentation`, and per-language skills
```

Insert `requirements-elicitation` and `milestones` so the sentence reads (adjust to match the exact surrounding wording in the file):

```
`adr-documentation`, `requirements-elicitation`, `milestones`, and per-language skills
```

(If `requirements-elicitation` is already listed from its own merge, only add `milestones`.)

- [ ] **Step 3: Verify**

Run: `grep -q 'milestones' README.md && echo OK`
Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "List milestones skill in README"
```

---

### Task 6: Whole-skill verification

**Files:** none (verification only)

- [ ] **Step 1: Generalisation scan (no leaks)**

Run:
```bash
grep -rniE 'grey st|TT-|tax ?traders|taxtraders|jira|atlassian|NZ english|compliance-service|CS-ADR|\.current-ticket|src/platform|development_rule|\btax\b' agents/skills/milestones/ && echo "LEAK FOUND" || echo "CLEAN"
```
Expected: `CLEAN`
If `LEAK FOUND`: open the offending file, replace the domain-specific term with a neutral equivalent, re-run, then commit the fix.

- [ ] **Step 2: All SKILL.md reference pointers resolve**

Run:
```bash
cd agents/skills/milestones && \
for f in $(grep -oE 'references/[a-z-]+\.md' SKILL.md | sort -u); do test -f "$f" && echo "OK $f" || echo "MISSING $f"; done; cd - >/dev/null
```
Expected: three `OK references/…md` lines, no `MISSING`.

- [ ] **Step 3: British spelling check (no US drift)**

Run:
```bash
grep -rniE '\b(optimize|behavior|organize|prioritize|analyze|color|favor)\b' agents/skills/milestones/ && echo "US-SPELLING" || echo "OK-SPELLING"
```
Expected: `OK-SPELLING` (fix any hit to the -ise/-our form, then re-run and commit).

- [ ] **Step 4: SKILL.md stayed slim**

Run: `wc -l agents/skills/milestones/SKILL.md`
Expected: roughly 90–120 lines. If much larger, move depth into a reference file.

- [ ] **Step 5: Final commit if any fixes were made**

```bash
git add -A agents/skills/milestones/
git commit -m "Fix milestones skill verification findings" || echo "nothing to fix"
```

---

## Self-review (completed by plan author)

**Spec coverage:**
- Repo-authoritative / tracker-agnostic → Task 1 (operating rules) + Task 3 (link-don't-mirror) + Task 4 (template `Tracker: <optional>`). ✓
- Milestone-only granularity → Task 1 + Task 2 (contrast with tasks). ✓
- Anti-staleness (update-in-change + verify-before-trust) → Task 3 (whole reference). ✓
- Two operations (generate/checkpoint) → Task 1 (`## Two operations`, `## Workflow`). ✓
- `MILESTONES.md` artifact incl. cold-start block → Task 4 (verbatim skeleton). ✓
- Slim SKILL.md + references house style → Task 1 + Task 6 Step 4. ✓
- Generalisation + British spelling → Task 6 Steps 1 & 3. ✓
- README wiring → Task 5. ✓

**Placeholder scan:** Exact-match artifacts (frontmatter, `MILESTONES.md` skeleton, all verification commands) are given verbatim. Prose references are specified by required-section lists because the prose is the implementer's craft, not a placeholder — each section states its content and the spec constrains it. No "TBD/handle edge cases" left.

**Type/name consistency:** Filenames (`milestone-design.md`, `progress-protocol.md`, `templates.md`), the artifact name (`MILESTONES.md`), the status vocabulary (`✅/🚧/⬜/⛔`), and the `Last verified` field name are used identically across Tasks 1–6.
