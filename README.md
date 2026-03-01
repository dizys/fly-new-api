# fly-new-api

Deploy [new-api](https://www.newapi.ai/) to [Fly.io](https://fly.io) with automated GitHub Actions deployment.

new-api is an OpenAI-compatible API gateway that supports multiple AI providers. This setup uses SQLite for storage via a Fly.io persistent volume.

## Prerequisites

- [flyctl](https://fly.io/docs/hands-on/install-flyctl/) installed and authenticated
- A [Fly.io](https://fly.io) account

## Setup

### 1. Create the Fly.io app

```bash
fly apps create your-app-name
```

Update `app` in `fly.toml` to match your app name.

### 2. Create the persistent volume

```bash
fly volumes create new_api_data --size 1 --region sjc
```

Adjust `--size` (GB) and `--region` as needed. The region should match `primary_region` in `fly.toml`.

### 3. Set secrets

new-api requires an initial admin token:

```bash
fly secrets set INITIAL_ROOT_TOKEN=your-secret-token
```

Set any other environment variables you need (e.g. session secret):

```bash
fly secrets set SESSION_SECRET=your-session-secret
```

### 4. Deploy

```bash
fly deploy
```

After the first deploy, open your app:

```bash
fly open
```

Log in at `https://your-app-name.fly.dev` — the default admin credentials are `root` / `123456` unless overridden by `INITIAL_ROOT_TOKEN`.

## GitHub Actions (automated deploy)

Every push to `main` triggers a deploy. To enable this:

1. Generate a Fly.io API token:
   ```bash
   fly tokens create deploy -x 999999h
   ```
2. Add it as a repository secret named `FLY_API_TOKEN` in your GitHub repo settings
   (`Settings → Secrets and variables → Actions → New repository secret`).

## Configuration

| File | Purpose |
|------|---------|
| `Dockerfile` | Wraps `calciumion/new-api:latest` for Fly.io builds |
| `fly.toml` | Fly.io app configuration (region, VM size, volume mount, HTTP settings) |
| `.github/workflows/deploy.yml` | CI/CD workflow — deploys on push to `main` |

### Changing region

Update `primary_region` in `fly.toml` and re-create the volume in the same region.

### Scaling memory

If new-api needs more memory, update `memory_mb` in `fly.toml`:

```toml
[[vm]]
  memory_mb = 1024
```
