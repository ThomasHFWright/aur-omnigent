# Omnigent for Arch Linux

Source-built desktop client from stable tags of
[omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent).
This package contains the desktop app, including its bundled Electron runtime.
Connect it to an existing Omnigent server. It does not install the Python CLI,
server, or agent host.

## Install

Published as [omnigent](https://aur.archlinux.org/packages/omnigent) on AUR:

```sh
shelly install aur omnigent
```

Launch **Omnigent** from the applications menu or run `omnigent-desktop`.
The name of the launcher avoids colliding with the upstream Python CLI.
The app's own binary updater is disabled; Shelly/pacman manages updates.

To build locally without installing:

```sh
makepkg --syncdeps
# Or: shelly build --sync-deps
```

## Release maintenance

The GitHub workflow checks for a new stable upstream release every six hours.
It updates the source version, SHA-256 checksum, and `.SRCINFO`, then builds as
an unprivileged user and runs the package's OIDC checks. Only a successful run
can commit the update and publish it to AUR using `AUR_SSH_PRIVATE_KEY`.
Prereleases and development commits are excluded. Installation stays manual.

GitHub Actions failure notifications identify releases needing maintenance.
Upstream build changes can require edits to the recipe. GitHub can disable
scheduled workflows in inactive public repositories after 60 days.

Manual checks:

```sh
python .github/scripts/update-release.py --self-test
python .github/scripts/update-release.py
makepkg --cleanbuild
```

The release tag is authoritative for the desktop version: upstream v0.17.0
still labels its Electron package 0.14.0. The recipe corrects that metadata.
