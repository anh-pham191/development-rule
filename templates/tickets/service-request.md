# Service Request Template

Use this template for Service Requests: support requests or incidents from service desk projects.

---

## Template

```markdown
## Request

[Clear statement of what is being requested. What does the requester need?]

## Context

[Background information. Why is this needed? What is the business situation?]

**Requester:** [Name/team]
**Urgency:** [When is this needed by? Why?]

## Details

[Specific information needed to fulfil the request.]

- **Affected system/area:** [e.g., an organisation portal, Driver app]
- **User/account:** [Identifiers if applicable]
- **Examples:** [Specific instances, transaction IDs, etc.]

## Attachments

[Screenshots, documents, or other supporting materials.]

## Proposed Resolution

[If known: what action is expected to resolve this request?]

## Notes

[Any additional context, related tickets, or follow-up requirements.]
```

---

## Guidance

### Service Requests and Development Work

Service requests often:
- Originate from non-technical users
- Require triage to understand what's needed
- May not have all technical details upfront

Service requests may include development work directly. For smaller changes, complete the work within the service request ticket. For larger changes that warrant separate tracking:
1. Create a Bug, Task, or Feature ticket with proper structure
2. Link the development ticket to the service request
3. Resolve the service request when the linked work is complete

### Triage Checklist

Before working on a service request, clarify:
- [ ] What exactly is being requested?
- [ ] Who is affected and how many?
- [ ] What is the urgency and why?
- [ ] Do we have enough information to proceed?

If information is missing, ask clarifying questions in comments before beginning work.

### Request Types

Common service request types:

**Support requests:**
- How do I...?
- I need help with...
- Something isn't working as expected

**Incident reports:**
- System is down/broken
- Users are experiencing...
- Something has stopped working

### Linking to Development Work

When a service request spawns separate development tickets:

```
## Development Work

This request requires code changes. Tracked in:
- [TICKET-XXX]: [Bug/Task/Feature summary]

The service request will be resolved when the above ticket is complete.
```

### Resolution Notes

When closing a service request, document:
- What was done to resolve it
- Any relevant ticket links
- Follow-up actions if applicable
