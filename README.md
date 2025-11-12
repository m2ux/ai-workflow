# AI-Augmented Software Engineering Workflow Automation

A curated collection of **prompts**, **methodologies**, and **resources** for enhancing software engineering workflows with AI assistance. This project captures proven techniques for automating and improving common development activities through structured AI collaboration.

## Purpose

This repository serves as a knowledge base for AI-augmented software engineering, focusing on:

- **Systematic workflows** for complex engineering tasks
- **Quality-driven automation** through structured prompts
- **Best practices** for AI-assisted code review, testing, and design
- **Proven methodologies** adapted for AI collaboration (TRIZICS, TDD)

Whether you're working with AI coding assistants like Claude, GPT, or other LLMs, these resources provide structured approaches to common engineering challenges.

---

## Project Structure

```
ai-workflow/
├── prompts/          # Structured prompts for specific workflows
├── resources/        # Reference materials and methodologies
└── rust-llms.txt     # Curated Rust learning resources
```

### 📝 Prompts

Comprehensive prompt templates for AI-assisted engineering tasks:

| Prompt | Purpose | Key Features |
|--------|---------|--------------|
| **[PR Review Analysis](prompts/pr-review-analysis-and-response-prompt.md)** | Systematically analyze and respond to GitHub PR reviews | Comment categorization, response templates, follow-up tracking |
| **[Progress Reports](prompts/progress-report-from-chat-and-git.md)** | Generate progress reports from chat transcripts and git diffs | Automated requirement tracking, structural change analysis |
| **[Rust Test Creation](prompts/rust-test-creation-template.md)** | Create comprehensive Rust tests using TDD principles | TDD strategy selection, test doubles, quality checklists |
| **[Rust Testing Review](prompts/rust-testing-review-prompt.md)** | Review existing Rust tests for quality and completeness | Coverage analysis, anti-pattern detection |
| **[Rust Substrate Review](prompts/rust-substrate-review-prompt.md)** | Review Substrate/Polkadot blockchain code | Substrate-specific patterns and best practices |
| **[TRIZICS Software Design](prompts/trizics-software-design-prompt.md)** | Systematic problem-solving for software design | Contradiction analysis, inventive principles |

### 📚 Resources

Supporting methodologies and reference materials:

- **[TDD Concepts for Rust](resources/tdd-concepts-rust.md)** - Comprehensive lexicon of Test-Driven Development adapted for Rust from "Test Driven Development for Embedded C"
- **[TRIZICS Software Design Suite](resources/)**
  - [README](resources/trizics-software-design-README.md) - Overview and usage guide
  - [Quick Reference](resources/trizics-software-design-quick-reference.md) - Cheat sheet for design sessions
  - [Practical Example](resources/trizics-software-design-example.md) - Real-world walkthrough

### 🦀 Rust Resources

**[rust-llms.txt](rust-llms.txt)** - Curated collection of resources for writing idiomatic Rust code, including:
- Core learning resources (The Book, Rust by Example, Design Patterns)
- Articles on Rust idioms and best practices
- Workshops and interactive learning materials
- Code examples and community discussions

---

## Key Methodologies

### TRIZICS for Software Design

**TRIZICS** (adapted from Gordon Cameron's systematic problem-solving methodology) provides a structured approach to software design:

1. **Inside-the-Box First** - Start with proven patterns and conventional solutions
2. **Outside-the-Box When Needed** - Apply inventive principles for complex contradictions
3. **Systematic Process** - Follow a clear roadmap from problem definition to solution

**When to Use:**
- Complex architectural decisions
- Contradictory requirements (performance vs. maintainability, security vs. usability)
- High-stakes design choices
- When conventional approaches have failed

**Resources:**
- [Full Methodology Prompt](prompts/trizics-software-design-prompt.md)
- [Quick Reference Guide](resources/trizics-software-design-quick-reference.md)
- [Practical Example](resources/trizics-software-design-example.md)

### Test-Driven Development for Rust

Comprehensive TDD practices adapted from "Test Driven Development for Embedded C":

- **Red-Green-Refactor Cycle** - The core TDD workflow
- **Test Doubles** - Mocks, stubs, spies, fakes for Rust
- **Breaking Dependencies** - Techniques for making code testable
- **Quality Principles** - FIRST principles and test organization

**When to Use:**
- Writing tests for new Rust code
- Retrofitting tests to existing code
- Improving test quality and coverage
- Learning TDD best practices

**Resources:**
- [Test Creation Template](prompts/rust-test-creation-template.md)
- [TDD Concepts Reference](resources/tdd-concepts-rust.md)

---

## How to Use This Repository

### For AI Assistants (Claude, GPT, etc.)

1. **Load relevant prompts** into the AI context for specific tasks
2. **Reference methodology documents** when working through complex problems
3. **Use templates** to structure responses and deliverables
4. **Follow checklists** to ensure comprehensive coverage

### For Software Engineers

1. **Start with a prompt** that matches your current task
2. **Collaborate with AI** using the structured workflow
3. **Reference methodology docs** for deeper understanding
4. **Adapt templates** to your specific context

### For Teams

1. **Standardize workflows** using these prompts as templates
2. **Train AI assistants** with project-specific variations
3. **Build shared vocabulary** around methodologies (TRIZICS, TDD)
4. **Document decisions** using the structured formats

---

## Workflow Examples

### Code Review Response Workflow

1. Use [PR Review Analysis prompt](prompts/pr-review-analysis-and-response-prompt.md)
2. Gather all PR comments via GitHub CLI
3. Filter to unresolved comments
4. Generate comprehensive response document
5. Implement approved follow-up actions
6. Post responses to PR threads

### Test Creation Workflow

1. Use [Rust Test Creation prompt](prompts/rust-test-creation-template.md)
2. Analyze code context (new vs. existing)
3. Plan test strategy using [TDD Concepts](resources/tdd-concepts-rust.md)
4. Generate comprehensive test suite
5. Apply quality checklist
6. Iterate with Red-Green-Refactor

### Design Decision Workflow

1. Use [TRIZICS Design prompt](prompts/trizics-software-design-prompt.md)
2. Define problem thoroughly (Phase 1)
3. Select problem type - specific problem vs. inventive goal (Phase 2)
4. Explore conventional solutions (Phase 3)
5. Apply inventive thinking if needed (Phase 4)
6. Document design decisions (Phase 5)

---

## Best Practices

### Prompt Engineering

- **Be Specific**: Provide clear context and constraints
- **Use Checklists**: Follow structured verification steps
- **Iterate**: Refine prompts based on results
- **Document**: Capture successful prompt patterns

### AI Collaboration

- **Verify Outputs**: Always review AI-generated code and documentation
- **Provide Feedback**: Help AI understand project-specific patterns
- **Use Examples**: Reference existing code as patterns
- **Maintain Standards**: Ensure consistency with project conventions

### Continuous Improvement

- **Collect Examples**: Build a library of successful applications
- **Share Learnings**: Document what works in your context
- **Adapt Templates**: Customize prompts for your domain
- **Measure Impact**: Track time saved and quality improvements

---

## Contributing

This repository is designed to evolve based on real-world usage. Consider:

- **Adding new prompts** for additional workflows
- **Enhancing existing templates** with lessons learned
- **Contributing examples** of successful applications
- **Documenting adaptations** for specific domains or technologies

---

## File Conventions

- **Prompts**: Stored in `prompts/` directory with descriptive names
- **Resources**: Reference materials in `resources/` directory
- **Planning**: AI-assisted planning documents in `.ai/planning/`
- **Reviews**: PR review analysis in `.ai/reviews/[feature-area]/`
- **Progress**: Progress reports in `.ai/progress/`

---

## Quick Start

1. **Identify your task** (PR review, test creation, design decision)
2. **Select the relevant prompt** from the `prompts/` directory
3. **Load into your AI assistant** (copy prompt into context)
4. **Provide task-specific inputs** (code, PR number, requirements)
5. **Follow the structured workflow** in the prompt
6. **Review and iterate** on the results

---

## Resources for Learning

### TRIZICS Methodology
- **Book**: "TRIZICS: Teach yourself TRIZ, how to invent, innovate and solve 'impossible' technical problems systematically" by Gordon Cameron
- **Theory**: Based on TRIZ (Theory of Inventive Problem Solving) by Genrich Altshuller

### Test-Driven Development
- **Book**: "Test Driven Development for Embedded C" by James W. Grenning
- **Adaptation**: [TDD Concepts for Rust](resources/tdd-concepts-rust.md) in this repository

### Idiomatic Rust
- **Resources**: [rust-llms.txt](rust-llms.txt) - Comprehensive collection of Rust learning materials
- **Official Docs**: [The Rust Book](https://doc.rust-lang.org/book/), [Rust by Example](https://doc.rust-lang.org/rust-by-example/)

---

## License and Attribution

These materials adapt existing methodologies (TRIZICS, TDD) for AI-augmented software engineering workflows. Original methodologies are credited to their respective authors:

- **TRIZICS**: Gordon Cameron
- **TDD for Embedded C**: James W. Grenning
- **Rust Resources**: Various community contributors (see rust-llms.txt)

---

## Feedback and Questions

For questions about using these workflows or suggestions for improvements:

1. **Experiment**: Try the prompts with your AI assistant
2. **Document Results**: Note what works and what doesn't
3. **Adapt**: Customize for your specific context
4. **Share**: Contribute improvements back to the repository

---

**Remember**: These workflows are guides, not rigid rules. Adapt them to your context, technology stack, and team preferences. The goal is systematic thinking and consistent quality, not adherence to a process for its own sake.

