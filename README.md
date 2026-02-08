# OpenTTD [![Docker Version](https://img.shields.io/docker/v/burrowops/openttd?sort=semver)](https://hub.docker.com/r/burrowops/openttd/) [![Docker Pulls](https://img.shields.io/docker/pulls/burrowops/openttd.svg?maxAge=600)](https://hub.docker.com/r/burrowops/openttd/) [![Docker Stars](https://img.shields.io/docker/stars/burrowops/openttd.svg?maxAge=600)](https://hub.docker.com/r/burrowops/openttd/)

## Tag descriptions

* `latest` – most up-to-date version.
* `14.0` – specific OpenTTD version (example; tags follow release versions).

---

[![dockeri.co](https://dockeri.co/image/burrowops/openttd)](https://hub.docker.com/r/burrowops/openttd)

This image is based on [bateau/openttd](https://hub.docker.com/r/bateau/openttd).

## Usage

### File locations

All persistent data lives under **`/config`** in the container. Mount a host path to `/config` so the server can read/write config and savegames. Typical layout:

- **`/config/openttd.cfg`** – server config (created on first run if missing)
- **`/config/save/`** – savegames
- **`/config/save/autosave/`** – autosaves (used by `loadgame=last-autosave` and `loadgame=exit`)

Use Docker’s `-v` to mount a host path at `/config` (see [Docker run reference](https://docs.docker.com/engine/reference/commandline/run/#volume)).

### Environment variables

Defaults are set in the image; override with Docker’s `-e`. See [Docker run reference](https://docs.docker.com/engine/reference/commandline/run/#env).

| Env          | Default             | Meaning |
| ------------ | ------------------- | ------- |
| `loadgame`   | `last-autosave`     | **false** = new game; **true** = load save (set `savename`); **last-autosave** = latest in `savepath/autosave`; **exit** = `savepath/autosave/exit.sav`. |
| `savepath`   | `/config/save`      | Directory for savegames; autosaves go in `savepath/autosave`. |
| `savename`   | –                   | Save filename when `loadgame=true` (e.g. `game.sav`). |
| `configpath` | `/config`           | Directory containing `openttd.cfg`. |
| `configfile` | `openttd.cfg`       | Config filename inside `configpath`. |
| `debug`      | `0`                 | OpenTTD debug flags; see OpenTTD docs. |
| `PUID`       | `911`               | User ID in container (for ownership of `/config`). |
| `PGID`       | `911`               | Group ID in container. |

### Networking

The server listens on **3979/tcp** and **3979/udp**. Expose with `-p 3979:3979/tcp -p 3979:3979/udp` (or `-P` for random host ports).

### Examples

Default is `loadgame=last-autosave`; mount `/config` so the server can find saves. For a **new game**, set `-e loadgame=false`.

### Docker Compose

```yaml
services:
  openttd:
    container_name: openttd
    image: BurrowOps/openttd:latest
    restart: unless-stopped
    volumes:
      - ./openttd:/config
    ports:
      - 3979:3979/tcp
      - 3979:3979/udp
```

Savegame filenames must not contain spaces.

## Kubernetes

Example deploy: use a ConfigMap for `openttd.cfg`, a Deployment, and a Service exposing port 31979 (UDP/TCP). Point the image to `burrowops/openttd:latest` (or the desired tag).

## Other tags

See [burrowops/openttd](https://hub.docker.com/r/burrowops/openttd) on Docker Hub for available tags.
