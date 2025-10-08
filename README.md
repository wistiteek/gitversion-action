# GitVersion Calculator Action

GitHub Action to calculate semantic version using [GitVersion](https://gitversion.net/) and optionally update `package.json`.

## Features

- 🏷️ Automatic semantic versioning based on Git history
- 📝 Optional `package.json` version update
- 📤 Artifact upload of updated `package.json`
- 🔧 Configurable GitVersion settings
- 📊 Detailed version summary output

## Usage

### Basic Usage

```yaml
- name: Calculate Version
  uses: wistiteek/gitversion-action@v1
```

### With Custom Configuration

```yaml
- name: Calculate Version
  id: version
  uses: wistiteek/gitversion-action@v1
  with:
    config-file: 'GitVersion.yml'
    update-package-json: 'true'
    upload-artifact: 'true'

- name: Use Version
  run: |
    echo "Version: ${{ steps.version.outputs.version }}"
    echo "Major: ${{ steps.version.outputs.major }}"
    echo "Minor: ${{ steps.version.outputs.minor }}"
    echo "Patch: ${{ steps.version.outputs.patch }}"
```

### In a Reusable Workflow

```yaml
jobs:
  version:
    name: Calculate Version
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.gitversion.outputs.version }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Calculate Version
        id: gitversion
        uses: wistiteek/gitversion-action@v1

  build:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - name: Download updated package.json
        uses: actions/download-artifact@v4
        with:
          name: ${{ needs.version.outputs.package_json_artifact }}
          path: .

      - name: Build with version ${{ needs.version.outputs.version }}
        run: npm run build
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Manual version override | No | _(auto-calculated)_ |
| `config-file` | Path to GitVersion config file | No | `GitVersion.yml` |
| `update-package-json` | Update package.json with calculated version | No | `true` |
| `upload-artifact` | Upload updated package.json as artifact | No | `true` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `version` | Full semantic version | `1.2.3` |
| `major` | Major version number | `1` |
| `minor` | Minor version number | `2` |
| `patch` | Patch version number | `3` |
| `build_number` | Build metadata | `42` |
| `prerelease` | Pre-release tag | `alpha.1` |
| `branch` | Branch name | `main` |
| `sha` | Commit SHA | `abc123...` |
| `package_json_artifact` | Artifact name | `updated-package-json-1.2.3` |

## GitVersion Configuration

This action requires a `GitVersion.yml` file at the root of your repository. Here's a minimal example:

```yaml
mode: ContinuousDeployment
branches:
  main:
    tag: ''
  develop:
    tag: 'beta'
  feature:
    tag: 'alpha'
```

For more configuration options, see the [GitVersion documentation](https://gitversion.net/docs/).

## Requirements

- Git repository with full history (use `fetch-depth: 0` in checkout)
- GitVersion.yml configuration file
- (Optional) package.json for version updates

## Example Workflow

```yaml
name: Build and Release

on:
  push:
    branches: [main]

jobs:
  version:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.calc-version.outputs.version }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Calculate Version
        id: calc-version
        uses: wistiteek/gitversion-action@v1

  build:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download versioned package.json
        uses: actions/download-artifact@v4
        with:
          name: updated-package-json-${{ needs.version.outputs.version }}
          path: .

      - name: Build
        run: npm run build

      - name: Create Release
        run: |
          echo "Creating release for version ${{ needs.version.outputs.version }}"
```

## License

MIT

## Author

[wistiteek](https://github.com/wistiteek)
