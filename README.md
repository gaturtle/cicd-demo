# cicd-demo

A minimal static site used to learn a real CI/CD pipeline end to end, at zero cost.

**Live site:** https://gaturtle.github.io/cicd-demo/

## How it works

1. **CI** (`.github/workflows/ci.yml`) — runs on every push and pull request. Installs
   dependencies and lints the HTML with `htmlhint`. This is the safety gate: nothing
   broken should ever reach `main`.
2. **CD** (`.github/workflows/deploy.yml`) — triggers automatically once the CI workflow
   finishes successfully on `main` (`workflow_run`), then publishes the site to
   **GitHub Pages** using GitHub's official Pages actions. No deploy keys or secrets —
   it uses the repo's built-in `GITHUB_TOKEN`.

Cost: **$0**. Public repos get unlimited-for-practical-purposes free GitHub Actions
minutes, and GitHub Pages hosting is free for public repos.

## Local dev

```bash
npm install
npm run lint     # same check CI runs
```

Open `index.html` directly in a browser, or serve it with any static server.

## Extending this to a real VPS later

Once you have a small VPS (e.g. a $4-6/mo box from Hetzner/DigitalOcean/Vultr), the CI
job stays exactly the same — only the CD job changes, from "upload to GitHub Pages" to
"copy files to the server and restart/reload it." Two common patterns, roughly in order
of how much you'll learn from them:

### Option A — `rsync` over SSH (good next step, static or simple apps)

Add a new job to `deploy.yml` (or a separate workflow) that:

1. Uses `webfactory/ssh-agent` (or `appleboy/ssh-action`) with a deploy-only SSH key
   stored as the `SSH_PRIVATE_KEY` repo secret (`gh secret set SSH_PRIVATE_KEY`).
2. Runs `rsync -avz --delete ./ user@your-server:/var/www/cicd-demo/`.
3. If it's a backend app (not static), follows up with an SSH command to restart the
   service, e.g. `ssh user@host 'sudo systemctl restart cicd-demo'`.

Secrets needed: `SSH_PRIVATE_KEY`, `SSH_HOST`, `SSH_USER`.

### Option B — Docker image + `docker compose pull && up -d` (more production-like)

1. CI builds and pushes a Docker image to GitHub Container Registry (`ghcr.io`) — free,
   already authenticated via `GITHUB_TOKEN`, no extra account needed.
2. CD SSHes into the VPS and runs `docker compose pull && docker compose up -d`, so the
   server always just pulls the latest tagged image.

This mirrors how most real small-team production deploys actually work, and is the
natural next step once you're comfortable with Option A.

Either way: **CI never changes.** Only the last step — "where do the passing files go"
— changes from "GitHub Pages action" to "SSH to my own box."
