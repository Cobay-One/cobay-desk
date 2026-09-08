# cobay-desk

Deployment configuration for the self-hosted Chatwoot instance ("Cobay Desk").

This repo does **not** contain Chatwoot's source code — the application runs from
the prebuilt `chatwoot/chatwoot` image published by upstream. What lives here is
how *our* instance is configured, sized, and branded.

## Layout

| Path | Purpose |
|---|---|
| `docker-compose.yaml` | The stack: rails, sidekiq, postgres (pgvector), redis |
| `branding/` | Logos, theme CSS, and a patched `vueapp.html.erb` layout |
| `.env.example` | Every variable the stack reads, with values stripped |

## Secrets

`.env` holds real credentials and is **gitignored** — it lives only on the server
at `~/chatwoot/.env`. `.env.example` mirrors its structure so a rebuild knows
what to fill in. Never commit the real file.

## Deploying a change

    cd ~/chatwoot
    git pull
    docker compose up -d

`git pull` updates the config; `docker compose up -d` applies it by recreating
only the containers whose definition changed. Named volumes (`chatwoot_pgdata`,
`chatwoot_redisdata`, `chatwoot_storage`) are untouched, so data survives.

## Notes

- The compose file pins `name: chatwoot`. Do not remove it — volume names derive
  from the project name, and changing it orphans the database.
- `branding/vueapp.html.erb` is a copy of an upstream file, forked at **v4.17.1**,
  bind-mounted over the image's copy. Diff it against the new image on every
  version bump; upstream changes to that layout are silently overridden.
- TLS terminates at nginx on the host (`/etc/nginx/conf.d/`), not in this repo.
