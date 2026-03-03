# doc-update

Reusable GitHub workflow that automatically updates documentation using Claude Code and doctrace.

## Quick Start

Add this workflow to your repo:

```yaml
# .github/workflows/update-docs.yml
name: Update Docs

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
    inputs:
      git_ref:
        description: 'Git ref for affected detection'
        required: false
        default: 'docs-base'
        type: string

permissions:
  contents: write
  pull-requests: write

jobs:
  update:
    uses: lucasvtiradentes/doc-update/.github/workflows/update-docs.yml@main
    with:
      docs_path: docs/
      git_ref: ${{ inputs.git_ref || 'docs-base' }}
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## Requirements

- Docs must have `sources:` frontmatter metadata

## Inputs

| Input       | Default     | Description                     |
|-------------|-------------|---------------------------------|
| `docs_path` | `docs/`     | Path to documentation directory |
| `git_ref`   | `docs-base` | Git ref for affected detection  |

### git_ref options

- `docs-base` - incremental, since last sync (default)
- `v1.0.0` - since specific tag
- `HEAD~5` - last 5 commits
- `main` - since diverging from main branch

## How It Works

1. Runs `doctrace affected` to detect docs impacted by code changes
2. Spawns Opus subagents (in parallel per phase) to validate/update each doc
3. Compares doc content against source code with conservative editing rules
4. Updates frontmatter metadata (sources/required_docs/related_docs)
5. Generates sync report with confidence levels and gap analysis
6. Creates/updates PR with detailed changelog
7. Updates `docs-base` tag to track incremental state

## Secrets Required

| Secret                    | Description             |
|---------------------------|-------------------------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code OAuth token |

## PR Behavior

- Creates new PR if none exists
- Updates existing PR if one is open (appends run info to body)
- Uses `docs-base` git tag for incremental tracking
- Branch format: `docs/auto-update-YYYYMMDD-HHMMSS`

## Features

- **Parallel execution** - docs within same phase processed concurrently
- **Conservative edits** - only fixes factual errors, no rewording
- **Gap analysis** - identifies undocumented changes
- **Confidence tracking** - low confidence triggers re-validation
- **Detailed PR body** - commits, changes per doc, action items
