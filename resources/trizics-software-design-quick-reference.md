# TRIZICS Software Design - Quick Reference Guide

## The TRIZICS Software Design Roadmap

```
┌─────────────────────────────────────────────────────────┐
│ STEP 1: IDENTIFY & DEFINE PROBLEM                      │
│ • Problem statement                                     │
│ • System understanding                                  │
│ • Impact & success criteria                            │
│ • Timeline & constraints                               │
│ • Root cause analysis                                   │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ STEP 2: SELECT PROBLEM TYPE                            │
│                                                         │
│ ┌──────────────────────┐  ┌──────────────────────┐   │
│ │ SPECIFIC PROBLEM     │  │ INVENTIVE GOAL       │   │
│ │                      │  │                      │   │
│ │ • Cause Known        │  │ • Improvement        │   │
│ │ • Cause Unknown      │  │ • Prevention         │   │
│ └──────────────────────┘  └──────────────────────┘   │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ STEP 3: CONVENTIONAL SOLUTIONS (Inside-the-Box)        │
│ • Design patterns                                       │
│ • Best practices                                        │
│ • Existing solutions                                    │
│ • Constraint analysis                                   │
└────────────────┬────────────────────────────────────────┘
                 │
                 ├──> Solution Found? ──> Go to Step 5
                 │
                 ▼ No adequate solution
┌─────────────────────────────────────────────────────────┐
│ STEP 4: INVENTIVE SOLUTIONS (Outside-the-Box)          │
│ • Identify contradictions                              │
│ • Apply inventive principles                           │
│ • Ideal solution thinking                              │
│ • Resource utilization                                  │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│ STEP 5: SOLUTION SYNTHESIS & DESIGN                    │
│ • Architecture design                                   │
│ • Design decisions & trade-offs                        │
│ • Implementation roadmap                               │
│ • Success metrics                                       │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Problem Definition Checklist

Use this checklist to ensure you've thoroughly defined the problem:

- [ ] **Problem Statement**: Can you describe it in one clear sentence?
- [ ] **Visual Understanding**: Have you diagrammed the current system?
- [ ] **Core Function**: What is the primary useful function needed?
- [ ] **Impact**: What's the cost of NOT solving this?
- [ ] **Success Criteria**: What does "done" look like?
- [ ] **Timeline**: When do you need this solved?
- [ ] **Resources**: What team, budget, and tools are available?
- [ ] **Constraints**: What limitations must be respected?
- [ ] **Root Cause**: Have you identified why this problem exists?
- [ ] **Stakeholders**: Who is affected and how?

---

## Common Software Design Contradictions

Recognizing contradictions helps unlock inventive solutions:

| Contradiction | Description | Example |
|--------------|-------------|---------|
| **Performance vs. Maintainability** | Fast code is often complex | Optimized algorithms vs. readable code |
| **Security vs. Usability** | More secure = more friction | Multi-factor auth vs. quick access |
| **Flexibility vs. Simplicity** | Extensible systems are complex | Plugin architecture vs. monolithic simplicity |
| **Consistency vs. Availability** | CAP theorem trade-off | Strong consistency vs. high availability |
| **Coupling vs. Performance** | Decoupling adds overhead | Microservices latency vs. monolith efficiency |
| **Generality vs. Specificity** | General solutions are slower | Generic framework vs. custom solution |
| **Innovation vs. Stability** | New features risk breaking things | Rapid deployment vs. thorough testing |
| **Automation vs. Control** | Automated systems are less controllable | Auto-scaling vs. manual resource management |

---

## Top 20 Inventive Principles for Software Design

Quick reference for inventive thinking:

1. **Segmentation** → Microservices, modules, components
2. **Extraction** → Separate concerns, extract services
3. **Local Quality** → Polyglot persistence, hybrid architectures
4. **Asymmetry** → Fast/slow paths, tiered solutions
5. **Consolidation** → Batch processing, unified APIs
6. **Universality** → Reusable services, platform approaches
7. **Nesting** → Layered architecture, containers
8. **Counterweight** → Caching, redundancy, compensation
9. **Preliminary Action** → Pre-computation, circuit breakers
10. **Cushion in Advance** → Graceful degradation, fallbacks
11. **Equipotentiality** → Event-driven, reactive systems
12. **Inversion** → Push vs. pull, pessimistic vs. optimistic
13. **Dynamicity** → Auto-scaling, feature flags
14. **Partial Action** → Eventually consistent, sampling
15. **Different Dimension** → Temporal/spatial distribution
16. **Mediator** → API gateway, message queue, proxy
17. **Self-Service** → Self-healing, auto-remediation
18. **Copying** → Mocks, digital twins, shadow systems
19. **Cheap/Disposable** → Serverless, stateless services
20. **Transformation** → Problems become solutions (logs → analytics)

---

## Decision Matrix Template

Use this to evaluate solution options:

| Solution Option | Pros | Cons | Effort | Risk | Meets Criteria? |
|----------------|------|------|--------|------|-----------------|
| Option 1       |      |      |        |      |                 |
| Option 2       |      |      |        |      |                 |
| Option 3       |      |      |        |      |                 |

---

## When to Use Each Phase

### Use Full TRIZICS Process When:
- Problem is complex or ambiguous
- High stakes or significant investment
- Multiple stakeholders with competing interests
- Previous attempts have failed
- Innovation is required

### Use Abbreviated Process When:
- Problem is well-understood
- Standard patterns clearly apply
- Low risk, quick iteration possible
- Time pressure for decision
- Similar problems solved before

---

## Red Flags That You Need Inventive Thinking

Watch for these signals that conventional approaches won't work:

🚩 "We've tried everything and nothing works"
🚩 "It's impossible to have both X and Y"
🚩 "We're stuck between two bad options"
🚩 "Every solution breaks something else"
🚩 "The requirements are contradictory"
🚩 "We need a breakthrough, not an incremental improvement"
🚩 "All the obvious solutions have been tried"
🚩 "We're hitting fundamental limits"

→ **These indicate you should move to Phase 4: Inventive Solutions**

---

## Questions to Ask at Each Phase

### Phase 1: Problem Definition
- "What exactly are we trying to solve?"
- "Why is this a problem?"
- "What does success look like?"

### Phase 2: Problem Type
- "Is this a known or unknown problem?"
- "Are we fixing or improving?"
- "Is this reactive or proactive?"

### Phase 3: Conventional Solutions
- "What's the standard way to solve this?"
- "What have others done?"
- "Can existing patterns work?"

### Phase 4: Inventive Solutions
- "What's the core contradiction?"
- "What if we could do the impossible?"
- "What resources are we not using?"

### Phase 5: Solution Design
- "How will this work?"
- "What could go wrong?"
- "How will we know it's working?"

---

## Common Pitfalls to Avoid

❌ **Jumping to Solutions Too Early**
→ Spend time understanding the problem first

❌ **Over-Engineering Simple Problems**
→ Try conventional solutions before getting inventive

❌ **Ignoring Constraints**
→ Real constraints inform good design

❌ **Not Identifying Root Cause**
→ Solving symptoms wastes time

❌ **Forgetting About Implementation**
→ Design must be implementable with available resources

❌ **Skipping Success Criteria**
→ How will you know if it works?

❌ **Not Documenting Decisions**
→ Future you will wonder "why did we do it this way?"

---

## Success Indicators

You're on the right track when:

✅ The problem is clearly articulated
✅ Stakeholders agree on success criteria
✅ Multiple solution options have been considered
✅ Trade-offs are explicit and understood
✅ There's a clear path from design to implementation
✅ Risks are identified with mitigation plans
✅ The solution addresses root causes, not symptoms
✅ Design decisions have documented rationale

---

## Deliverables Checklist

At the end of the process, you should have:

- [ ] **Problem Definition Document**
  - [ ] Problem statement
  - [ ] Root cause analysis
  - [ ] Success criteria
  - [ ] Constraints

- [ ] **Solution Design Document**
  - [ ] Architecture overview
  - [ ] Component descriptions
  - [ ] Key design decisions
  - [ ] Trade-offs made
  - [ ] Alternatives considered

- [ ] **Implementation Plan**
  - [ ] Phased roadmap
  - [ ] Dependencies
  - [ ] Risk assessment
  - [ ] Success metrics
  - [ ] Review checkpoints

---

## Quick Start: Three Questions

If you only have time for three questions, ask these:

1. **"What problem are we solving?"** (Get clarity)
2. **"What does success look like?"** (Define target)
3. **"What's the simplest thing that could work?"** (Start inside-the-box)

Then expand as needed based on problem complexity.

---

## Remember: The TRIZICS Philosophy

> **"Think inside-the-box first, then outside-the-box when needed."**

- Start with what's known and proven
- Only get inventive when necessary
- Be systematic, not random
- Document your thinking
- Focus on implementation, not just ideation

---

## Further Resources

- [Full TRIZICS Software Design Prompt](../prompts/trizics-software-design-prompt.md)
- [Practical Example](trizics-software-design-example.md)
- [README & Overview](trizics-software-design-README.md)
- Original TRIZICS methodology: Gordon Cameron's "TRIZICS" book

