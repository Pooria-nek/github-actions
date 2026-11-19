# PlatformIO Library Release

A reusable GitHub Actions workflow for automatically packaging and publishing PlatformIO libraries to the [PlatformIO Registry](https://registry.platformio.org/) and creating GitHub Releases.

## Features

- Reusable workflow that can be called from multiple repositories.
- Automatically checks library version consistency.
- Packages the library using PlatformIO CLI.
- Publishes the package to the PlatformIO Registry.
- Creates a GitHub Release with automatically generated release notes.
- Uses GitHub Environments to manage publishing credentials.

## How It Works

The workflow performs the following steps:

1. Checks out the calling repository.
2. Sets up Python 3.
3. Installs the PlatformIO CLI.
4. Reads the library version from `library.json` and `library.properties`.
5. Verifies that both version values match.
6. Packages the library into a `.tar.gz` archive.
7. Publishes the package to the PlatformIO Registry.
8. Creates a GitHub Release using the version as its tag.

The generated release tag follows this format:

```text
v<version>
```

For example, version `1.2.0` produces the tag `v1.2.0`.

## Requirements

The calling repository must be a valid PlatformIO library containing both:

```text
library.json
library.properties
```

The version declared in both files must be identical.

Example:

**library.json**

```json
{
  "name": "MyLibrary",
  "version": "1.2.0"
}
```

**library.properties**

```ini
name=MyLibrary
version=1.2.0
```

## Configuration

### 1. Create a GitHub Environment

In the repository that calls this workflow:

1. Open **Settings**.
2. Navigate to **Environments**.
3. Create an environment named:

   `platform io`

4. Add the following environment secret:

| Secret | Description |
|---|---|
| `PLATFORMIO_AUTH_TOKEN` | Authentication token for publishing packages to PlatformIO Registry. |

The environment name must match the `environment` value in the reusable workflow.

### 2. Configure the Calling Repository

Create a workflow file in the library repository:

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
```

Replace the repository and workflow path if your reusable workflow is stored elsewhere.

If you use a different branch or tag for the reusable workflow, replace `@main` accordingly.

### 3. Pass the Secret to the Reusable Workflow

The calling workflow must pass the PlatformIO token:

```yaml
jobs:
  release:
    uses: Pooria-nek/github-actions/.github/workflows/platformio-release.yml@main
    secrets:
      PLATFORMIO_AUTH_TOKEN: ${{ secrets.PLATFORMIO_AUTH_TOKEN }}
```

Alternatively, you can use:

```yaml
jobs:
  release:
    uses: Pooria-nek/github-actions/.github/workflows/platformio-release.yml@main
    secrets: inherit
```

`secrets: inherit` passes the secrets available to the caller to the reusable workflow.

**Important:** The secret is read from the calling repository's context, not automatically from the repository where the reusable workflow is stored. Each calling repository needs access to its own configured secret.

## Permissions

The workflow requires:

| Permission | Purpose |
|---|---|
| `contents: write` | Create GitHub Releases and tags. |

The `GITHUB_TOKEN` is used for GitHub Release creation. The PlatformIO token is used only for publishing to the PlatformIO Registry.

## Release Process

When a commit is pushed to the `main` branch, the workflow starts automatically, unless the push only changes files excluded by `paths-ignore`.

You can also trigger it manually through:

**Actions → Release Library → Run workflow**

The workflow will fail if:

- `library.json` is missing.
- `library.properties` is missing.
- The versions in the two files do not match.
- The PlatformIO authentication token is missing or invalid.
- Packaging or publishing fails.
- GitHub Release creation fails.

## Important Notes

- Update the version in both `library.json` and `library.properties` before publishing a new release.
- The workflow does not automatically increment the library version.
- The workflow does not currently check whether the version has already been released.
- If a GitHub Release with the same tag already exists, release creation may fail.
- The workflow runs in the context of the calling repository. Its `platform io` environment must therefore be configured in that repository.
- The reusable workflow can be maintained centrally and called by multiple repositories.

## Workflow Location

This workflow is maintained in:

[Pooria-nek/github-actions](https://github.com/Pooria-nek/github-actions)

Reusable workflow:

`.github/workflows/platformio-release.yml`

## License

Use and modify this workflow according to the license of the repository containing it.
