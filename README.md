# scaffolding-workflows

Meraki-owned GitHub Actions reusable workflows for `app-starter` forks. [`P5.4`]

## What this is

The actual provisioning/promotion logic — six workflows totaling ~1,000 lines — lives
here instead of inside a client's own repository. A fork keeps only a thin caller stub
per workflow: the real trigger (`workflow_dispatch`, `schedule`, `pull_request` +
`workflow_run`) and a `uses:` line pointing back here. Fire Meraki, and the stub is still
there, but the link to this repo either stops resolving or access is revoked — there is
nothing left in the client's repo to excise.

## Workflows

| File                        | Behind app-starter's caller | Purpose                                                              |
| ---------------------------- | ---------------------------- | --------------------------------------------------------------------- |
| `provision-workspace.yml`    | `provision-workspace.yml`    | Stand up a new workspace clone (DB, Cognito, CDK stack)               |
| `deprovision-workspace.yml`  | `deprovision-workspace.yml`  | Tear a workspace down                                                 |
| `redeploy-workspace.yml`     | `redeploy-workspace.yml`     | Rebuild + redeploy a workspace when its branch gets new commits       |
| `revert-promotion.yml`       | `revert-promotion.yml`       | Open a PR on `dev` containing the exact inverse of a promotion commit |
| `sweep-workspaces.yml`       | `sweep-workspaces.yml`       | Auto-retirement sweep for idle/promoted/past-grace workspaces         |
| `ws-automerge.yml`           | `ws-automerge.yml`           | Auto-merge the agent's PR into a `ws/**` branch once CI is green      |

## How a fork consumes these

```yaml
jobs:
  run:
    uses: meraki-digital/scaffolding-workflows/.github/workflows/provision-workspace.yml@v1
    with:
      # ...inputs, unchanged from the workflow_call declaration below...
    secrets: inherit
```

`secrets`/`vars` referenced inside these files resolve against the **calling**
repository, not this one — that's how GitHub Actions reusable workflows work. Nothing
here needs its own copy of a fork's secrets.

## Versioning

References are pinned by tag (`@v1`), not `@main`. A change here does not reach any
caller until something deliberately bumps the tag in the caller's own repo — a one-line,
reviewable diff, the same shape as depending on a published version of
`@meraki-digital/workspace-cdk` [`P5.3`].

## Provenance

Extracted from `app-starter`'s `.github/workflows/_reusable-*.yml`, 2026-08-23. Every
file's `on:`/`jobs:` body is byte-for-byte identical to the app-starter original at the
time of the move — only the filename, the `name:` field, and header prose changed.
