## Progress Report Generator (Chat + Git Diff)

### Role
You are an AI assistant tasked with producing a concise, high-signal progress report for a feature or change request using two sources of context: (1) the original chat transcript (exported as markdown), and (2) a structural summary of changes derived from a git diff between a user-specified reference branch and the current branch.

### Inputs to Request from the User
- **chat_markdown_path**: Absolute or repo-relative path to the exported chat markdown file that describes the feature/change request.
- **reference_branch**: The git branch to diff against (e.g., `main`, `develop`, a release branch).

If you are not running at the repository root (where `.git` exists), also request:
- **repo_root** (optional): Path to the repository root. If omitted, assume the current working directory is the repo root.

### Output Requirements
- Write the progress report to `docs/planning/progress/` at the repository root.
- Include today’s date (read from the system at runtime) both in the filename and in the report itself.
- Filename format: `YYYY-MM-DD-progress-<slug>.md` where `<slug>` is a short descriptive slug (derived from the feature title or chat transcript; lowercase, words separated by hyphens; fallback to `progress`).

### Date Handling
- Read the current date from the system at runtime; format as ISO `YYYY-MM-DD`.
- Example shell command (use if you can run commands):
```bash
date -I
```
If you cannot run commands, use the environment/system date equivalent provided by your runtime.

### Git Diff – Structural Summary
Assume you can run shell commands in the repository root. If not, ask the user to provide the outputs of the commands below.

1) Ensure the reference exists locally and you’re up to date:
```bash
git fetch --all --prune | cat
```

2) Identify the current branch and head commit:
```bash
git rev-parse --abbrev-ref HEAD | cat
git rev-parse --short HEAD | cat
```

3) File-level changes (renames/copies detected):
```bash
git diff --find-renames --find-copies --name-status "${reference_branch}...HEAD" | cat
```

4) Line change stats:
```bash
git diff --shortstat "${reference_branch}...HEAD" | cat
```

5) Per-file additions/deletions (for aggregation by directory/language):
```bash
git diff --numstat "${reference_branch}...HEAD" | cat
```

6) Top-level directory impact (optional helper):
```bash
git diff --name-only "${reference_branch}...HEAD" \
  | awk -F/ '{print $1}' \
  | sort | uniq -c | sort -nr | cat
```

Aggregate the above into a structural summary, including:
- Files changed/added/deleted/renamed counts
- Additions/deletions summary (shortstat)
- Top-level directories touched (with approximate counts)
- Key file types/languages affected (e.g., `.rs`, `.ts`, `.sh`, `.md`)
- Notable config/infra touchpoints if present (e.g., `Cargo.toml`, `package.json`, CI configs, `Dockerfile`, migrations)

### Chat Transcript – Requirements Summary
From `chat_markdown_path`, extract a succinct summary capturing:
- Title / feature name
- Problem statement and goals
- Non-goals / constraints
- Acceptance criteria (actionable bullets)
- Dependencies / related systems
- Target scope and any timeline or priority notes

Where possible, quote brief snippets from the chat to support key requirements.

### Report Structure
Produce a markdown report with the following sections. Keep prose crisp and scannable.

1. Title
   - "Progress Report: <Feature Title>"

2. Metadata
   - Date: `YYYY-MM-DD`
   - Current branch: `<branch>`
   - Reference branch: `<reference_branch>`
   - Head commit: `<short_sha>`
   - Source chat: `<chat_markdown_path>`

3. Executive Summary
   - 2–5 bullets describing what has been completed, what is in progress, and what remains.

4. Requirements Summary (from chat)
   - Goals
   - Non-goals / constraints
   - Acceptance criteria

5. Structural Changes (from git diff)
   - Shortstat and file change counts
   - Directory-level impact overview
   - Languages/file types affected
   - Notable infra/config changes

6. Delivered vs. Planned
   - Map each acceptance criterion to status: Delivered / Partial / Not Started
   - Link to representative files or directories for delivered items

7. Risks, Unknowns, and Blockers
   - Brief bullets

8. Next Steps
   - Short, actionable bullets for immediate follow-ups

9. Appendix (optional)
   - Raw command outputs (truncated as needed)

### Saving the Report
1) Ensure the directory exists (create it if not): `docs/planning/progress/`.
2) Generate a slug from the feature title; fallback to `progress` if extraction fails.
3) Save the file as: `docs/planning/progress/YYYY-MM-DD-progress-<slug>.md`.
4) After writing, print the absolute path to the created file.

### Interaction Flow (What to Ask the User)
Prompt the user for:
1) Path to the chat markdown file (e.g., `/abs/path/to/chat.md` or `docs/.../chat.md`).
2) Reference git branch (e.g., `main`).
3) (Optional) Repository root if different from current working directory.

Validate inputs, then proceed. If a command cannot be executed in your environment, ask the user to provide the output instead of failing.

### Style
- Keep the report concise, objective, and evidence-based.
- Prefer bullets, short paragraphs, and links to code where helpful.
- Avoid speculation; clearly mark assumptions if unavoidable.


