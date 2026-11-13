# Changelog

All notable user-visible changes to the Qdrant TOS application package.

## 1.20.1-1 — 2026-11-14

### Added
- First release. Qdrant (`qdrant/qdrant:v1.20.1-unprivileged`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8009 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/qdrantdocker/`.
