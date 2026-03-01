# The SDLC Is Being Rewritten. Most Engineering Leaders Aren't Ready.

Back in early November 2025, I started questioning an assumption I'd held for my entire career: that the value of a software organisation is proportional to the number of engineers who can write good code.

I'd been experimenting with Claude Code — not as a productivity tool for individual contributors, but as a vehicle for a different operating model. Custom architectural constraints, automated quality enforcement, agent orchestration. The early results were uneven, but the signal was clear: the more structure I provided, the less human intervention the system needed. Not incrementally less. Categorically less.

Then, a few months later, OpenAI published their [harness engineering](https://openai.com/index/harness-engineering/) post — describing how their Codex team built a million lines of production code in five months without any manually written source files. Their engineers had stopped writing code entirely. Instead, they designed constraints, wrote specifications, and built feedback loops for agents to operate within.

Reading it, I realised two things. First, this validated the patterns I'd been discovering through experimentation. Second, and more importantly, it gave the approach a systematic framework — one that has profound implications for how we structure engineering organisations, measure productivity, and think about technical leadership.

That convergence led me to build Claude Code Forge — and to rethink what a modern SDLC should actually look like.

## The Strategic Question

Every CTO and VP of Engineering I talk to is asking the same surface-level question: *How do we use AI to make our developers faster?*

It's the wrong question.

The right question is: **What does the engineering organisation look like when AI agents handle execution and humans handle intent?**

This isn't a tools conversation. It's an operating model conversation. And the organisations that treat it as the former will find themselves structurally disadvantaged against those that recognise it as the latter.

## The Operating Model: Human Intent, Agent Execution

Claude Code Forge implements a complete SDLC pipeline with a simple principle:

> Humans define what to build. Agents handle how to build it. Specifications are the contract between the two.

The pipeline has 11 phases and exactly two human decision points:

```
SPEC → STORIES → DESIGN → TEST PLAN → EXEC PLAN → [APPROVE]
→ IMPLEMENT → TEST → E2E → DEVOPS → REVIEW → [APPROVE] → PR
```

The first approval gate is strategic: *Is this the right thing to build, and is the plan sound?* The second is evaluative: *Did the system deliver what was specified?* Everything between those two gates — decomposition, implementation, testing, review — is autonomous.

Twenty specialised agents. Twenty-three reusable skills. Eleven quality enforcement hooks. This isn't a prototype — it's a working delivery system.

## What Makes This Work at Scale

The OpenAI team articulated a principle I've found to be absolutely foundational: **mechanical enforcement over documentation.** It's a principle that applies far beyond agent orchestration — it's a leadership principle.

Here's what that looks like in practice:

### Architectural Constraints as Governance

```
Types → Config → Repo → Service → Runtime → UI
```

Dependencies flow in one direction only. This isn't documented in a wiki that nobody reads — it's enforced by linters that run on every file write. Agents cannot introduce architectural violations. Neither can a senior engineer under deadline pressure at 11 PM.

The implication for engineering leaders: **the cost of architectural governance drops to near zero when it's mechanically enforced.** No architecture review boards. No "we'll fix it later" technical debt accumulation. The system simply doesn't allow structural regression.

### Specification as the Single Source of Truth

In most organisations, requirements fragment across Jira tickets, Slack threads, Confluence pages, meeting recordings, and individual engineers' mental models. By the time code is written, the specification has been through a game of telephone.

In this model, a spec-writer agent conducts a structured extraction — acceptance criteria, data models, API contracts, business rules, edge cases — producing a single versioned document that every downstream agent references deterministically.

For leadership, the insight is this: **specification quality becomes the highest-leverage activity in the organisation.** A well-written spec compounds through every phase. A vague spec compounds too — in the wrong direction.

### Quality as Infrastructure, Not Culture

The industry has spent decades trying to make quality a cultural value. Write tests. Do code reviews. Follow the style guide. It works until deadlines hit, and then it doesn't.

This system makes quality infrastructural:

- **TDD is mandatory** — hooks verify that failing tests exist before any production code is written. Coverage gates enforce 80%+ minimum. This isn't discipline; it's architecture.
- **Eight parallel reviewers** examine every PR — five gating (code quality, security, performance, architecture, spec compliance) and three advisory (design consistency, simplification, requirements alignment). All five gating reviewers must approve.
- **Eleven lifecycle hooks** enforce constraints in real-time — at the moment code is written, not days later in a review.

The leadership takeaway: **you can stop relying on individual discipline and start relying on system design.** Quality becomes a property of the pipeline, not a hope about the team.

### Parallelisation Without Coordination Tax

When a feature decomposes into multiple stories, the system spawns parallel agent teams — each working in isolated sessions with file ownership rules preventing conflicts. A team lead agent distributes work, monitors progress, and resolves dependencies.

This is where the economics become compelling. In traditional engineering, adding headcount to a late project makes it later (Brooks's Law). The coordination overhead scales superlinearly with team size. Agent teams don't have this problem — the architecture and ownership rules eliminate the ambiguity that drives human coordination costs.

## The Implications for Engineering Leadership

I want to be direct about what I think this means for our industry:

**The role of the software engineer is evolving.** Not disappearing — evolving. The most valuable engineers will be the ones who can write precise specifications, design robust architectural constraints, and evaluate outcomes critically. The mechanical translation of intent into syntax becomes a commodity.

**Engineering productivity metrics need to change.** Lines of code, PRs merged, velocity points — these measure output of the old model. In an agent-orchestrated model, the metrics that matter are specification quality, constraint coverage, and defect escape rate. We need to measure the quality of the harness, not the volume of its output.

**Organisational structure will shift.** Fewer implementation engineers, more specification engineers, architect-engineers, and quality engineers. The ratio changes dramatically when execution is automated. A small team with a well-designed harness can deliver what previously required a department.

**Technical debt becomes a design choice, not an accident.** When architecture is mechanically enforced and quality is infrastructural, technical debt doesn't accumulate through attrition. It only exists where you deliberately choose to accept it — which is how it should have always worked.

## Five Principles for the Transition

For engineering leaders evaluating this shift, here's what I've learned:

**1. Constraints enable autonomy.** This is counterintuitive but fundamental. The more rigid the architectural boundaries and conventions, the more reliably agents operate. Giving agents freedom produces inconsistency. Giving them guardrails produces remarkable quality. The same principle, incidentally, applies to human engineering teams — but we've historically lacked the tooling to enforce it without bureaucracy.

**2. Invest in specification capability.** This is where your highest-leverage talent should be deployed. A well-written spec saves 10x the implementation time. Agent-written code without a spec is sophisticated technical debt. The organisations that win will be the ones with the best spec writers, not the best coders.

**3. Two approval gates is the sweet spot.** Approve the plan. Approve the result. Everything between is execution. More gates add friction without adding safety. Fewer gates remove the human's ability to course-correct. This mirrors good executive leadership: set direction, evaluate outcomes, don't micromanage execution.

**4. Treat the harness as a strategic asset.** The real value isn't any individual application the system produces. It's the system itself — portable, reproducible, and improving with every project. Each engagement teaches the harness something new, and that learning persists. This is a compounding advantage.

**5. Start now, but start with structure.** The temptation is to let individual engineers experiment with AI tools ad hoc. That produces local optimisations and global inconsistency. Instead, invest in the scaffolding: architectural enforcement, quality hooks, specification templates, review pipelines. The infrastructure comes first; the productivity follows.

## The Bigger Picture

The future of software delivery isn't AI-assisted. It's AI-executed, human-directed.

We're not eliminating engineering roles. We're elevating them. From writing code to engineering the systems that write it. From firefighting production incidents to designing the constraints that prevent them. From arguing about code style in pull requests to defining the architectural boundaries that make style arguments irrelevant.

The organisations that recognise this shift early — and restructure accordingly — will operate at a fundamentally different velocity than those still optimising for the old model.

If your AI strategy for engineering is "give everyone Copilot," you're optimising for a world that's already passing.

---

*I've open-sourced the Claude Code Forge scaffolding framework. If you're leading an engineering organisation and thinking about agent-orchestrated delivery, I'd welcome the conversation — whether you're already experimenting or just starting to evaluate the shift. Let's connect.*

---

#EngineeringLeadership #AI #SDLC #HarnessEngineering #SoftwareArchitecture #CTO #AgentOrchestration #FutureOfWork #TechnicalStrategy #DevOps
