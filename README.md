# Tag and Release Semantic Version

A GitHub Action that automates semantic versioning for your repository by creating tags and releases with support for parent version tag updates.

## Features

- Automatic semantic version tag creation (v1.2.3)
- Optional parent version tag management (v1, v1.2)
- GitHub release generation with automatic changelog
- Dry run mode for testing
- Validation to prevent duplicate tags
- Comprehensive outputs for downstream workflow steps

## Usage

### Basic Example

```yaml
name: Release
on:
  workflow_dispatch:
    inputs:
      version_bump:
        description: 'Version bump type'
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: chase-roohms/tag-and-release@v1
        with:
          version_bump: ${{ inputs.version_bump }}
```

### With Parent Tag Updates

```yaml
- uses: chase-roohms/tag-and-release@v1
  with:
    version_bump: patch
    update_parent_tags: true
    create_release: true
```

### Dry Run Mode

```yaml
- uses: chase-roohms/tag-and-release@v1
  with:
    version_bump: minor
    dry_run: true
```

### Using Outputs

```yaml
- id: release
  uses: chase-roohms/tag-and-release@v1
  with:
    version_bump: minor

- name: Use version outputs
  run: |
    echo "New version: ${{ steps.release.outputs.new_version }}"
    echo "Previous version: ${{ steps.release.outputs.previous_version }}"
    echo "Tags updated: ${{ steps.release.outputs.tags_updated }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token with contents write access | No | `${{ github.token }}` |
| `version_bump` | Version bump type: `major`, `minor`, or `patch` | Yes | - |
| `dry_run` | Calculate version but don't create tags | No | `false` |
| `create_release` | Create a GitHub release in addition to tags | No | `true` |
| `update_parent_tags` | Update parent version tags | No | `false` |

### Version Bump Types

- `major`: Increments the major version (1.0.0 → 2.0.0)
- `minor`: Increments the minor version (1.0.0 → 1.1.0)
- `patch`: Increments the patch version (1.0.0 → 1.0.1)

### Parent Tag Updates

When `update_parent_tags` is enabled:

**Patch Release (v1.2.3):**
- Creates: `v1.2.3`
- Updates: `v1.2` → points to v1.2.3
- Updates: `v1` → points to v1.2.3

**Minor Release (v1.3.0):**
- Creates: `v1.3.0`, `v1.3`
- Updates: `v1` → points to v1.3.0

**Major Release (v2.0.0):**
- Creates: `v2.0.0`, `v2.0`, `v2`

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `new_version` | The new full semantic version created | `v3.5.2` |
| `minor_version` | The minor version tag | `v3.5` |
| `major_version` | The major version tag | `v3` |
| `previous_version` | The previous version before this release | `v3.5.1` |
| `tags_updated` | JSON array of tags that were created or updated | `["v3.5.2", "v3.5", "v3"]` |

## Permissions

This action requires the `contents: write` permission to create tags and releases:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
```

## Requirements

- The repository must be checked out with full git history (`fetch-depth: 0`)
- Git tags must follow the semantic versioning format: `v*.*.*` (e.g., v1.0.0)
- The GitHub token must have write access to repository contents

## How It Works

1. **Version Calculation**: Finds the latest semantic version tag and calculates the next version based on the bump type
2. **Validation**: Checks if the calculated version tag already exists
3. **Tag Creation**: Creates the full semantic version tag (e.g., v1.2.3)
4. **Parent Tag Updates** (optional): Updates or creates parent version tags (v1.2, v1)
5. **Release Creation** (optional): Creates a GitHub release with automatic changelog
6. **Summary Generation**: Provides a detailed summary in the workflow run

## Initial Release

If no semantic version tags exist in the repository, the action will create `v1.0.0` as the initial version regardless of the bump type specified.

## Example Workflows

### Manual Release Workflow

```yaml
name: Create Release
on:
  workflow_dispatch:
    inputs:
      version_bump:
        description: 'Version bump type'
        required: true
        type: choice
        options:
          - patch
          - minor
          - major
      update_parents:
        description: 'Update parent version tags'
        type: boolean
        default: false

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: chase-roohms/tag-and-release@v1
        with:
          version_bump: ${{ inputs.version_bump }}
          update_parent_tags: ${{ inputs.update_parents }}
```

### Automated Release on Main Branch

```yaml
name: Auto Release
on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: chase-roohms/tag-and-release@v1
        with:
          version_bump: patch
          update_parent_tags: true
```

### PR Preview with Dry Run

```yaml
name: Preview Release
on:
  pull_request:
    types: [labeled]

jobs:
  preview:
    if: contains(github.event.pull_request.labels.*.name, 'release')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: chase-roohms/tag-and-release@v1
        with:
          version_bump: patch
          dry_run: true
```

### Multi-Job Workflow with Version Outputs

```yaml
name: Release and Deploy
on:
  workflow_dispatch:
    inputs:
      version_bump:
        type: choice
        options: [patch, minor, major]
        required: true

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    outputs:
      version: ${{ steps.release.outputs.new_version }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - id: release
        uses: chase-roohms/tag-and-release@v1
        with:
          version_bump: ${{ inputs.version_bump }}
          update_parent_tags: true

  deploy:
    needs: release
    runs-on: ubuntu-latest
    steps:
      - name: Deploy version
        run: |
          echo "Deploying version ${{ needs.release.outputs.version }}"
```

## License

This action is provided as-is under the MIT License.

## Contributing

Contributions are welcome. Please open an issue or pull request for bug fixes, improvements, or new features.
