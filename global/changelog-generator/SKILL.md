---
name: changelog-generator
description: Use when preparing release notes, creating changelogs, or writing customer-facing update summaries from git commits. Transforms technical commit history into polished release documentation.
metadata:
  author: DDD
  version: 1.0.0
  category: documentation
---

# Changelog Generator

Transforms technical git commits into polished, user-friendly changelogs. Analyzes commit history, categorizes changes, filters internal noise, and outputs professional release documentation.

## When to Use

- Preparing release notes for a new version
- Creating weekly/monthly product update summaries
- Writing changelog entries for app store submissions
- Generating customer-facing update notifications
- Maintaining a public changelog or product updates page

## How to Use

From your project repository:

```
Create a changelog from commits since last release
```

```
Generate changelog for commits from the past week
```

```
Create release notes for version 2.5.0, using guidelines from CHANGELOG_STYLE.md
```

## Example

**Request**: "Create a changelog for commits from the past 7 days"

**Output**:
```markdown
# Updates - Week of March 10, 2024

## New Features
- **Team Workspaces**: Create separate workspaces for different
  projects. Invite team members and keep everything organized.

## Improvements
- **Faster Sync**: Files now sync 2x faster across devices
- **Better Search**: Search now includes file contents, not just titles

## Fixes
- Fixed issue where large images wouldn't upload
- Resolved timezone confusion in scheduled posts
```

See `references/workflow.md` for detailed categorization rules and formatting options.
