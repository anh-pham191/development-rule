# Documenting the Outcome

Once the discussion has pinned down the requirements, write them where the project keeps its
requirements, in the project's own style. The documentation is the deliverable — a great
discussion that lands in the wrong place, or in a foreign format, has low value.

## General principles (any project)

1. **Discover the convention before writing.** Look for an existing `docs/`, requirements folder,
   ADR folder, or templates. Match what's there — location, file naming, header style, frontmatter
   (or absence of it). Don't impose a new structure on a project that has one.
2. **One document per coherent topic**, named in the project's scheme (commonly `kebab-case.md`).
3. **Update the index.** If there's a `README.md` or index listing requirement docs, add the new
   doc to it in the same turn — an unlisted doc is an undiscoverable doc.
4. **Link, don't duplicate.** If a cross-cutting rule already lives in another doc (a "meta" doc,
   a non-functional doc, an ADR), link to it rather than restating it. Duplication rots.
5. **Keep the open-questions register in the doc** until every item is resolved. Don't ship a
   requirements doc with silent gaps; ship it with the gaps named.
6. **Match the project's prose conventions** — spelling (e.g. NZ/UK vs US English), terminology
   from the glossary, and tone.
7. **Decisions become ADRs.** A requirement says *what must be true*; an ADR records *why a
   reversible-at-cost choice was made*. When the discussion produced the latter, write both and
   cross-link.

## File placement decision

```
Is there a per-service / per-module requirements folder?
  ├── Yes → write to <module>/docs/requirements/<topic>.md  (and update its index)
  └── No  → is there a top-level docs/ folder?
             ├── Yes → docs/requirements/<topic>.md (create the folder if the project clearly wants it)
             └── No  → propose a location to the user before creating anything
```

When unsure, propose the path and header to the user and get a nod before writing — cheaper than
relocating later.

## Worked example: a convention-driven service repo

Many mature repos converge on the same shape. Treat the following as a *pattern to recognise and
match* — adapt every name to whatever the target repo actually uses.

**Where requirements live:** a per-module requirements folder, e.g.
`services/<module>/docs/requirements/<topic>.md`

- `kebab-case.md`, no dates or numbers in the filename (`<feature>.md`, `change-management.md`).
- Requirement kinds are often pre-segmented into separate files — reuse the existing split rather
  than inventing your own:
  - `meta.md` — cross-cutting requirements (e.g. temporal correctness, provenance, monetary handling).
  - `non-functional.md` — performance, availability, security, observability.
  - `change-management.md` — how rule/policy changes are absorbed over time.
  - `<feature>.md` — one functional doc per domain concept.

**Header style:** copy whatever the existing docs use. A common form is a title plus a short
context blockquote:

```markdown
# <Module Name> — <Topic> Requirements

> <Programme/project name> — <topic> requirements for the <Module Name>.
> Ticket: <link to the tracking issue, if the project links them>.
> Session decisions: <YYYY-MM-DD>[, <YYYY-MM-DD>, …]  (append each elicitation session's date).
```

**Index:** if the module has a `docs/README.md` (or similar) that indexes requirement docs, add an
entry for the new doc in the same turn.

**ADRs:** if a decision falls out, create an ADR in the project's ADR folder using its naming
scheme (e.g. `<PREFIX>-ADR-NNN-<slug>.md` with a zero-padded number), and update the ADR index.
Match whatever prefix/number convention already exists.

**Prose:** match the project's spelling and terminology conventions, and any "cross-link, don't
duplicate" rule its contributor docs state.

**Commit:** follow the repo's commit convention — typically an issue prefix plus an imperative
subject under ~72 chars, e.g. `<TICKET>: Add <topic> requirements for <module>`. Match its branch
naming too.

**Close-out:** if the project's workflow expects a comment on the tracking ticket after a doc lands,
offer to post it.

**Mirroring note:** some repos require a skill to exist as byte-identical copies in more than one
location (e.g. a canonical `agents/skills/` plus tool-specific mirrors), often enforced by CI. If
the target repo has such a rule, treat mirroring as a deliberate, separate step after authoring.

## Quality checklist before sign-off

- [ ] Each in-scope requirement type from [requirement-types.md](requirement-types.md) is covered or explicitly marked N/A.
- [ ] Boundaries, failure modes, and concurrency behaviour are specified, not just the happy path.
- [ ] Temporal behaviour is addressed (or explicitly N/A) for any rule/rate the system encodes.
- [ ] Acceptance criteria are testable (a test could pass/fail against each).
- [ ] Non-goals are written down.
- [ ] Every assumption is stated; every open question has an owner.
- [ ] Architectural decisions are captured as ADRs and cross-linked.
- [ ] The doc lives in the right folder, named in the house scheme, and the index is updated.
- [ ] Prose matches the project's spelling/terminology conventions.
