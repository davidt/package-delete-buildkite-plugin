# Package Delete Buildkite Plugin

Deletes packages from a [Buildkite package
registry](https://buildkite.com/docs/packages) by inspecting build artifacts
from previous steps. This is useful when using the package registry for
development builds across repositories.

Exits successfully if the package doesn't exist, making it safe for first builds.

## Example

```yaml
steps:
  - label: ":hammer: Build"
    commands:
      - python -m build
    artifact_paths:
      - "dist/*.whl"

  - wait

  - label: ":wastebasket: Delete existing dev package"
    plugins:
      - "https://github.com/davidt/package-delete-buildkite-plugin.git":
          registry: "my-org/my-registry"
          artifacts: "*.whl"

  - wait

  - label: ":rocket: Publish"
    plugins:
      - "publish-to-packages#v2.2.0":
          registry: "my-org/my-registry"
          artifacts: "*.whl"
```

## Configuration

| Option | Required | Description |
|---|---|---|
| `registry` | Yes | Registry slug. Supports `org/registry` or just `registry` (defaults org to `$BUILDKITE_ORGANIZATION_SLUG`). |
| `artifacts` | Yes | Glob pattern for build artifacts to inspect (e.g. `dist/*.whl`). Package name and version are extracted from filenames. |

## Supported formats

| Format | Extension | Filename convention |
|---|---|---|
| Python wheel | `.whl` | `{name}-{version}-{python}-{abi}-{platform}.whl` |

## Authentication

This plugin uses [Buildkite OIDC](https://buildkite.com/docs/agent/v3/cli-oidc) to authenticate with the Packages API. Your registry must have an OIDC policy configured that grants the pipeline permission to manage packages.

## Requirements

- `curl`
- `jq`
- `buildkite-agent`
