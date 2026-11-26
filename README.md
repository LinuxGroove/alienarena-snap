# Alien Arena snap

A snap of [Alien Arena](https://alienarena.org), the free arena shooter: old school deathmatch with modern features, in bright, arcade-like arenas with a retro alien theme. It's a core26 snap that builds the game from its source (the tag in `snap/snapcraft.yaml`) and isn't made by the Alien Arena team. The source is [kenvandine/alienarena](https://github.com/kenvandine/alienarena), a copy of [alienarena/alienarena](https://github.com/alienarena/alienarena) that carries game controller support until upstream merges it.

```sh
sudo snap install alienarena          # once it's in the Snap Store
alienarena                            # the game
alienarena.server +map dm-dynamo      # a dedicated server, in a terminal
sudo snap connect alienarena:joystick # to play with a game controller
```

## What's in it

- **The game and its data.** Alien Arena keeps its game data (`data1/`, about 1.5 GB) in the same repository as the engine, so one shallow clone of the release tag brings both, and `make install` puts the data in `usr/share/alienarena/`. The snap is about 900 MB.
- **Two apps.** `alienarena` (on the desktop as Alien Arena) and `alienarena.server`, the dedicated server.
- **Interfaces.** The `gnome` extension (desktop, X11, Wayland, OpenGL and the GPU drivers from `mesa-2604`), plus `audio-playback`, `network` and `network-bind` (hosting a game, LAN games, the dedicated server), which connect on their own, and `joystick` (game controllers), which doesn't yet.
- **Where things go.** Settings, downloaded map packs, demos and screenshots are in `~/snap/alienarena/common/.local/share/cor-games/`, which isn't copied for every revision.

## Things to know

- **X11 only.** The engine draws with GLX, so on a Wayland desktop it runs through Xwayland.
- **Controllers need `joystick` connected.** Upstream's Linux engine reads no gamepads (`IN_JoyMove` is empty in `source/unix/rw_unix.c`); the fork adds that through SDL2 (see [Controller support](#controller-support)), and the snap builds it in. Without the `joystick` plug connected the game sees no pads, and on the gamepad shell the stick-as-mouse emulation carries on as before; once it's connected, the shell leaves the pad to the game. It doesn't connect on its own until the Snap Store grants that.
- **How it finds its data.** The game looks for `data1/` in the working folder before the folder built into it (which isn't where snapd mounts the snap), so `snap/local/alienarena-launch` starts it in `$SNAP/usr/share/alienarena`. It also makes the parents of `cor-games/`, which the game won't.
- **Licenses.** The engine is GPL 2 or later. The game data may be passed on freely as a whole, for use with Alien Arena, but not in pieces or for profit (upstream's `docs/license.txt`, in the snap at `usr/share/doc/alienarena/`), so it isn't free software in the Debian sense. The icon is upstream's `unix_dist/alien-arena.png`. The packaging in this repository is MIT.

## Building

```sh
snapcraft                             # in LXD or Multipass
sudo snap install --dangerous alienarena_*.snap
```

CI (`.github/workflows/snap.yml`) builds it for amd64 and arm64 on every push and pull request and keeps each snap as an artifact for 5 days. Every build from `main` is published: pushes go to `edge` and published GitHub releases to `candidate`. That needs the `alienarena` name registered and the `STORE_LOGIN` secret (store credentials from `snapcraft export-login`); without them the publish step fails.

**New Alien Arena builds** are tags on the fork, and the snap's version is the tag. The fork copies upstream's release tags (`7.72.4`) and, each time its sync job has built and tested a release with the controller patches on top, tags that too (`7.72.4-pad.1`, then `7.72.4-pad.2` when the patches change). Once a day `.github/workflows/upstream.yml` compares the fork's newest build with `source-tag` in `snap/snapcraft.yaml`: the newest pad tag once there is one (so the snap never drops back to a build without the patches, even on the day upstream releases and the fork hasn't tagged it yet), and upstream's newest release before that. When the fork is ahead, it builds that tag for both architectures, publishes it to `candidate` and `edge`, and commits the new `source-tag` to `main` (so `main` has to accept pushes from GitHub Actions). If the build fails, nothing is published, `main` stays as it was and the next day's run tries again. Run it by hand from Actions (Upstream release, Run workflow) to build any tag, release or pad, with or without publishing. Changing `source-tag` yourself still works too. GitHub turns off scheduled workflows in a public repository after 60 days with no commits, and its own commits count.

## Controller support

Native gamepad support is being written as a stack of upstream pull requests on [kenvandine/alienarena](https://github.com/kenvandine/alienarena) (its `fork-tools` branch explains how the fork stays current). The fork's daily job replays the patches onto each upstream release, builds and smoke tests them, and only then tags `<release>-pad.N`, which this snap picks up on its own. When upstream merges everything, the fork keeps tagging upstream's releases as `-pad.N` builds, so the snap keeps following it.

The snap builds the pad tags with SDL2 (`libsdl2-dev` in the part's `build-packages`, `libsdl2-2.0-0` in its `stage-packages`; configure finds it on its own), and CI checks that a pad build's `alienarena` links to SDL2. The `alienarena` app has the `joystick` plug, which players connect with `sudo snap connect alienarena:joystick`. Once a pad build has been tried with real controllers, ask on the Snapcraft forum for `joystick` to connect on its own, so players don't have to.
