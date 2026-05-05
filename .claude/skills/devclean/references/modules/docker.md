# Docker

platform: all
detect: which docker && docker info >/dev/null 2>&1

## Scan

Run only if the Docker daemon is running (detect condition handles this):
- `docker system df` — breakdown of images, containers, volumes, build cache
- `docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}" | sort -k4` — images by age

## Analysis

**Build cache:**
- Safe — intermediate layers from `docker build`; auto-managed by Docker but accumulates over time

**Dangling images (untagged `<none>`):**
- Safe — leftover intermediate images from builds; not used by any container

**Stopped containers:**
- Review — may contain data volumes or logs worth inspecting, but usually safe to remove
- Check: do any stopped containers have volume mounts with data you need?

**Unused images (tagged but no running/stopped container):**
- Review — safe if you can re-pull the image; takes time to re-download on next use
- Confirm before deleting large base images (node, ubuntu, etc.)

**Volumes:**
- Risky — may contain database data, persistent application state
- NEVER include `--volumes` in prune commands without explicit double-confirmation
- List volumes first: `docker volume ls`; user must individually confirm

## Cleanup

Safe   | Remove build cache | `docker builder prune -f`
Safe   | Remove dangling images only | `docker image prune -f`
Review | Remove stopped containers | `docker container prune -f`
Review | Remove unused images + stopped containers + networks | `docker system prune -f`
Review | Remove ALL unused images (including tagged) | `docker system prune -af`
Risky  | Remove unused volumes — ONLY with explicit double-confirmation | `docker volume prune -f`

## NEVER Delete

- Docker daemon config: `/etc/docker/daemon.json`
- Named volumes containing databases or user data — always list and ask per volume
- Running containers (Docker will refuse anyway, but never attempt)
