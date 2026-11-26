# OpenTTD

[![GitHub](https://img.shields.io/badge/GitHub-BurrowOps%2FOpenTTD-181717?logo=github)](https://github.com/BurrowOps/OpenTTD) [![Docker Version](https://img.shields.io/docker/v/burrowops/openttd?sort=semver)](https://hub.docker.com/r/burrowops/openttd/) [![Docker Pulls](https://img.shields.io/docker/pulls/burrowops/openttd.svg?maxAge=600)](https://hub.docker.com/r/burrowops/openttd/) [![Docker Stars](https://img.shields.io/docker/stars/burrowops/openttd.svg?maxAge=600)](https://hub.docker.com/r/burrowops/openttd/)

Dedicated [OpenTTD](https://www.openttd.org/) server image. Source code: [github.com/BurrowOps/OpenTTD](https://github.com/BurrowOps/OpenTTD). Based on [bateau/openttd](https://hub.docker.com/r/bateau/openttd) and [linuxserver.io](https://www.linuxserver.io/) Ubuntu with s6-overlay.

[![dockeri.co](https://dockeri.co/image/burrowops/openttd)](https://hub.docker.com/r/burrowops/openttd)

## Tags

Current tags are listed on [Docker Hub](https://hub.docker.com/r/burrowops/openttd/tags) (the badge above also shows the newest version).

| Tag | Meaning |
| --- | --- |
| `latest` | Newest published image |
| `{major}` | Newest image for that OpenTTD major (for example `14`, `15`) |
| `{version}` | First image for that OpenTTD release (for example `15.3`); never overwritten by rebuilds |
| `{version}-{n}` | Rebuild of the same OpenTTD with newer bases (OpenGFX, Ubuntu, …) |

`{version}` is the first image for that OpenTTD release. Later non-OpenTTD bumps are `{version}-1`, `{version}-2`, and so on. Published `{version}` and `{version}-{n}` tags never change. `latest` and `{major}` move to the newest rebuild.

## File locations

Mount a host path at **`/config`**. That volume holds everything that should survive a recreate:

| Path | Purpose |
| --- | --- |
| `/config/openttd.cfg` | Server config (created on first run if missing) |
| `/config/save/` | Savegames |
| `/config/save/autosave/` | Autosaves (used by `loadgame=last-autosave`) |

Edit `openttd.cfg` after the first start to set the server name, password, public listing, and so on, or set them with [config variables](#openttd-config-settings).

## Environment variables

Defaults are baked into the image. Override with `-e` / Compose `environment`.

### Server

| Variable | Default | Meaning |
| --- | --- | --- |
| `loadgame` | `last-autosave` | What to load on start. See values below. |
| `savepath` | `/config/save` | Directory for savegames. Autosaves go in `savepath/autosave`. |
| `savename` | *(unset)* | Save filename when `loadgame=true` (for example `game.sav`). Do not use spaces. |
| `configpath` | `/config` | Directory that contains the config file. |
| `configfile` | `openttd.cfg` | Config filename inside `configpath`. |
| `debug` | `0` | OpenTTD debug level (`-d`). See [OpenTTD debug docs](https://wiki.openttd.org/en/Development/Debugging). |

### `loadgame` values

| Value | Behaviour |
| --- | --- |
| `false` | Start a new game |
| `true` | Load `${savepath}/${savename}` (requires `savename`) |
| `last-autosave` | Load the newest file in `${savepath}/autosave/` (default; falls back to a new game if none exist) |

### OpenTTD config settings

Any setting in OpenTTD's config files can be set from the environment, without the image knowing about it. The name says which file, section and key to set:

| Variable pattern | File |
| --- | --- |
| `OPENTTD_CFG__<section>__<key>` | `${configpath}/${configfile}` (`openttd.cfg`) |
| `OPENTTD_PRIVATE__<section>__<key>` | `${configpath}/private.cfg` |
| `OPENTTD_SECRETS__<section>__<key>` | `${configpath}/secrets.cfg` |

`<section>` and `<key>` are written exactly as in the file (case-sensitive). The value is written as-is.

These are applied on every start, before OpenTTD launches, so they overwrite whatever is in the file. Settings not set this way keep their value from the file. Removing a variable does not undo it: the last value stays in the file.

OpenTTD keeps some settings outside `openttd.cfg` and ignores them there. Use the matching prefix:

| Setting | Variable |
| --- | --- |
| Server name | `OPENTTD_PRIVATE__network__server_name=My server` |
| Public listing | `OPENTTD_CFG__network__server_game_type=public` |
| Server password | `OPENTTD_SECRETS__network__server_password=secret` |
| Rcon password | `OPENTTD_SECRETS__network__rcon_password=secret` |
| Admin port password | `OPENTTD_SECRETS__network__admin_password=secret` |
| Map size | `OPENTTD_CFG__game_creation__map_x=9` (2^9 = 512) |

Check an existing `openttd.cfg`, `private.cfg` or `secrets.cfg` for other section and key names.

### linuxserver / permissions

| Variable | Default | Meaning |
| --- | --- | --- |
| `PUID` | `911` | User ID that owns `/config`. Set this to your host user (`id -u`). |
| `PGID` | `911` | Group ID that owns `/config`. Set this to your host group (`id -g`). |
| `TZ` | *(unset)* | Container timezone, for example `Europe/Amsterdam`. |
| `UMASK` | `022` | File-creation mask for files written under `/config`. |

## Networking

| Port | Required | Purpose |
| --- | --- | --- |
| `3979/tcp` | Yes | Game (default `server_port`) |
| `3979/udp` | Yes | Game |
| `3977/tcp` | No | Remote admin (`server_admin_port` in `openttd.cfg`) |

Publish with `-p 3979:3979/tcp -p 3979:3979/udp`. Forward both protocols on your router if clients connect from the internet.

To appear on the public server list, set `server_game_type = public` in `openttd.cfg` (or use the in-game / rcon equivalent). Listing uses the Game Coordinator; you do not need to publish port 3978.

Control the running server with [rcon](https://wiki.openttd.org/en/Manual/Dedicated%20server): set `rcon_pw` in `openttd.cfg` or via the dedicated-server console.

Give the process time to write `exit.sav` on stop (`stop_grace_period: 30s` in Compose, or `docker stop -t 30`).

## Docker run

Default (resume last autosave, or start a new game if none exist):

```bash
docker run -d \
  --name openttd \
  --restart unless-stopped \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/Amsterdam \
  -e loadgame=last-autosave \
  -v /path/to/openttd:/config \
  -p 3979:3979/tcp \
  -p 3979:3979/udp \
  burrowops/openttd:latest
```

New game:

```bash
docker run -d \
  --name openttd \
  --restart unless-stopped \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/Amsterdam \
  -e loadgame=false \
  -v /path/to/openttd:/config \
  -p 3979:3979/tcp \
  -p 3979:3979/udp \
  burrowops/openttd:latest
```

Load a named save (`/path/to/openttd/save/game.sav`):

```bash
docker run -d \
  --name openttd \
  --restart unless-stopped \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/Amsterdam \
  -e loadgame=true \
  -e savename=game.sav \
  -v /path/to/openttd:/config \
  -p 3979:3979/tcp \
  -p 3979:3979/udp \
  burrowops/openttd:latest
```

To pin a release, replace `latest` with a tag from [Docker Hub](https://hub.docker.com/r/burrowops/openttd/tags).

## Docker Compose

```yaml
services:
  openttd:
    container_name: openttd
    image: burrowops/openttd:latest
    restart: unless-stopped
    stop_grace_period: 30s
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Amsterdam
      - loadgame=last-autosave
      # - savename=game.sav
      # - savepath=/config/save
      # - configpath=/config
      # - configfile=openttd.cfg
      # - debug=0
      # - UMASK=022
      # Any OpenTTD setting: OPENTTD_{CFG,PRIVATE,SECRETS}__<section>__<key>
      # - OPENTTD_PRIVATE__network__server_name=My server
      # - OPENTTD_CFG__network__server_game_type=public
      # - OPENTTD_SECRETS__network__rcon_password=change-me
    volumes:
      - ./openttd:/config
    ports:
      - 3979:3979/tcp
      - 3979:3979/udp
      # Optional: remote admin port (server_admin_port in openttd.cfg)
      # - 3977:3977/tcp
```

A copy of this file lives at [`docker-compose.yml`](https://github.com/BurrowOps/OpenTTD/blob/main/docker-compose.yml). From the repo root:

```bash
docker compose up -d
```

## First run

1. Start the container once so OpenTTD writes `/config/openttd.cfg`.
2. Stop it (`docker compose down` or `docker stop -t 30 openttd`).
3. Edit `openttd.cfg` (server name, password, `server_game_type`, rcon, NewGRFs).
4. Start it again.

Savegame filenames must not contain spaces.

## Version updates

[Renovate](https://github.com/apps/renovate) watches OpenTTD, OpenGFX, the linuxserver Ubuntu base, and GitHub Actions. Install the Renovate GitHub App on this repo. It opens (and auto-merges) PRs; after a `docker/dockerfile` change lands, CI publishes a new image.

linuxserver republishes the `noble` tag without renaming it. That tag is pinned to an image digest in `docker/dockerfile`, so a remote base rebuild shows up here as a digest bump.

A new OpenTTD release becomes `{version}`. A bump of only OpenGFX or the Ubuntu digest becomes `{version}-1`, then `{version}-2`, and so on.
