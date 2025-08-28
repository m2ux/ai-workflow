# PR Review Analysis and Response Prompt

## Purpose
This prompt helps analyze review comments from a GitHub Pull Request and craft thoughtful, professional responses in markdown format for easy copying into PR replies.

## Prerequisites
- GitHub CLI (`gh`) must be installed and authenticated
- Access to the target repository
- **CRITICAL**: A specific PR number must be provided - do not proceed without one

## Instructions

### Step 1: Gather PR Comments
**RULE**: Do not proceed without a specific PR number being provided by the user.

#### 1.1: Retrieve All PR Comments
Use the following command to retrieve all review comments for the specified PR:
```bash
gh api repos/input-output-hk/partner-chains-mcs/pulls/[PR_NUMBER]/comments --paginate
```

Replace `[PR_NUMBER]` with the actual PR number provided by the user.

#### 1.2: Verify Complete Comment Capture
**CRITICAL**: Ensure no comments are missed by performing verification:

```bash
# Get total comment count
gh api repos/input-output-hk/partner-chains-mcs/pulls/[PR_NUMBER]/comments --paginate | jq '. | length'

# Get all reviewer comments (excluding PR author)
gh api repos/input-output-hk/partner-chains-mcs/pulls/[PR_NUMBER]/comments --paginate | jq '.[] | select(.user.login != "[PR_AUTHOR]") | {id: .id, body: .body, html_url: .html_url}'

# Save all comments for comprehensive analysis
gh api repos/input-output-hk/partner-chains-mcs/pulls/[PR_NUMBER]/comments --paginate | jq '.[] | select(.user.login != "[PR_AUTHOR]")' > /tmp/all_reviewer_comments.json
```

#### 1.3: Identify Question-Type Comments
Find all question-type comments that require specific technical responses:
```bash
cat /tmp/all_reviewer_comments.json | jq -r '.body' | grep -i "what\|how\|why\|which" | nl
```

**Verification Checklist:**
- [ ] Total comment count confirmed using: `gh api ... | jq '[.[] | select(.user.login != "[PR_AUTHOR]")] | length'`
- [ ] All reviewer comments captured (excluding PR author's self-notes)
- [ ] Question-type comments identified and categorized
- [ ] No comments appear truncated or missing
- [ ] Comments saved in structured format for analysis: `/tmp/all_reviewer_comments.json`
- [ ] Editorial suggestions (```suggestion blocks) distinguished from substantive questions
- [ ] Cross-reference final document against complete comment list to prevent gaps

### Step 2: Analyze Comment Relevance
For each review comment retrieved:

1. **Read the comment context**: Note the line numbers, file paths, and specific concerns raised
2. **Compare against current document/code**: Determine if the comment still applies to the latest version
3. **Categorize comments**:
   - Still applicable and needs response
   - Already addressed in updates
   - No longer relevant due to changes
   - Needs clarification or discussion

### Step 3: Create Response List
Generate a numbered list of applicable review comments that still need responses, including:
- Brief description of the concern
- Link to the original GitHub discussion
- Priority level (if applicable)

Format example:
```
1. Clarify "graceful degradation" under Operational Requirements — what behavior is expected when Chain Sync is unavailable. [Discussion link](https://github.com/input-output-hk/partner-chains-mcs/pull/43#discussion_r2288831775)
```

### Step 4: Craft Thoughtful Responses
For each applicable comment, provide responses that are:

**Professional and Technical**:
- Use measured, technical language appropriate for software development
- Avoid hyperbolic statements and superlatives
- Focus on factual observations and technical merit
- Write in the tone of a senior software engineer

**Comprehensive and Nuanced**:
- Address the specific concern raised
- Provide concrete examples where helpful
- Include implementation details when relevant
- Consider trade-offs and alternatives
- Reference related standards or patterns when applicable

**Properly Formatted**:
- Present each response using blockquotes (>) for italic formatting, NOT code blocks
- Use combined comment and link format: `**Comment:** ["comment text"](link)`
- Include "Optional doc wording" suggestions when proposing text changes
- Structure responses for direct pasting into PR comments
- Co-locate questions and responses in the same section for readability

### Step 5: Response Format Template
Use this format for each response section:

```markdown
### [NUMBER]. [Brief Description] [PRIORITY]

**Comment:** ["exact comment text"](https://github.com/[repo]/pull/[PR]#discussion_r[ID])

**Response:**

> [Detailed explanation addressing the concern]
> 
> **[Section heading if needed]:**
> - [Bullet point elaborating on key aspects]
> - [Additional technical details]
> - [Implementation considerations]
> 
> **Optional doc wording:**
> "[Suggested documentation text]"
> 
> [Concluding statement or summary]
```

**Critical Format Requirements:**
- Use `>` blockquotes for ALL response content (creates italic formatting)
- Never use markdown code blocks (` ```markdown `) for responses
- Always combine comment text and link in single line
- Ensure responses are ready for direct copying to PR threads

### Step 6: Create Comprehensive Review Document
**MANDATORY**: Create a complete review analysis document for permanent record and reference.

#### 6.1: Document Location
Create the review document at:
```
[PROJECT_ROOT]/docs/planning/[FEATURE_AREA]/reviews/[YYYY-MM-DD]-pr[PR_NUMBER]-review-analysis-and-responses.md
```

**Examples:**
- For Cardano integration: `docs/planning/cardano-integration/reviews/2025-08-27-pr43-review-analysis-and-responses.md`
- For general features: `docs/planning/reviews/2025-08-27-pr123-review-analysis-and-responses.md`

#### 6.2: Document Structure
The review document must include:

1. **Document Header** with metadata (version, date, PR link, reviewer, analysis method)
2. **Executive Summary** with key findings and reviewer concerns
3. **Analysis Methodology** describing steps followed
4. **Complete Comment List** with all applicable comments and priority levels
5. **Crafted Responses** with ready-to-copy markdown blocks
6. **Additional Technical Questions** (if any were missed in initial analysis)
7. **Additional Technical Responses** (for the missed questions)
8. **Scope Impact Analysis** (features removed/deferred, complexity reduction)
9. **Documentation Restructuring** recommendations (if applicable)
10. **Follow-up Actions** with concrete next steps
11. **Success Criteria** for implementation
12. **Conclusion** with key outcomes and recommendations

#### 6.3: Quality Requirements
- **Complete Coverage**: All reviewer comments addressed, none missed
- **Professional Tone**: Technical communication following repository standards
- **Ready-to-Use Responses**: All responses in copyable markdown code blocks
- **Comprehensive Analysis**: Include impact analysis, scope changes, and technical clarifications
- **Permanent Reference**: Document serves as record for future similar PRs

#### 6.4: Verification Steps
- [ ] All comment IDs from verification step included
- [ ] Response count matches question-type comment count
- [ ] Document includes proper GitHub discussion links
- [ ] All responses follow professional communication standards
- [ ] Document includes complete analysis methodology for reproducibility

## Response Quality Standards

### Technical Accuracy
- Verify technical claims and implementation details
- Reference existing patterns and established practices
- Consider security, performance, and operational implications
- Ensure consistency with project architecture

### Clarity and Completeness
- Define technical terms that may be ambiguous
- Explain the "why" behind technical decisions
- Address potential follow-up questions preemptively
- Provide actionable guidance where appropriate

### Professional Communication
- Acknowledge valid concerns raised by reviewers
- Provide constructive explanations for design decisions
- Offer alternatives when applicable
- Maintain a collaborative, solution-oriented tone

## Example Response Categories

### Clarification Requests
- Define ambiguous technical terms
- Explain implementation approaches
- Clarify design decisions and trade-offs

### Justification Requests
- Explain technology/library choices
- Defend architectural decisions
- Compare alternatives considered

### Implementation Details
- Specify operational behaviors
- Define error handling approaches
- Describe integration patterns

### Documentation Updates
- Propose clearer wording
- Add missing context
- Improve technical explanations

## Output Requirements
1. **Analysis Summary**: List of applicable comments requiring responses
2. **Response Collection**: Individual markdown code blocks for each response
3. **Documentation Updates**: Any suggested text changes for the reviewed document
4. **Follow-up Items**: Any issues requiring further discussion or investigation
5. **Comprehensive Review Document**: Complete analysis document created at specified location following structure requirements

### Review Document Completeness Checklist
- [ ] **Comment Verification**: Total comment count confirmed and all captured (cross-reference against API results)
- [ ] **Complete Analysis**: All reviewer comments addressed with detailed responses (no gaps)
- [ ] **Professional Quality**: All responses follow technical communication standards
- [ ] **Correct Format**: All responses use blockquotes (>) NOT code blocks, with combined comment/link format
- [ ] **Co-located Q&A**: Questions and responses appear in same section for readability
- [ ] **Proper Documentation**: Created at correct location with complete structure
- [ ] **Permanent Record**: Document serves as reference for future similar PRs
- [ ] **GitHub Integration**: All GitHub discussion links included and verified
- [ ] **Format Consistency**: Document follows established template with 24+ sections if comprehensive

## Notes
- Responses should be immediately usable by copying from the blockquoted sections (NOT code blocks)
- All responses must use `>` blockquote format for italic presentation and readability
- Maintain consistency with existing project documentation style
- Consider the reviewer's perspective and expertise level
- Focus on improving the overall quality and clarity of the project documentation
- Never use markdown code blocks for responses - always use blockquotes for proper formatting

## Critical Success Factors

### Comment Capture Completeness
**LESSON LEARNED**: Initial analyses may miss specific technical questions embedded within longer reviews.

**Prevention Strategy**:
1. Always verify total comment count against captured comments
2. Use grep/search to identify question-type comments specifically  
3. Cross-reference GitHub comment IDs to ensure none are skipped
4. Save all comments to temporary files for comprehensive analysis
5. Create supplementary analysis sections for any missed items

### Gap Analysis Methodology
**LESSON LEARNED**: Even with successful API retrieval, analysis may miss substantive comments.

**Systematic Gap Detection**:
1. **Extract Current Coverage**: `grep -o "r[0-9]\{10\}" [review_doc] | sort > current_ids.txt`
2. **Compare Against API**: `cat /tmp/all_comments.json | jq -r '.id' | sort > all_ids.txt`
3. **Find Missing IDs**: `comm -23 all_ids.txt current_ids.txt`
4. **Analyze Missing Content**: `cat /tmp/all_comments.json | jq 'select(.id == [MISSING_ID])'`
5. **Categorize Missing Items**: Editorial suggestions vs substantive questions vs scope concerns
6. **Add Missing Responses**: Update review document with additional sections

**Missing Comment Analysis Criteria**:
- **Include**: Direct questions, clarification requests, technical concerns, scope feedback
- **Exclude**: Pure editorial suggestions (```suggestion blocks without questions)
- **Include**: Comments with "Why?", "What?", "How?", "I don't think...", "Can we...?"
- **Include**: Comments requesting removal/deferral with substantive reasoning

### Quality Assurance
- GitHub API commands work correctly - issue is usually incomplete analysis, not missing data
- Prioritize question-type comments requiring specific technical responses
- Verify all comment IDs mentioned in analysis exist in GitHub
- Ensure review document includes ALL substantive reviewer concerns (typically 20+ sections for comprehensive reviews)
- Test random sample of GitHub discussion links to confirm accuracy
- Perform gap analysis if initial review seems incomplete compared to total comment count
