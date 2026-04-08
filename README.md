# bilidown-docker

Docker image for [bilidown](https://github.com/iuroc/bilidown) — a Bilibili video download tool with web UI.

Automatically builds multi-arch images from the latest bilidown release and publishes to GitHub Container Registry.

## Quick Start

```bash
mkdir bilidown && cd bilidown
curl -O https://raw.githubusercontent.com/yangwudong/bilidown-docker/main/docker/docker-compose.yml
docker compose up -d
```

Open [http://localhost:8098](http://localhost:8098) in your browser.

## Configuration

Edit `docker-compose.yml` to customize:

| Setting | Default | Description |
|---------|---------|-------------|
| `TZ` | `Asia/Shanghai` | Container timezone |
| `ports` | `8098:8098` | Web UI port |
| `./download` | — | Downloaded video files |
| `./data.db` | — | SQLite database (login sessions, tasks) |

### Logging

Logs rotate automatically (10MB per file, 3 files max). View logs:

```bash
docker logs -f bilidown
```

## Image Tags

- `ghcr.io/yangwudong/bilidown:latest` — latest release
- `ghcr.io/yangwudong/bilidown:<version>` — specific version (e.g. `v2.1.1`)

Supported platforms: `linux/amd64`, `linux/arm64`

## Build from Source

```bash
git clone https://github.com/yangwudong/bilidown-docker.git
cd bilidown-docker

# Build using the latest bilidown release
docker build -f docker/Dockerfile -t bilidown https://github.com/iuroc/bilidown.git#$(gh api repos/iuroc/bilidown/releases/latest --jq '.tag_name')
```

## How It Works

1. GitHub Actions runs daily (or on manual trigger)
2. Fetches the latest release tag from `iuroc/bilidown`
3. Builds a multi-arch Docker image with:
   - Frontend: VanJS + Bootstrap (built with Vite + pnpm)
   - Backend: Go server with CGO (SQLite via modernc.org/sqlite)
   - Runtime: FFmpeg for video/audio merging
4. Pushes to `ghcr.io/yangwudong/bilidown`

## Acknowledgements

- [iuroc/bilidown](https://github.com/iuroc/bilidown) — upstream project
