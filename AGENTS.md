# Base44 sandbox notes

## Current state of the repository

The `main` branch (commit `15cad3c822c48155633d3eeef2ae4e0f22b7b560`) contains **only**
`LICENSE` and `README.md` — no application source, no `package.json`, no Dockerfile, no
compose file. That is why the app "fails to start": there is nothing to build or serve.

The product described in the README (*Animate-Master*) lives in a Base44 app that is not
yet connected to this repository. This repository gains a working preview only once that
app's code is pushed here.

## Running the sandbox

```bash
docker compose -f docker-compose.base44.yml up -d --build
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:3000/   # expect 200
docker compose -f docker-compose.base44.yml ps
docker compose -f docker-compose.base44.yml logs -f web
```

- Host port **3000** is the only public entry point; it maps to the Vite dev server on
  container port 5173 (`--host 0.0.0.0 --port 5173`).
- Dependencies are installed inside the container at startup, from the mounted
  `package.json`; `node_modules` lives in the named volume `web_node_modules`, so the
  sandbox host needs no Node runtime.
- The source is bind-mounted, so edits are served live. Call `reload_preview` only when a
  change cannot appear through the dev server.

## Sandbox-only overrides

Passed through as bare Compose `environment:` entries, never hardcoded:

- `BASE44_PREVIEW_MODE` — set to `1` by the platform in the sandbox. Any sandbox-only
  behaviour must be gated on an exact comparison with `'1'`; otherwise original
  behaviour applies.
- `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS` — set by the platform; Vite >= 6.1 appends it
  to `server.allowedHosts` so the preview host is accepted. The preview sends
  `Host: 3000-<suffix>` and `Origin: https://3000-<suffix>`.

## What still needs to happen

1. Push the Base44 app's source into this repository (from Base44, connect the app to
   `mdcblancom02-cloud/Animaci-n-3d`).
2. If the pushed app is not a Vite project, update `docker-compose.base44.yml`: keep the
   source bind-mounted, keep a live-reload dev command, keep host port 3000, keep the
   healthcheck.
3. Declare any credential the app needs on boot with `set_secrets` /
   `generate_development_secrets` and record it in `.base44/environment.json`.
