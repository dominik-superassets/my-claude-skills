# Changelog Generator Workflow

## Step-by-Step Process

### 1. Scan Git History

Analyze commits from a specific time period or between versions:

```bash
# By date range
git log --since="2024-03-01" --until="2024-03-15" --oneline

# By tag/version
git log v2.4.0..HEAD --oneline

# Past N days
git log --since="7 days ago" --oneline
```

### 2. Categorize Changes

Group commits into logical categories:

| Category | Commit Patterns |
|----------|-----------------|
| New Features | `feat:`, `add:`, "new", "introduce" |
| Improvements | `improve:`, `enhance:`, `update:`, "optimize" |
| Bug Fixes | `fix:`, `bugfix:`, "resolve", "correct" |
| Breaking Changes | `BREAKING:`, `!:`, major version bumps |
| Security | `security:`, "CVE", "vulnerability" |

### 3. Filter Noise

Exclude internal-only commits:

- `chore:` - maintenance tasks
- `refactor:` - code restructuring
- `test:` - test additions/changes
- `docs:` (internal) - developer documentation
- `ci:` - CI/CD changes
- `build:` - build system changes

### 4. Translate to User Language

Transform technical commits into customer-friendly descriptions:

| Technical Commit | User-Friendly |
|------------------|---------------|
| `fix: resolve null pointer in auth flow` | Fixed login issues some users experienced |
| `feat: add websocket support for real-time` | Real-time updates now sync instantly |
| `perf: optimize query caching` | Pages now load 2x faster |

### 5. Format Output

Standard changelog structure:

```markdown
# Updates - [Date/Version]

## New Features
- **Feature Name**: Brief description of user benefit

## Improvements
- **Area**: What got better and why users care

## Fixes
- Brief description of what was broken and is now fixed

## Breaking Changes (if any)
- What changed and what users need to do
```

## Output Formats

### Standard Changelog (CHANGELOG.md)
Full format with all categories, dates, and version numbers.

### Release Notes (GitHub/GitLab)
Condensed format suitable for release pages.

### App Store Updates
Brief, marketing-friendly format with character limits in mind.

### Email/Newsletter
Narrative format suitable for customer communications.

## Tips for Best Results

- Run from your git repository root
- Specify date ranges for focused changelogs
- Reference a CHANGELOG_STYLE.md file for consistent formatting
- Review and adjust before publishing
- Consider your audience (technical vs non-technical)
