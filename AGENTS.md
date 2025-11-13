# AI Agent Guidelines and Project Structure

## Overview

This document provides guidelines for AI agents working on this project. It covers project organization, communication standards, workflow patterns, and agentic behavior rules derived from successful AI-assisted development patterns.

## Project Folder Structure

### AI-Generated Artifacts Organization

All AI-generated documentation and artifacts should be organized under the `.ai/` folder at the project root:

```
.ai/
├── planning/          # Planning documents and design docs
├── investigations/    # Debugging sessions, issue analysis, and diagnostic work
├── reviews/           # Code review analyses and responses
│   └── [feature]/     # Feature-specific review subdirectories
├── summaries/         # PR summaries, branch analyses, and progress summaries
├── progress/          # Progress reports and status updates
├── contexts/          # Context files listing relevant files for complex features
└── completed/         # Completed work artifacts and deliverables
```

**Folder Purposes**:
- **planning/**: Implementation plans, design documents, feature roadmaps, and architectural decisions
- **investigations/**: Debugging sessions, error analysis, issue diagnostics, and troubleshooting documentation
- **reviews/**: PR review analyses, code review responses, and review-driven refactoring plans
- **summaries/**: Branch comparison summaries, PR description generation, feature completion summaries
- **progress/**: Status updates, milestone reports, and development progress tracking
- **contexts/**: File lists organized by purpose for complex features (PRIMARY/REFERENCE/SHARED/INTEGRATION)
- **completed/**: Final deliverables, templates, and archived artifacts

### Naming Conventions

- **Planning documents**: `YYYY-MM-DD-<descriptive-slug>.md`
- **Investigation documents**: `YYYY-MM-DD-<issue-description-slug>.md`
- **Review documents**: `YYYY-MM-DD-pr[PR_NUMBER]-review-analysis-and-responses.md`
- **Summary documents**: `YYYY-MM-DD-<summary-type>-<slug>.md`
- **Progress reports**: `YYYY-MM-DD-progress-<slug>.md`
- **Context files**: `[feature]-context.md` (no date prefix, updated as needed)

**Examples**:
- Planning: `2025-08-29-plan-witness-implementation-with-tdd-approach.md`
- Investigation: `2025-10-23-debugging-dispatch-error-for-lp-pool.md`
- Review: `2025-10-08-pr50-review-analysis-and-responses.md`
- Summary: `2025-09-25-summarize-branch-changes-for-pr.md`
- Progress: `2025-10-14-progress-bitcoin-integration.md`
- Context: `ada-witness-context.md`, `ethereum-integration-context.md`

### Context Files

Context files help organize relevant file lists for complex features:

```
.ai/
├── contexts/
│   └── [feature]-context.md    # Lists of relevant files organized by purpose
```

**Context file organization patterns**:
- **PRIMARY**: Files you'll modify most often
- **REFERENCE**: Design pattern examples from similar features
- **SHARED**: Common infrastructure used across features
- **INTEGRATION**: Integration points and dependencies

**Example context file** (`.ai/contexts/ada-witness-context.md`):
```markdown
# ADA Witness Implementation Context

## PRIMARY - ADA Implementation Files
- multichain-engine/src/witness/ada.rs
- multichain-engine/src/witness/ada/deposits.rs
- multichain-engine/src/witness/ada/source.rs

## REFERENCE - BTC/ETH Pattern Examples  
- multichain-engine/src/witness/btc/deposits.rs
- multichain-engine/src/witness/eth/deposits.rs

## SHARED - Common Infrastructure
- multichain-engine/src/witness/common/chain_source.rs
- multichain-engine/src/witness/common/chunked_chain_source.rs

## INTEGRATION - State Chain & Runtime
- node/runtime/src/lib.rs
- toolkit/pallets/cf/cf-ingress-egress/src/lib.rs
```

## Agent Guardrails

### Code Modification Boundaries

**CRITICAL**: Follow these rules for code modifications:

- **Do not modify code unless explicitly directed by the user**
- Always request permission before making changes to existing code
- Prefer reading, analyzing, and understanding code over modifying it
- When modifications are requested, implement them precisely as specified
- Focus on factual observations rather than prescriptive changes
- **Never add process attribution comments** to code (no references to AI, agents, users, reviews, or planning documents)

### Implementation Workflow Boundaries

- **For complex implementations**: Complete only ONE numbered task at a time
- Stop and ask user permission before proceeding to the next item
- NEVER implement multiple recommendations in a single iteration
- Provide brief summary of completed work before asking to continue
- Break complex tasks into manageable, specific steps

### File and Directory Restrictions

- Do not modify core configuration files without explicit user direction
- Avoid changes to build scripts unless specifically requested
- Be cautious with database migration files and schema changes
- Request approval before modifying CI/CD configuration or Docker files

## Communication Standards

### Tone and Language

- Use measured, technical language appropriate for senior software engineering
- Avoid hyperbolic statements and superlatives ("amazing", "terrible", "perfect", "excellent")
- Use precise, descriptive language:
  - ✓ "well-structured" instead of "excellent"
  - ✓ "needs improvement" instead of "poor"
  - ✓ "follows established patterns" instead of "perfect"
- Provide respectful, constructive feedback focused on code quality, not authorship
- Focus on technical merit rather than emotional assessments

### Documentation Standards

- **AVOID process attribution references**: Do not include citations referencing how code was generated or reviewed:
  - ❌ User request references: "(per user request)", "(user asked for this)", "(requested by team)"
  - ✓ Direct statements: "Uses X pattern for Y reason" or "Implements Z to handle edge case"
- Present design decisions and implementation choices directly without referencing their origin
- Comments should explain **what** code does and **why** it exists
- Focus on self-documenting code and maintainability
- Keep comment density reasonable (aim for <15% of lines), focused on complex logic
- Explain **why** decisions were made, not **what** the code does (code should be self-explanatory)

**Rationale**: Code comments should document the system as it exists, not the process that created it. Future maintainers need to understand the code's purpose and behavior, not its provenance.

## Context Management Strategy

### Optimal Context Provision

**Direct Context (Preferred for Most Cases)**:
- Use `@filename` or `@filepath` to add specific files to context
- Open relevant files in tabs - agents can see "open and recently viewed files"
- Most explicit and immediate - clear signal of what's important
- Better for focused, specific tasks

**File Lists (Useful for Specific Scenarios)**:
- Document project areas and recurring context patterns
- "Context recipes" for different task types
- Large-scale refactoring requiring many files
- Onboarding documentation for complex features

**Hybrid Approach (Often Best)**:
1. Add a context document explaining the task and scope
2. @ mention 2-4 key files that are most relevant
3. Let the agent search for the rest using semantic search

### Context File Best Practices

- Organize by priority/relevance (most frequently changed files first)
- Include design pattern reference implementations
- Add integration points and dependencies
- Maintain accuracy - update paths when structure changes
- Use clear section headers for different layers/components

## Task Management

### Todo System Usage

Use the todo system for complex tasks:

**When to Create Todos**:
- Complex multi-step tasks (3+ distinct steps)
- Non-trivial tasks requiring careful planning
- Multiple tasks provided as numbered/comma-separated list
- After receiving new instructions (capture requirements)

**When NOT to Create Todos**:
- Single, straightforward tasks
- Trivial tasks with no organizational benefit
- Tasks completable in < 3 trivial steps
- Purely conversational/informational requests

**Todo Management Rules**:
- Only ONE task marked `in_progress` at a time
- Mark complete IMMEDIATELY after finishing
- Complete current tasks before starting new ones
- Update status in real-time
- Create specific, actionable items with clear completion criteria

**Never include operational tasks in todos**: Linting, testing, or code examination are operational activities done in service of higher-level tasks, not tasks themselves.

## Workflow Patterns

### Iterative Implementation

1. **Understand requirements** - Read context and requirements thoroughly
2. **Plan approach** - Create todos for complex tasks
3. **Implement incrementally** - One task at a time with validation
4. **Validate continuously** - Run tests and checks after each change
5. **Document decisions** - Update relevant documentation
6. **Request approval** - Get user approval before proceeding to next task

### Information Gathering

**Before Making Changes**:
- Use codebase search to understand existing patterns
- Read related files to understand context
- Search for similar implementations as reference
- Identify integration points and dependencies

**During Implementation**:
- Validate assumptions by checking actual code
- Cross-reference with related modules
- Check test coverage for patterns
- Verify naming conventions in similar code

## Error Recovery and Edge Cases

### When Things Go Wrong

- Acknowledge errors clearly and directly
- Analyze the root cause before proposing fixes
- Don't make the same mistake multiple times
- Learn from correction patterns in project history
- Ask clarifying questions when uncertain

### Handling Ambiguity

- When requirements are unclear, ask specific questions
- Propose concrete options with tradeoffs
- Reference similar patterns in codebase for context
- Don't guess at critical implementation details

## Project-Specific Extensions

### Custom Tooling

Document any project-specific tools or scripts:
- Build automation scripts
- Development environment setup
- Testing utilities
- Code generation tools

### Integration Points

Document key integration points:
- External service APIs
- Database schemas
- Message formats
- Configuration files

## Version Control Practices

### Commit Standards

- Use clear, descriptive commit messages
- Reference issue numbers when applicable
- Break changes into logical, atomic commits
- Follow project commit message format if defined

### Branch Management

- Understand project branching strategy
- Follow naming conventions for feature branches
- Keep branches focused on specific features/fixes
- Regularly sync with main branch

### Destructive Operations

- **NEVER** run destructive/irreversible operations without explicit request:
  - Force push to protected branches
  - Hard resets that lose history
  - Mass deletions without confirmation
- **NEVER** skip hooks (--no-verify, --no-gpg-sign) unless explicitly requested
- **NEVER** commit changes unless user explicitly asks

## Further Reading

### Project Documentation

- Architecture documentation
- Developer guides
- API documentation
- Setup and installation guides

### External Resources

- Framework documentation
- Language best practices
- Design patterns
- Testing methodologies

---

## Summary

This document provides comprehensive guidelines for AI-assisted development. The core principles are:

1. **Respect code ownership** - Don't modify without permission
2. **Communicate clearly** - Use professional, technical language
3. **Work incrementally** - One task at a time with validation
4. **Maintain quality** - Follow established patterns and gates
5. **Document decisions** - Keep documentation current and clear
6. **Organize artifacts** - Use consistent folder and naming structure
7. **Prevent issues** - Apply review criteria as design constraints
8. **Stay focused** - Complete current work before starting new tasks
9. **No process attribution** - Never reference AI, user requests, reviews, or planning documents in code comments

Following these guidelines ensures productive, high-quality AI-assisted development that aligns with project standards and team expectations.

