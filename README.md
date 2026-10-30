# Workers

Shared GitHub Actions for deploying Cloudflare Workers. Every action in
this repository is value-free: it contains no project names, URLs, or
secret names — callers pass all of them as inputs, Marketplace style.

## Actions

### `deploy-worker`

```yaml
- uses: Karasu-Lab/Workers/deploy-worker@v1
  with:
    mode: production # or preview
    hub-repo: owner/hub-repository
    app-path: apps/example
    deploy-command: pnpm --filter example deploy
    preview-command: pnpm --filter example preview -- --name "{branch}"
    worker-name: example-worker
```

Assembles a pnpm workspace by cloning the hub repository (recursive
submodules) and pinning the app submodule to the triggering commit,
installs dependencies, runs the caller's optional prepare/build
commands, then deploys:

- **production** mode runs `deploy-command`, then `sync-command` if
  given.
- **preview** mode runs `preview-command` with the branch name.

Afterwards the worker's Previews are pruned: only the newest
`keep-previews` (default 5) survive, and the current branch's Preview is
never deleted. Pruning uses the Cloudflare API preview collection and
falls back to ordering the app repository's branches by last commit
date; it warns instead of failing the deployment.

Commands accept placeholders: `{branch}`, `{env}`, `{app_path}`,
`{secrets_file}`, `{secret_names}`.

### Environment contract

The caller job provides, as environment variables:

- `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` (used by the
  deploy/preview commands and the pruning step).
- One variable per name listed in `secret-names`, when `secrets-command`
  builds a secrets file from them.

Callers typically map these from their GitHub Environment's secrets and
variables, so the same workflow serves production and preview with
different values.
