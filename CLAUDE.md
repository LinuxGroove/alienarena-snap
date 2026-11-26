# Alien Arena snap

Snap packaging for [Alien Arena](https://github.com/alienarena/alienarena), an upstream game LinuxGroove doesn't make, built from [kenvandine/alienarena](https://github.com/kenvandine/alienarena), a fork that carries game controller support until upstream merges it. Only the packaging lives here: `snap/snapcraft.yaml`, the launcher in `snap/local/`, the desktop file and icon in `snap/gui/`, and CI in `.github/workflows/`: `snap.yml` builds and publishes, and `upstream.yml` checks the fork daily for a new build (a copied upstream release tag like `7.72.4`, or, once there are any, only its tested `7.72.4-pad.N` tags), builds it with `snap.yml`, publishes it to `candidate` and `edge`, and commits the new `source-tag` to `main`. README.md explains the choices (data path, no `joystick` plug yet, licenses) and what to change when controller builds are ready.

## Commands

```sh
snapcraft                                          # needs LXD or Multipass
sudo snap install --dangerous alienarena_*.snap
alienarena                                         # or alienarena.server +map dm-dynamo
```

Without snapcraft (the cloud container can't reach the Snap Store), check a change by building upstream the way the autotools part does, on Ubuntu 26.04 (the snap is core26; in the cloud container that's a `mirror.gcr.io/library/ubuntu:26.04` Docker container run with `--network host` and the proxy variables): `./configure --prefix=/usr && make -j && make install DESTDIR=/tmp/aa`, then run `snap/local/alienarena-launch alienarena` with `SNAP=/tmp/aa` (after copying the launcher to `/tmp/aa/bin/`) and `SNAP_USER_COMMON` set, under `xvfb-run`. The build packages are in `snap/snapcraft.yaml`.

## Rules

- **Builds are tags on the fork**: a new build means changing `source-tag`, nothing else, unless the build changed. `upstream.yml` does that on its own; keep the `    source-tag: X` line format, since both workflows read and rewrite it with `sed`.
- **Don't patch the game here.** Fixes to the game go upstream, and the controller patches live on the fork's `pad/*` branches (its `fork-tools` branch has the job that keeps them current); this repository only packages it.
- **Keep the data where the launcher looks**: the game finds `data1/` in its working folder, so the launcher `cd`s to `$SNAP/usr/share/alienarena`.
