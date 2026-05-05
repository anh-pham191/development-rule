# Technology Selection Philosophy

Choosing the right technology for a given problem is one of the most consequential decisions a development team makes. The wrong choice creates years of friction; the right choice becomes invisible — it simply works, and the team focuses on the problem rather than fighting the tool.

We are a team that is genuinely curious about technology. We find energy in learning, satisfaction in mastery, and motivation in staying at the leading edge of our industry. This document honours that spirit while providing the principles and frameworks that turn enthusiasm into good decisions.

This document codifies our principles for selecting languages, frameworks, databases, infrastructure, and tooling. It draws on our lived experience adopting technologies across multiple stacks, and reflects the values that guide how we work.

## Audience

- **Primary:** Developers evaluating, proposing, or deciding on technology choices
- **Secondary:** AI agents making or advising on technology decisions

## Scope

This document covers:

- Principles that guide technology selection decisions
- Per-technology guidance for our current stack (Go, PHP, JavaScript/TypeScript, Python)
- Philosophical positions on database, infrastructure, and infrastructure-as-code choices
- A decision framework for evaluating new technologies
- How to introduce new technologies and retire old ones

The following are **out of scope**:

- Language-specific coding standards and conventions (see individual philosophy documents)
- Service architecture and decomposition decisions (see [Service-Oriented Architecture Philosophy](service-oriented-architecture.md))
- Developer tooling choices (IDEs, CI/CD platforms, observability tools)
- Detailed framework comparisons or benchmarks

---

## The Central Tension

This document contains principles that deliberately pull in different directions. Some encourage exploration, curiosity, and early adoption. Others encourage consolidation, simplicity, and restraint. **This is intentional.**

The tension between exploration and consolidation is not a problem to be solved. It is a core part of how we operate. We are a team that values courageous innovation *and* thoughtful stewardship. Both forces are real, both are necessary, and neither should win permanently.

Good technology decisions do not come from following one set of principles and ignoring the other. They come from holding both in mind and making a judgment call in context. The principles below are grouped to make the tension visible:

| Direction | Principles |
|-----------|------------|
| **Exploration** | Explore and Invest |
| **Restraint** | Ecosystem Maturity, Long-Term Maintainability, Complexity Budget, Strategic Consistency, Interoperability |
| **Grounding** | Problem Fit, Alignment with Domain |
| **People** | See the Person |
| **Process** | Incremental Adoption, Evaluate Honestly |

When these principles conflict (and they will) the resolution is not in the document. It is in the conversation. Discuss, disagree, decide, commit.

---

## Philosophy

These principles answer: *"What do we value when choosing technology?"*

### 1. Problem Fit

Match technology to what the problem needs. Every technology has a sweet spot: the class of problems it was designed to solve well. Start here: *what does this problem require in terms of performance, scale, deployment, data access, and user experience?*

Problem fit is a grounding force. It anchors decisions in reality when enthusiasm or inertia might otherwise dominate.

- ✓ DO: Evaluate technology against the specific requirements of the problem at hand
- ✓ DO: Consider performance characteristics, deployment model, and data access patterns
- ✓ DO: Match performance requirements to actual measured needs, not hypothetical ones, but take them seriously when they are real
- ✗ DON'T: Skip the problem analysis and jump straight to a technology choice
- ✗ DON'T: Let familiarity substitute for fit — the best-known tool is not always the best-suited tool

*In tension with: Explore and Invest, Strategic Consistency*

### 2. Explore and Invest

We pay attention to where technology is heading, not just where it is today. We aim to be leaders in our space, and that sometimes means learning, upskilling, and sharing knowledge before the wider industry has worked it all out.

Excitement matters. Motivation matters. The energy that comes from working with something new and promising is itself valuable, and the opportunity to develop new skills is a genuine benefit of technology adoption, not just a cost to be managed. We are not afraid to be early adopters when we see clear potential, and we pro-actively look for additions or changes to our technology stack even when maintaining the status quo is the comfortable option.

We back ourselves to learn. We do not shy away from technologies that require the team to stretch. We invest in learning through dedicated time, mentoring, peer review, and knowledge sharing, because we trust that a motivated team can acquire capability quickly.

- ✓ DO: Pay attention to industry direction and emerging trends — being early is sometimes being right
- ✓ DO: Pro-actively seek out new technologies that could improve our products and services
- ✓ DO: Value the motivation and energy that comes from working with new technology
- ✓ DO: Back the team to learn new technologies with appropriate support
- ✓ DO: Invest in dedicated learning time, mentoring, and knowledge sharing as part of adoption
- ✓ DO: Allocate time for exploration even when it does not have an immediate production application
- ✗ DON'T: Wait for the industry to fully validate something before exploring it
- ✗ DON'T: Confuse exploration with adoption — exploring is low-cost; adopting is a commitment
- ✗ DON'T: Avoid new technology because the team doesn't know it yet — that is what learning is for
- ✗ DON'T: Expect learning to happen without investment in time and support

*In tension with: Complexity Budget, Strategic Consistency, Ecosystem Maturity, Long-Term Maintainability*

### 3. See the Person

Technology decisions are made by people. Different individuals have different tolerances to change, different appetites for learning, and different strengths. The best technology decisions account for this.

Everyone contributes to technology decisions and has a voice. We listen to concerns with patience and empathy. Sometimes we agree to disagree, and then we commit to the decision together.

- ✓ DO: Ensure everyone has a voice in technology decisions
- ✓ DO: Consider individual strengths, interests, and readiness
- ✓ DO: Listen to concerns raised by others — including scepticism and nervousness
- ✗ DON'T: Dismiss discomfort as resistance — it's a valuable signal
- ✗ DON'T: Let one voice dominate technology decisions, whether for or against change

### 4. Ecosystem Maturity

Prefer technologies with stable ecosystems, good tooling, and active communities. A language or framework is only as useful as the ecosystem around it: package managers, libraries, IDE support, documentation, and community knowledge.

Mature ecosystems reduce risk. When something goes wrong, someone has likely encountered and documented the solution. When you need a library, one probably exists and is maintained. Ecosystem maturity is an important factor but not a sole determinant. Sometimes the right technology has a younger ecosystem, and we accept that trade-off deliberately.

- ✓ DO: Check for active maintenance, community size, and quality of documentation
- ✓ DO: Evaluate the licensing of dependencies and their compatibility with our objectives
- ✗ DON'T: Adopt technologies with a single maintainer or no clear stewardship
- ✗ DON'T: Reject a technology solely because its ecosystem is young — weigh it against other principles

*In tension with: Explore and Invest*

### 5. Long-Term Maintainability

Consider who will maintain this in five years. The initial build is a small fraction of a technology's lifetime. Most of its life is spent being read, debugged, extended, and maintained, sometimes by people who were not involved in the original decision.

Choose technologies that are likely to be well-supported, well-understood, and well-staffed for the long term. Avoid orphan technologies that leave the team stranded.

- ✓ DO: Consider the technology's trajectory — is it growing, stable, or declining?
- ✓ DO: Think about hiring — can you find developers with this skill?
- ✓ DO: Document known limitations and establish upgrade paths for early implementations
- ✗ DON'T: Adopt single-vendor technologies without considering a realistic migration path away from that vendor
- ✗ DON'T: Ignore the maintenance burden that follows the excitement of initial adoption

*In tension with: Explore and Invest*

### 6. Complexity Budget

Every new technology has a cost. Be intentional about additions to the stack. Each technology adds cognitive load for developers, operational overhead for infrastructure, and coordination cost for the team. This cost is real even when the technology itself is excellent.

Think of the stack as having a complexity budget. Every addition spends from that budget. Some additions are clearly worth the cost; others are not. The question is not "is this technology good?" but "is the benefit worth the complexity it adds to our world?"

- ✓ DO: Articulate what the new technology enables that justifies the cost
- ✓ DO: Consider the total cost: learning, tooling, deployment, monitoring, hiring
- ✗ DON'T: Treat the addition of new technology as free
- ✗ DON'T: Assume "better" technology justifies the transition cost in every case

*In tension with: Explore and Invest. Note: learning itself is a valid return on complexity investment, but name it explicitly rather than using it as an unexamined justification.*

### 7. Strategic Consistency

Prefer consolidation over proliferation. Consistency across the stack reduces context-switching, simplifies onboarding, and enables developers to move between projects. A smaller number of well-understood technologies, used broadly, is generally preferable to many technologies used narrowly.

- ✓ DO: Default to existing stack technologies unless there is a clear reason to diverge
- ✓ DO: Look for opportunities to reduce the number of technologies in active use
- ✗ DON'T: Let silo mentality — "they do it their way, we do it our way" — impede sensible consolidation
- ✗ DON'T: Use different technologies in different teams to solve the same kind of problem without a clear reason

*In tension with: Explore and Invest, Problem Fit, Alignment with Domain*

### 8. Incremental Adoption

Start small, build confidence, expand deliberately. New technology should be introduced through a contained, low-risk proof of concept before it is used in critical systems. This approach respects both the technology (it gets a fair trial) and the team (they build competence before the stakes are high).

Our experience with Go adoption illustrates this well: beginning with a small, well-defined internal microservice, evaluating the result, then gradually expanding usage as the team gained knowledge and confidence. This pattern (prove, learn, expand) applies to any technology introduction.

- ✓ DO: Identify an appropriate, bounded project as a candidate for first use
- ✓ DO: Document challenges, learnings, and patterns during early adoption
- ✓ DO: Use feature flags or isolation to control rollout where appropriate
- ✓ DO: Create space for motivated individuals to explore and champion new technology
- ✗ DON'T: Adopt everything at once — find the balance between wholesale change and artificial limitation
- ✗ DON'T: Skip from proof of concept to production-critical use without building team understanding

### 9. Interoperability

New choices must integrate well with existing systems. Technology does not exist in isolation. A new language, framework, or tool must work alongside what already exists: communicating over APIs, sharing infrastructure, fitting into CI/CD pipelines, and coexisting in developers' workflows.

- ✓ DO: Evaluate how the technology fits with existing deployment, monitoring, and communication patterns
- ✓ DO: Consider whether the technology can produce and consume the APIs and data formats already in use
- ✗ DON'T: Adopt a technology that requires rebuilding surrounding infrastructure to accommodate it
- ✗ DON'T: Ignore the integration cost when evaluating a new tool

*In tension with: Explore and Invest. Sometimes adopting new technology means evolving the surrounding ecosystem too.*

### 10. Alignment with Domain

Some domains have natural technology fits. Data science gravitates towards Python. High-throughput services suit Go. Browser interfaces require JavaScript. Content-driven web applications are well served by PHP frameworks.

Recognise these natural affinities rather than fighting them. Choosing a technology that aligns with the problem domain means better library support, more community knowledge, and developers whose instincts match the work.

- ✓ DO: Let the domain inform the technology choice
- ✓ DO: Consider what the wider industry uses for similar problems
- ✗ DON'T: Force a general-purpose tool into a specialised domain when a better-fit option exists
- ✗ DON'T: Assume domain alignment alone is sufficient — other principles still apply

*In tension with: Strategic Consistency. Domain fit sometimes requires a different tool from the standard stack.*

### 11. Evaluate Honestly

Assess limitations alongside benefits, for both new and existing technologies. Thorough evaluation means honestly acknowledging what a technology does poorly as well as what it does well. It means consulting peers, drawing on experience, seeking external perspectives, and assessing security, performance, and maintainability with rigour.

Enthusiasm is not evaluation. Comfort is not evaluation. Show integrity by doing the work either way.

- ✓ DO: Assess security standards, performance characteristics, and licensing implications
- ✓ DO: Seek peer review of proof-of-concept implementations
- ✓ DO: Consult multiple perspectives, including those who are sceptical
- ✓ DO: Be honest about the limitations of existing tools, not just new ones
- ✗ DON'T: Present only the benefits when proposing a technology
- ✗ DON'T: Dismiss concerns raised by others — listen with patience and empathy

---

## Technology Landscape

This section provides guidance on our current technology stack. Each technology has strengths and appropriate use cases. For detailed coding standards and conventions, refer to the individual philosophy documents.

### Go

**Use when:** High-performance services, data APIs, CLI tools, concurrent workloads, infrastructure tooling, new microservices, data processing

**Our go-to for:** Data-only APIs and back-end services. Go is our default choice for new services that primarily serve data.

**Strengths:** Performance, simplicity, strong typing, excellent concurrency, single binary deployment, versatile hosting equally suited to serverless (Lambda) and containerised (ECS) deployments

**Considerations:** Smaller local talent pool than some alternatives (though growing, particularly among teams with a PHP background). The type-strict, compiled nature provides safety and security benefits but requires developers to adapt from dynamically typed languages. Our experience has shown that PHP developers transition well with appropriate support.

See [Go Development Philosophy](go-development.md) for coding standards.

### PHP

**Use when:** Web applications (especially existing systems), Symfony and SilverStripe projects, content-driven sites

**Strengths:** Mature ecosystem, extensive libraries, deep team expertise, large existing codebase, strong framework support

**Considerations:** Less suited to high-throughput data processing or services where performance and concurrency are primary concerns. Not the best choice for rapid prototyping of new interfaces. For that, we prefer approaches more aligned with rapid agentic AI development (e.g. JavaScript and/or Python/Go). Recognise both the strengths of PHP for its sweet spot and its limitations for workloads that have outgrown it.

See [PHP Development Philosophy](php-development.md) for coding standards.

### JavaScript / TypeScript

**Use when:** Front-end applications, Vue single-page applications, rapid prototyping of new interfaces

**Strengths:** Universal browser support, rich UI frameworks (Vue), npm ecosystem, rapid iteration on product ideas, serverless SPA hosting (Amplify), well-suited to agentic AI-assisted development

**Considerations:** TypeScript is preferred for applications with significant complexity. We avoid JavaScript for back-end services (Node.js) and prefer Go for back-end work due to its type consistency and performance characteristics. The front-end ecosystem moves quickly, so evaluate frameworks and libraries for stability before committing. Hosting through services like AWS Amplify has proven effective for eliminating infrastructure overhead for SPAs.

See [Front-end JavaScript Development Philosophy](javascript-development.md) and [Vue Development Philosophy](vue-development.md) for coding standards.

### Python

**Use when:** Data analysis, machine learning, scripting, automation, prototyping algorithms, scientific computing

**Strengths:** Data science libraries, readability, rapid development, extensive scientific computing ecosystem

**Considerations:** Not currently a primary application development language in our stack. Use for its strengths in data and automation; avoid using it where Go or PHP would be more appropriate for web services.

### Beyond Languages

The principles in this document apply equally to database, infrastructure, and tooling decisions. A few philosophical positions are worth making explicit.

#### Databases

Let the data access pattern drive the database choice, not habit or convention. We host relational databases on AWS RDS and in practice prefer MySQL or MySQL-compatible technology. PostgreSQL is a valid choice where its strictness and advanced type system are an advantage. Redis serves well for in-memory caching.

When a problem looks like it needs a NoSQL database, evaluate carefully. We have assessed options such as MongoDB and Cassandra and have generally preferred the JSON capabilities of MySQL, which often provide the document-style flexibility of NoSQL without introducing an entirely new database technology to the stack. The evaluation itself is the important part: understand what the data needs before reaching for a new tool, and be honest about whether the existing stack can serve the requirement well enough.

For narrow, well-suited use cases such as DynamoDB for session storage, purpose-built datastores are a pragmatic choice.

#### Infrastructure

We are not in the server maintenance business. We prefer AWS managed infrastructure that lets us focus on building products and services rather than maintaining operating systems and runtime environments.

In practice, this means we favour managed compute services (ECS for containerised workloads, Lambda for serverless functions) and reach for EC2 only when OS-level configurability is a genuine requirement. We build our own container images for our products and services, which gives us control over the runtime while leaving orchestration and scaling to AWS.

We commit to AWS as our cloud provider but design for portability where practical. Containerising applications makes them inherently portable, even when we deploy to AWS-specific services like ECS or Lambda. This is not about planning to leave AWS. It is about making good architectural choices that happen to keep options open.

Deployment strategies vary by context (container images, CodeDeploy, bespoke scripts) and the right approach depends on the service and its operational requirements. Consistency within a service matters more than uniformity across all services.

#### Infrastructure as Code

We value infrastructure as code where we own our own platform. Defining infrastructure declaratively provides repeatability, auditability, and the ability for the development team to understand and evolve the environments their services run in. Where platform ownership is shared or constrained by external parties, pragmatism applies.

---

## Decision Framework

When facing a technology decision, work through these questions in order.

### Can an existing technology do the job?

Before evaluating new options, consider whether something already in the stack can serve the need. This is not about defaulting to the familiar; it is about being intentional:

1. **Does an existing technology fit the problem well?** If yes, using it avoids unnecessary complexity.
2. **What would we sacrifice by using the existing technology?** Be honest about the trade-offs in both directions: the cost of something new *and* the cost of sticking with something that does not fit well.
3. **Is there a learning or strategic reason to explore something new?** Enthusiasm, curiosity, and the desire to build new capability are legitimate reasons to look beyond the current stack. They just benefit from being stated openly.

### Evaluating a new technology

When exploring a new technology, whether driven by a specific problem, strategic direction, or curiosity, evaluate it against these criteria:

1. **Problem fit** — Does this technology solve the specific problem better than alternatives?
2. **Ecosystem maturity** — Is the ecosystem stable, well-documented, and actively maintained?
3. **Learning and growth** — What does the team gain by learning this? Does it build capability that serves us beyond the immediate use case?
4. **Team investment** — What support does the team need to succeed with this? Dedicated learning time, mentoring, training resources?
5. **Integration** — Does it work with our existing infrastructure, deployment, and monitoring?
6. **Licensing** — Does the licence permit us to fulfil our objectives?
7. **Long-term viability** — Will this technology be supported and staffed in five years?
8. **Complexity cost** — Is the benefit (including learning) worth the addition to our stack's complexity budget?

### How should it be introduced?

If the decision is to adopt:

1. **Identify a bounded first project** — a small, well-defined scope where the risk of learning is contained
2. **Assign a champion** — someone motivated to explore, build a proof of concept, and mentor others
3. **Evaluate the proof of concept** — does it meet the original intention? What was learned?
4. **Share knowledge** — through peer reviews, informal sessions, documentation, and mentoring
5. **Expand gradually** — increase usage as the team gains knowledge, understanding, and confidence
6. **Document patterns** — write coding standards, guidelines, and internal best practices as understanding matures

### When should a technology be retired?

Technologies should be retired when:

- A clearly superior alternative is established and proven in our stack
- The ecosystem is declining (fewer maintainers, stale dependencies, shrinking community)
- The technology creates disproportionate operational or maintenance burden
- The talent pool has dried up, making hiring and onboarding unreasonably difficult

Retirement should be planned, not abrupt:

1. **Stop new adoption** — no new projects should use the retiring technology
2. **Document the migration path** — how existing systems will transition
3. **Migrate incrementally** — prioritise high-value or high-risk systems first
4. **Plan for refactoring** — as understanding of the replacement matures, revisit early implementations

---

## Anti-Patterns

These patterns cause problems in practice. Avoid them.

### Résumé-Driven Development

Choosing a technology because it looks good on a CV rather than because it fits the problem. This leads to unnecessary complexity and technologies that the broader team cannot support.

**Prevention:** Evaluate against the problem, not personal career goals. Apply the decision framework honestly.

### Hype Without Evaluation

Adopting a technology based solely on industry buzz, without doing the work to understand whether it fits our context. Enthusiasm is valuable, but enthusiasm without evaluation is a gamble.

The distinction is important: being an early adopter because you have evaluated the technology and believe in its potential is courageous innovation. Adopting because everyone is talking about it, without testing, is hype.

**Prevention:** Channel enthusiasm into proof-of-concept evaluation. Assess honestly, including limitations. Seek perspectives from sceptics as well as advocates.

### Comfort-Zone Lock-in

Refusing to consider alternatives because the current technology is familiar. This is the opposite of hype-driven adoption but equally harmful. It prevents the team from evolving.

**Prevention:** Recognise the limitations of existing tools alongside their strengths. Create space for exploration and evaluation of alternatives. Pro-actively look for improvements even when maintaining the status quo is comfortable.

### Big-Bang Adoption

Attempting to adopt a new technology across the entire stack at once. This overwhelms the team, introduces risk across multiple systems simultaneously, and provides no contained environment for learning.

**Prevention:** Follow the incremental adoption principle. Start small, prove value, build confidence, then expand.

### Silo-Driven Divergence

Different teams or individuals adopting different technologies for the same class of problem, leading to a fragmented stack with duplicated effort and no shared knowledge.

**Prevention:** Make technology decisions collaboratively. Ensure visibility across teams. Apply the strategic consistency principle.

---

## Related Documents

- [Go Development Philosophy](go-development.md)
- [PHP Development Philosophy](php-development.md)
- [Front-end JavaScript Development Philosophy](javascript-development.md)
- [Vue Development Philosophy](vue-development.md)
- [Service-Oriented Architecture Philosophy](service-oriented-architecture.md)
- [API Design Philosophy](api-design.md)
