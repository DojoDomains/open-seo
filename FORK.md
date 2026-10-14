# Fork notes

This fork tracks the upstream OpenSEO repo (`upstream`) and deploys to our own Cloudflare account.
Keep fork-only notes in this file. Don't edit upstream's docs: every edit there conflicts on sync.

## What the fork adds

- `CUSTOM_DOMAIN` in `.env.selfhost`: serves the app from our own hostname instead of workers.dev
  and points the Cloudflare Access application at it (`deploy/alchemy/alchemy.run.ts`).
  The hostname's zone must be in the same Cloudflare account. Each deploy detaches custom domains
  it wasn't asked for, so manage the domain only through this setting.
  With Search Console, add `https://<CUSTOM_DOMAIN>/api/gsc/oauth/callback` to the Google OAuth
  client's redirect URIs.
- `.env.selfhost` sets `AUTH_MODE=cloudflare_access` (Cloudflare Access, not password accounts).

## Sync with upstream and deploy

On the host (commits are GPG-signed; the key is only there):

```bash
git fetch upstream && git rebase upstream/main && git push --force-with-lease origin main
```

In the dev container:

```bash
pnpm install && pnpm deploy:selfhost --yes
```

Then check `https://<CUSTOM_DOMAIN>/api/health`.

## If the deploy fails with "Could not read the Zero Trust organization: Unauthorized"

The alchemy login lacks the `access:write` scope. The login lives in `~/.alchemy`, bind-mounted
from the host so it survives container rebuilds; this only recurs if that folder is lost.

1. `pnpm alchemy login deploy/alchemy/alchemy.run.ts --configure`, choose OAuth, answer yes to
   "Customize OAuth scopes?" and tick `access:write`.
2. The browser callback goes to `localhost:9976` on the host, not the container. Copy the full
   URL from the "can't connect" page and run `curl '<that URL>'` in a second container shell
   within 5 minutes.
