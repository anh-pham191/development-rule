# Create Ticket

Create a ticket in the project of your choice (default) or another project the user names explicitly. Runs a draft-review-create loop so tickets start life in a shape that can pass `/preflight-ticket` cleanly.

## Context

Ticket quality at creation time determines how much back-and-forth happens later. A ticket created to the standards in [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) and using the correct template from [`templates/tickets/`](../../templates/tickets/) arrives at `/preflight-ticket` needing minimal refinement; one created off-the-cuff often doesn't make it past the Definition of Ready on first pass.

This command wraps the [`ticket-management`](../skills/ticket-management/SKILL.md) skill with an explicit draft-then-approve workflow and a duplicate-search step.

Consumer projects itself uses the project key of your choice. Consumer projects that adopt this command replace the project key and their own; the structure is the same.

## Process

### 1. Clarify the ticket type

which ticket type and template applies. Templates available at [`templates/tickets/`](../../templates/tickets/):

- `bug.md` — something is broken and shouldn't be.
- `feature.md` — net-new capability.
- `task.md` — self-contained unit of work that isn't a feature or bug.
- `epic.md` — container for related tickets.
- `service-request.md` — request to another team or system.
- `subtask.md` — breakdown of a larger ticket.

If the type is unclear from context, ask the user. Do not guess.

### 2. Search for duplicates

Before drafting, search your ticket system for related or duplicate tickets using the ticket-system search tool. A reasonable search:

- Keyword match on the main concepts in the proposed summary.
- Filter to the relevant project (default `YOUR-PROJECT-KEY`).
- Exclude *Done* / *Closed* unless the user wants historical context.

If duplicates or closely related tickets surface, surface them to the user before drafting. Possible outcomes:

- **Create anyway** — genuinely distinct ticket; surface relationship and link it (`Relates to`).
- **Don't create** — an existing ticket covers this; point the user at it.
- **Expand existing** — an existing ticket should absorb this work; propose an edit rather than a new ticket.

### 3. Draft the ticket

Use the matching template from [`templates/tickets/<type>.md`](../../templates/tickets/). Populate:

- **Summary** — starts with an action verb; specific; scannable.
- **Description** — purpose, scope, out-of-scope. A reader who didn't have the originating conversation should understand what is being asked and why.
- **Acceptance criteria** — specific, testable, complete. Each verifiable as pass/fail.
- **Risk fields** (the custom fields captured in [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md)):
  - Risk Description
  - Likelihood
  - Impact
  - Risk Level
  - Mitigation Plan
  - Residual Risk
  - Risk Status
- **Labels** — apply sparingly; match existing label conventions (check recent tickets if unsure).
- **Links** — `Relates to`, `Blocks`, `Blocked by` as appropriate.

Do **not** auto-assign risk fields by guesswork. If the user hasn't provided them, propose a draft and ask for confirmation.

### 4. Present the draft

Show the complete ticket (summary, description, AC, risk fields, labels, links) to the user. Offer to revise. Do **not** create until the user approves.

### 5. Create on approval

Use the ticket-system create-issue tool:


- `projectKey` — `YOUR-PROJECT-KEY` (or as directed).
- `issueTypeName` — from the type decided in step 1.
- `summary`, `description`, AC (usually inside description or in a dedicated field depending on the project's config).
- Risk and label fields via `additional_fields`.
- Custom field IDs vary by project; use the `ticket-management` skill's conventions for this repository.

### 6. Confirm and hand over

After creation:

- Confirm the ticket key (e.g. `TICKET-105`).
- Confirm the status — usually *To Do*.
- Recommend the next command:
  - Usually `/preflight-ticket TICKET-XX` to validate Definition of Ready before implementation.
  - For epics or tickets that aren't intended for immediate work, just hand back the key.

## Rules

- Do NOT create the ticket without explicit user approval of the draft.
- Do NOT guess risk fields; propose and confirm.
- Do NOT skip the duplicate search.
- Do NOT apply labels outside the project's existing convention without flagging.
- DO use the matching template from `templates/tickets/`.
- DO show the complete draft before creation.
- All written outputs use NZ English.

## See also

- [`agents/skills/ticket-management/SKILL.md`](../skills/ticket-management/SKILL.md) — the ticket-skill conventions this command wraps.
- [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) — full ticket-structure, AC format, and risk-field definitions.
- [`templates/tickets/`](../../templates/tickets/) — the template set per ticket type.
- [`agents/commands/preflight-ticket.md`](./preflight-ticket.md) — the usual next command.

