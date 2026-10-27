# Aedis Conformance

GitHub Action that checks a repository against the [Aedis Platform](https://github.com/aedis-build/aedis-platform)
conformance levels, **L1 Baseline, L2 Hardened and L3 Regulated**: implemented, verifiable controls, not a
certification. It runs `aedis platform validate --level N`, which renders and validates a GitOps repository
(`apps/**`, three value layers, schema, kubeconform) and checks the platform controls the CLI can verify locally
(`PLT-xx` in [levels.yaml](https://github.com/aedis-build/aedis-platform/blob/main/conformance/levels.yaml): pod
baseline, secrets via External Secrets, probes, PDB, image by digest in prod, egress policy, WAF, prod only by PR,
platform workflows pinned on a release tag). A repository without `apps/` (a library, an action) answers only for the
repository controls. With a token it also runs `aedis org audit` for the repository in its organization: OpenSSF Best
Practices criteria by tier, live branch protection, Scorecard minimums and release evidence, and fails when the level
is not reached. The summaries are uploaded as the `conformance-summary` artifact.

## Usage

```yaml
jobs:
  conformance:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false
      - uses: aedis-build/aedis-conformance@<sha of the release tag> # vX.Y.Z
        with:
          level: 2
```

With the organization audit (the token of the Aedis Compliance GitHub App, or any token that reads the repository's
settings, rulesets, releases and workflow runs):

```yaml
      - uses: actions/create-github-app-token@64ab729a6294000f5aa4a4bc0b73d1f32bc45510 # v3.3.0
        id: app
        with:
          app-id: ${{ secrets.AEDIS_APP_ID }}
          private-key: ${{ secrets.AEDIS_APP_PRIVATE_KEY }}
      - uses: aedis-build/aedis-conformance@<sha of the release tag> # vX.Y.Z
        with:
          level: 3
          token: ${{ steps.app.outputs.token }}
```

Pin the action by the commit SHA of a release tag with the tag in the comment, as above: Dependabot
(`package-ecosystem: github-actions`) then opens the repin pull request at every release, SHA and comment together.

### Inputs

| Input | Default | Meaning |
|---|---|---|
| `level` | `3` | Conformance level to check: 1, 2 or 3 |
| `path` | `.` | Root of the repository to validate; its `conformance/levels.yaml`, when present, raises the thresholds (never lowers them) |
| `chart` | `""` | Chart for the GitOps manifests: a directory or `oci://...`; empty uses `charts/aedis-app` of the platform checkout |
| `chart-version` | `""` | Chart version when `chart` is an OCI reference |
| `kubernetes-version` | `1.31.0` | Kubernetes version kubeconform validates against |
| `out` | `aedis-conformance` | Directory for the rendered manifests, `validate-summary.json` and the audit report |
| `token` | `""` | Token for `aedis org audit` of this repository; empty skips the audit |
| `platform-ref` | pinned in `action.yml` | Commit of `aedis-build/aedis-platform` the CLI is built from |

### Outputs

| Output | Meaning |
|---|---|
| `result` | `pass` or `fail`: the level is reached when no control fails |

Controls that depend on the CI summaries or the cluster (Scorecard, coverage, SAST, DAST, signed image, Kyverno) are
reported as not verifiable locally by `validate`; the audit, with a token, reads them from the organization.

## How the CLI gets there

The CLI is not on nuget.org yet, so the action checks out `aedis-build/aedis-platform` at the pinned commit and builds
it with `dotnet run` (the .NET 10 SDK is set up by the action; about a minute). helm and kubeconform are installed only
when the repository has `apps/`. Once `Aedis.Cli` is published, the action switches to `dotnet tool install`.

## License

Apache-2.0. See [LICENSE](LICENSE) and [SECURITY.md](SECURITY.md).
