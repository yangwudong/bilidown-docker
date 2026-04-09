# ============================================
# Stage 1: Build frontend (VanJS + Bootstrap)
# ============================================
FROM node:22-alpine AS frontend

RUN corepack enable && corepack prepare pnpm@9 --activate

WORKDIR /build/client
COPY client/package.json client/pnpm-lock.yaml* ./
RUN pnpm install --frozen-lockfile || pnpm install

COPY client/ ./
RUN pnpm build

# ============================================
# Stage 2: Build Go server
# ============================================
FROM golang:1.23-bookworm AS server

RUN apt-get update && apt-get install -y --no-install-recommends \
    pkg-config gcc libayatana-appindicator3-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build/server

# Copy Go module files first for layer caching
COPY server/go.mod server/go.sum ./
RUN go mod download

# Copy server source and patched main.go will be applied via sed
COPY server/ ./
COPY --from=frontend /build/server/static/ ./static/

# Patch main.go: replace systray.Run(onReady, nil) with direct server start
RUN sed -i 's/systray\.Run(onReady, nil)/onReady(); select {}/' main.go

# Build with CGO enabled
RUN CGO_ENABLED=1 go build -ldflags="-s -w" -o bilidown .

# ============================================
# Stage 3: Runtime
# ============================================
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg curl ca-certificates libayatana-appindicator3-1 \
    && rm -rf /var/lib/apt/lists/*

# Install app to /opt so it stays outside the volume mount
COPY --from=server /build/server/bilidown /opt/bilidown/bilidown
COPY --from=server /build/server/static/ /opt/bilidown/static/

WORKDIR /app

RUN mkdir -p /app/download

COPY <<'EOF' /usr/local/bin/entrypoint.sh
#!/bin/sh
# Ensure binary and static are up-to-date from the image
cp -a /opt/bilidown/bilidown /opt/bilidown/static /app/
exec /app/bilidown
EOF
RUN chmod +x /usr/local/bin/entrypoint.sh

EXPOSE 8098

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
