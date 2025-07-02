# Dependabot Release Action

[![github stars](https://badgen.net/github/stars/phucbm/dependabot-release-action?icon=github)](https://github.com/phucbm/dependabot-release-action/)
[![github license](https://badgen.net/github/license/phucbm/dependabot-release-action?icon=github)](https://github.com/phucbm/dependabot-release-action/blob/main/LICENSE)
[![Made in Vietnam](https://raw.githubusercontent.com/webuild-community/badge/master/svg/made.svg)](https://webuild.community)

A GitHub Action that automatically creates releases when Dependabot merges PRs to the main branch. Perfect for automating dependency update releases without manual intervention.

## Features
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   DETECT    │───▶│   VERSION   │───▶│   RELEASE   │───▶│   NOTIFY    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
   Dependabot push     Calculate new       Create Git tag      Generate notes
   Filter by actor     version from        Create GitHub       Professional
   Exit if not bot     current + bump      release draft       release page
```

## Quick Start

1. **Create Workflow File**
   Create `.github/workflows/auto-release.yml`:

```yaml
name: Auto Release

on:
  push:
    branches: [main]

permissions:
  contents: write  # To create tags and releases

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Dependabot Release
        uses: phucbm/dependabot-release-action@v1
        with:
          version-bump: 'patch'  # Always patch for dependency updates
```

2. **That's it!** 🎉
   - When Dependabot merges PRs, releases are automatically created
   - Each release gets a patch version bump (1.0.0 → 1.0.1)
   - Release notes are auto-generated from commit messages
   - Perfect for triggering publish workflows

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version-bump` | Version bump type (patch, minor, major) | ❌ No | `patch` |
| `github-token` | GitHub token for creating releases | ❌ No | `${{ github.token }}` |
| `create-tag` | Create git tag for the release | ❌ No | `true` |
| `release-notes` | Generate release notes from commits | ❌ No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The new version that was released |
| `tag` | The git tag created |
| `release-url` | URL of the created release |

## Usage Examples

**Basic setup** (patch bumps only):
```yaml
- name: Dependabot Release
  uses: phucbm/dependabot-release-action@v1
```

**Custom version bumping:**
```yaml
- name: Dependabot Release
  uses: phucbm/dependabot-release-action@v1
  with:
    version-bump: 'minor'  # For minor dependency updates
```

**Minimal releases** (no auto-generated notes):
```yaml
- name: Dependabot Release
  uses: phucbm/dependabot-release-action@v1
  with:
    release-notes: 'false'
```

## Advanced Usage

**Using outputs for notifications:**
```yaml
- name: Dependabot Release
  id: release
  uses: phucbm/dependabot-release-action@v1

- name: Notify Success
  if: steps.release.outputs.version
  run: |
    echo "Released version ${{ steps.release.outputs.version }}"
    echo "Release URL: ${{ steps.release.outputs.release-url }}"
```

**Custom GitHub token:**
```yaml
- name: Dependabot Release
  uses: phucbm/dependabot-release-action@v1
  with:
    github-token: ${{ secrets.CUSTOM_GITHUB_TOKEN }}
```

## How It Works

```
🎯 DETECTION PHASE
   └── Check if push was made by dependabot[bot]
   └── Exit early if not a Dependabot push
   └── Ensure we only run for dependency updates

📦 VERSION CALCULATION
   └── Read current version from package.json
   └── Calculate new version based on bump type
   └── Support semantic versioning (major.minor.patch)
   └── Handle edge cases and invalid versions

🏷️ RELEASE CREATION
   └── Create annotated Git tag with new version
   └── Push tag to repository
   └── Generate release notes from recent commits
   └── Create GitHub release with professional formatting

✨ NOTIFICATION
   └── Provide outputs for downstream workflows
   └── Log comprehensive summary
   └── Ready to trigger publish workflows
```

## Integration with Publishing

This action is designed to work seamlessly with publishing workflows:

**Complete automation pipeline:**
```yaml
# 1. Dependabot Release (this action)
name: Dependabot Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/dependabot-release-action@v1

# 2. Auto Publish (separate workflow)
name: Publish Package
on:
  release:
    types: [published]
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/publish-npm-action@v1
        with:
          npm-token: ${{ secrets.NPM_TOKEN }}
```

## Example Release Output

**Auto-generated release notes:**
```
🤖 Automated release triggered by Dependabot dependency updates.

## What's Changed
- chore(deps-dev): bump @types/node from 18.0.0 to 18.1.0
- chore(deps): bump express from 4.18.0 to 4.18.1

## Details
- 📦 Version bump: patch
- 🤖 Triggered by: dependabot[bot]
- ℹ️ Note: Package.json will be updated by the publish workflow

---
Automated by Dependabot Release Action by @phucbm
```

## Version Bump Types

| Bump Type | Current | New | Use Case |
|-----------|---------|-----|----------|
| `patch` | 1.0.0 | 1.0.1 | Bug fixes, dependency updates |
| `minor` | 1.0.0 | 1.1.0 | New features, minor updates |
| `major` | 1.0.0 | 2.0.0 | Breaking changes |

**Recommendation**: Use `patch` for Dependabot updates since they typically don't introduce breaking changes.

## Workflow Integration

**Perfect for Dependabot automation:**
```yaml
# .github/workflows/dependabot-automation.yml
name: Dependabot Automation

on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  # Step 1: Test PRs
  test:
    if: github.event_name == 'pull_request' || contains(github.event.comment.body, '/test')
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/test-pr-action@v1
        with:
          dependabot-auto-merge: 'true'

  # Step 2: Dependabot Release after merge
  release:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/dependabot-release-action@v1
```

## Requirements

- Repository must have a `package.json` file for version detection
- Workflow must have `contents: write` permission for creating tags and releases
- Works best with semantic versioning (e.g., `1.2.3`)

## Troubleshooting

**Action doesn't trigger**
- Ensure the push was made by `dependabot[bot]`
- Check that workflow has `contents: write` permission
- Verify the action runs only on push to main branch

**Invalid version errors**
- Ensure `package.json` has a valid semantic version
- Check that version follows `major.minor.patch` format

**Tag creation fails**
- Verify GitHub token has permission to push tags
- Check if tag already exists (action won't overwrite)

**Release creation fails**
- Ensure workflow has `contents: write` permission
- Check GitHub token permissions for repository

## Best Practices

1. **Use with Dependabot** for automatic dependency management
2. **Combine with test workflows** to ensure quality before release
3. **Set up publish workflows** to trigger on release creation
4. **Monitor release notes** for dependency update summaries
5. **Use patch bumps** for dependency updates (safe default)

## Important Notes

- **Package.json versioning**: This action only creates tags and releases. It does NOT update `package.json` - leave that for your publish workflow to avoid conflicts.
- **Dependabot-only**: Only triggers for pushes made by `dependabot[bot]` to prevent unintended releases.
- **Semantic versioning**: Assumes your project follows semantic versioning practices.

## License

MIT License - feel free to use in your projects!

## Contributing

Issues and pull requests welcome! This action is designed to be simple, reliable, and focused on Dependabot automation.
