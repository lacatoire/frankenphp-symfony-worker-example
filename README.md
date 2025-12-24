# FrankenPHP + Symfony (Worker mode)

This repository provides a **production-oriented Symfony setup running on FrankenPHP in worker mode**, using a **clean multi-stage Dockerfile**.
It is intended as a **reference implementation** for teams migrating from PHP-FPM to worker-based execution.

---

## Scope

This setup demonstrates:

- Symfony running on **FrankenPHP worker mode**
- A **multi-stage Dockerfile** (builder / runtime / dev / prod)
- Explicit **Caddy configuration** for PHP workers
- Correct **OPcache** usage for long-living processes
- **PCOV enabled in development only**
- A production image without PHP-FPM or Nginx

---

## Development

Requirements:
- Docker
- Docker Compose

```bash
docker compose up --build
```

Then open: `http://localhost:80`

Development runs FrankenPHP in worker mode with source code mounted as a volume.

## Dockerfile design
The Dockerfile is split into clearly scoped stages:
- php-builder – compiles PHP extensions 
- php-runtime – minimal runtime image 
- dev – development tooling and mounted source 
- prod – optimized production image (no dev deps, warmed cache, assets installed)

## Caddy (worker mode)
```bash
:8080 {
    root * /srv/app/public
    php_server {
        root /srv/app/public
        worker index.php
    }

    file_server
    log {
        output stdout
        level ERROR
    }
}
```

## This repository is intended for:
- Symfony developers curious about FrankenPHP 
- Teams migrating from PHP-FPM 
- Developers looking for a copy-friendly base setup

No deep Caddy knowledge is required.