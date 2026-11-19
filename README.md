# PlatformIO Library Release

A reusable GitHub Actions workflow that automates publishing PlatformIO libraries to the PlatformIO Registry and creating corresponding GitHub Releases.

## Features

- Reusable workflow for multiple repositories.
- Automatic library version validation.
- Automatic library packaging.
- Publishing to the PlatformIO Registry.
- Automatic GitHub Release creation.
- Repository-level secret management.
- No GitHub Environments required.

## Requirements

The calling repository must contain:

```text
library.json
library.properties
```

Both files must specify the same library version.

Example:

**library.json**

```json
{
  "name": "MyLibrary",
  "version": "1.0.0"
}
```

**library.properties**

```ini
name=MyLibrary
version=1.0.0
```

## Setup

### 1. Configure the PlatformIO Token

In each repository that uses this workflow:

1. Open **Settings**.
2. Navigate to **Secrets and variables → Actions**.
3. Select **New repository secret**.
4. Create the following secret:

| Name | Description |
|---|---|
| `PLATFORMIO_AUTH_TOKEN` | PlatformIO Registry authentication token. |

No GitHub Environment is required.

### 2. Add the Calling Workflow

Create:

`.github/workflows/release.yml`

```yaml
name: Release Library

on:
  push:
    branches:
      - main
    paths-ignore:
      - ".github/workflows/**"
      - "README.md"
      - "docs/**"

  workflow_dispatch:

permissions:
  contents: write

jobs:
  release:
    uses: Pooria-nek/github-actions/.github/workflows/platformio-release.yml@main
    secrets:
      PLATFORMIO_AUTH_TOKEN: ${{ secrets.PLATFORMIO_AUTH_TOKEN }}
```

### 3. Configure Repository Permissions

The calling workflow requires:

```yaml
permissions:
  contents: write
```

This allows the workflow to create GitHub Releases and tags.

## Workflow Execution

The workflow performs these steps:

1. Checks out the calling repository.
2. Sets up Python.
3. Installs PlatformIO CLI.
4. Reads the version from `library.json`.
5. Reads the version from `library.properties`.
6. Checks that both versions match.
7. Packages the library using `pio pkg pack`.
8. Publishes the package to PlatformIO Registry.
9. Creates a GitHub Release.

## Release Tags

GitHub Release tags are generated automatically from the library version.

For example:

| Library version | GitHub tag |
|---|---|
| `1.0.0` | `v1.0.0` |
| `1.1.0` | `v1.1.0` |
| `2.0.0` | `v2.0.0` |

Version numbers must be updated manually in both library metadata files before publishing.

## Triggering Releases

There are two ways to trigger the workflow:

**Automatic**

Push changes to the `main` branch. Changes matching the ignored paths will not trigger a release.

**Manual**

Navigate to:

`GitHub → Actions → Release Library → Run workflow`

## Secret Management

The reusable workflow receives the PlatformIO authentication token from the calling repository.

```yaml
secrets:
  PLATFORMIO_AUTH_TOKEN: ${{ secrets.PLATFORMIO_AUTH_TOKEN }}
```

The reusable workflow accesses it using:

```yaml
${{ secrets.PLATFORMIO_AUTH_TOKEN }}
```

**Important:** Repository-level secrets belong to the calling repository. Each repository using this reusable workflow must configure its own `PLATFORMIO_AUTH_TOKEN`.

## Error Handling

The workflow stops if:

- Either library metadata file is missing.
- The versions do not match.
- The PlatformIO token is missing or invalid.
- Packaging fails.
- Publishing fails.
- GitHub Release creation fails.

## Repository

Maintained in:

[Pooria-nek/github-actions](https://github.com/Pooria-nek/github-actions)

Reusable workflow path:

`.github/workflows/platformio-release.yml`

## Current Limitations

- Version numbers are not automatically incremented.
- Existing GitHub Release tags are not checked before release creation.
- The workflow publishes to PlatformIO before creating the GitHub Release. If release creation fails, PlatformIO may already have received the package.
