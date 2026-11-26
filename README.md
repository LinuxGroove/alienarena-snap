# Alien Arena snap

A snap of [Alien Arena](https://alienarena.org), the free arena shooter: old school deathmatch with modern features, in bright, arcade-like arenas with a retro alien theme. It's a core26 snap that builds the game from its source at [alienarena/alienarena](https://github.com/alienarena/alienarena) (the tag in `snap/snapcraft.yaml`) and isn't made by the Alien Arena team.

```sh
sudo snap install alienarena          # once it's in the Snap Store
alienarena                            # the game
alienarena.server +map dm-dynamo      # a dedicated server, in a terminal
```

## What's in it

- **The game and its data.** Alien Arena keeps its game data (`data1/`, about 1.5 GB) in the same repository as the engine, so one shallow clone of the release tag brings both, and `make install` puts the data in `usr/share/alienarena/`. The snap is about 900 MB.
- **Two apps.** `alienarena` (on the desktop as Alien Arena) and `alienarena.server`, the dedicated server.
- **Interfaces.** The `gnome` extension (desktop, X11, Wayland, OpenGL and the GPU drivers from `mesa-2604`), plus `audio-playback`, `network` and `network-bind` (hosting a game, LAN games, the dedicated server). All of them connect on their own.
- **Where things go.** Settings, downloaded map packs, demos and screenshots are in `~/snap/alienarena/common/.local/share/cor-games/`, which isn't copied for every revision.

## Things to know

- **X11 only.** The engine draws with GLX, so on a Wayland desktop it runs through Xwayland.
- **No controller support.** The Linux engine reads no gamepads (`IN_JoyMove` is empty in `source/unix/rw_unix.c`), so the snap has no `joystick` plug. That's on purpose: on the gamepad shell, a snap with `joystick` connected gets no stick-as-mouse emulation, which is the only way a pad can drive this game.
- **How it finds its data.** The game looks for `data1/` in the working folder before the folder built into it (which isn't where snapd mounts the snap), so `snap/local/alienarena-launch` starts it in `$SNAP/usr/share/alienarena`. It also makes the parents of `cor-games/`, which the game won't.
- **Licenses.** The engine is GPL 2 or later. The game data may be passed on freely as a whole, for use with Alien Arena, but not in pieces or for profit (upstream's `docs/license.txt`, in the snap at `usr/share/doc/alienarena/`), so it isn't free software in the Debian sense. The icon is upstream's `unix_dist/alien-arena.png`. The packaging in this repository is MIT.

## Building

```sh
snapcraft                             # in LXD or Multipass
sudo snap install --dangerous alienarena_*.snap
```

CI (`.github/workflows/snap.yml`) builds it for amd64 and arm64 on every push and pull request and keeps each snap as an artifact for 5 days. With the name registered, the `STORE_LOGIN` secret set and the repository variable `PUBLISH_SNAP` set to `true`, pushes to `main` go to `edge` and published GitHub releases to `candidate`.

**New Alien Arena releases** are tags upstream, and the snap's version is the tag. Once a day `.github/workflows/upstream.yml` compares upstream's newest release tag with `source-tag` in `snap/snapcraft.yaml`. When upstream is ahead, it builds that release for both architectures, publishes it to `candidate` and `edge`, and commits the new `source-tag` to `main` (so `main` has to accept pushes from GitHub Actions). If the build fails, nothing is published, `main` stays as it was and the next day's run tries again; until `PUBLISH_SNAP` is set it only leaves a notice. Run it by hand from Actions (Upstream release, Run workflow) to build any tag, with or without publishing. Changing `source-tag` yourself still works too. GitHub turns off scheduled workflows in a public repository after 60 days with no commits, and its own commits count.
