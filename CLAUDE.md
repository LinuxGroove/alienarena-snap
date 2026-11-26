# Alien Arena snap

Snap packaging for [Alien Arena](https://github.com/alienarena/alienarena), an upstream game LinuxGroove doesn't make. Only the packaging lives here: `snap/snapcraft.yaml`, the launcher in `snap/local/`, the desktop file and icon in `snap/gui/`, and CI in `.github/workflows/`: `snap.yml` builds and publishes, and `upstream.yml` checks daily for a new upstream release, builds it with `snap.yml`, publishes it to `candidate` and `edge`, and commits the new `source-tag` to `main`. README.md explains the choices (data path, no `joystick` plug, licenses).

## Commands

```sh
snapcraft                                          # needs LXD or Multipass
sudo snap install --dangerous alienarena_*.snap
alienarena                                         # or alienarena.server +map dm-dynamo
```

Without snapcraft (the cloud container can't reach the Snap Store), check a change by building upstream the way the autotools part does, on Ubuntu 26.04 (the snap is core26; in the cloud container that's a `mirror.gcr.io/library/ubuntu:26.04` Docker container run with `--network host` and the proxy variables): `./configure --prefix=/usr && make -j && make install DESTDIR=/tmp/aa`, then run `snap/local/alienarena-launch alienarena` with `SNAP=/tmp/aa` (after copying the launcher to `/tmp/aa/bin/`) and `SNAP_USER_COMMON` set, under `xvfb-run`. The build packages are in `snap/snapcraft.yaml`.

## Rules

- **Upstream releases are tags**: a new release means changing `source-tag`, nothing else, unless the build changed. `upstream.yml` does that on its own; keep the `    source-tag: X` line format, since both workflows read and rewrite it with `sed`.
- **Don't patch the game here.** Fixes to the game go upstream; this repository only packages it.
- **Keep the data where the launcher looks**: the game finds `data1/` in its working folder, so the launcher `cd`s to `$SNAP/usr/share/alienarena`.
