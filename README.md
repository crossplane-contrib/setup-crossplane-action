# Setup Crossplane

Installs the [Crossplane CLI](https://docs.crossplane.io/latest/cli/) and adds it
to the `PATH`. The binary is downloaded straight from the official release
bucket, checked against a SHA-256 digest, and optionally cached for later runs.

## Usage

```yaml
- uses: crossplane-contrib/setup-crossplane-action@<commit-sha> # v1.0.0
  with:
    version: v2.5.0
```

Pin the action to a full commit SHA rather than a branch or tag. Dependabot and
Renovate can keep the SHA up to date for you.

### Pinning the binary's checksum

By default the binary is verified against the `.sha256` file published next to
it. That catches corrupted or truncated downloads, but not a compromised
release bucket, since the checksum comes from the same place. For stronger
guarantees, pin the digest yourself:

```yaml
- uses: crossplane-contrib/setup-crossplane-action@<commit-sha> # v1.0.0
  with:
    version: v2.5.0
    # From https://cli.crossplane.io/stable/v2.5.0/bin/linux_amd64/crossplane.sha256
    checksum: 065dd6b40c9ea5fc71e49900d5c924cc0bec7f58375922ddbde6b402f292347e
```

The checksum is per platform, so for jobs that run on several runners, set it
per matrix entry:

```yaml
strategy:
  matrix:
    include:
      - runner: ubuntu-latest
        checksum: 065dd6b40c9ea5fc71e49900d5c924cc0bec7f58375922ddbde6b402f292347e # linux_amd64
      - runner: macos-latest
        checksum: 6e0e00370fce82299b931fabee8559eed8a79446172ceb2c24c6d1d76f76fb8c # darwin_arm64
runs-on: ${{ matrix.runner }}
steps:
  - uses: crossplane-contrib/setup-crossplane-action@<commit-sha> # v1.0.0
    with:
      version: v2.5.0
      checksum: ${{ matrix.checksum }}
```

## Inputs

| Name | Default | Description |
|---|---|---|
| `version` | `v2.5.0` | Exact CLI version to install, e.g. `v2.5.0`. `current` and `latest` are rejected. |
| `channel` | `stable` | Release channel, e.g. `stable` or `master`. |
| `checksum` | `""` | SHA-256 of the binary for this runner's platform. When empty, the upstream `.sha256` file is used. |
| `cache` | `true` | Cache the binary with `actions/cache`. |
| `cache-prefix` | `crossplane` | Prefix for the cache key. |

## Outputs

| Name | Description |
|---|---|
| `version` | Version reported by the installed binary. |
| `path` | Absolute path to the installed `crossplane` binary. |
| `cache-hit` | Whether the binary was restored from the cache. |
| `downloaded` | Whether the binary was downloaded during this run. |

## Supported runners

Linux (x64, ARM64, ARM), macOS (x64, ARM64) and Windows (x64). On Windows the
action uses Git Bash, which is preinstalled on GitHub-hosted runners.

## How it works

1. Works out the download URL from the version and the runner's OS and
   architecture. Versions before v2.3.0 come from `releases.crossplane.io`
   (published as `crank`); later versions come from `cli.crossplane.io`.
2. Restores the binary from the cache if possible, otherwise downloads it over
   HTTPS into a staging directory under `$RUNNER_TEMP`.
3. Verifies the SHA-256 of the binary. This also happens on cache hits. A binary
   that fails verification is deleted and the step fails.
4. Runs `crossplane version --client` and checks it reports the requested version.
5. Prepends the install directory to the `PATH`, so it takes precedence over any
   other `crossplane` on the runner.

Nothing is written to the workspace.

## Upgrading from earlier revisions

- The binary now lives under `$RUNNER_TEMP` instead of `$GITHUB_WORKSPACE/bin`.
  Use the `path` output if you need its location.
- `version: current` is no longer accepted. Specify an exact version.
- The install script from `crossplane/crossplane` is no longer used.
