# qdrant-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Qdrant](https://github.com/qdrant/qdrant).

## What this is

A four-file TOS Docker application archive (`qdrantdocker.tar.gz`) that installs
Qdrant on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `qdrantdocker`)
- `qdrantdocker.lang` — 23-language superset (TOS requires 14)
- `qdrantdocker.svg` — application icon (341 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `qdrant/qdrant:v1.20.1-unprivileged`

## How it works

| | |
|---|---|
| Container image | `qdrant/qdrant:v1.20.1-unprivileged` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8009/` (container port 6333) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/qdrant/storage` → `/Volume1/DockerAppData/qdrantdocker/storage` |
| Logs | `docker logs qdrantdocker` (stdout/stderr) |

`/qdrant/storage` (mounted from `DockerAppData/qdrantdocker/storage`) holds the collections, WAL and snapshots; the dashboard is served at `/dashboard` on the same port.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 6333.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Unprivileged image variant.** The package uses `qdrant/qdrant:<version>-unprivileged`, which is built to run as uid 1000 - the standard image requires root and panics otherwise.
- **One HTTP port.** REST API, dashboard and health endpoint share 6333; gRPC 6334 stays internal.
- **API-only by default.** No dashboard auth - the package says so and recommends a proxy.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/qdrantdocker/storage` | Qdrant application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/qdrantdocker.tar.gz + .sha256
scripts/build.sh 1.20.1-2   # packaging iteration bump
scripts/build.sh 1.20.1-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/qdrantdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v1.20.1-1`**, assets `qdrantdocker.tar.gz` +
   `qdrantdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `qdrantdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `1.20.1-1` → automated validation → review.

## License / branding

Qdrant is distributed by its upstream authors under **Apache-2.0**. The container image is the official image from Docker Hub (`qdrant/qdrant:v1.20.1-unprivileged`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- No authentication out of the box; anyone reaching port 6333 can read and write collections.
- Only the HTTP API is published; gRPC clients from other machines need a reverse proxy or a second port, which the store rules do not allow.
- The dashboard is a thin viewer; heavy administration is done through the API or SDKs.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
