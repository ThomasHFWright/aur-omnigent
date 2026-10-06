# Omnigent Desktop for Arch Linux and CachyOS

Source-built desktop client from stable tags of
[omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent).
This package contains the desktop app, including its bundled Electron runtime.
Connect it to an existing Omnigent server. It does not install the Python CLI,
server, or agent host.

The recipe targets x86_64 Arch Linux and CachyOS installations without local
paths, CPU tuning, or desktop-environment assumptions. Build tools are only
make dependencies; Node and pnpm are not required to run the installed app.

## Install

Published as [omnigent-desktop](https://aur.archlinux.org/packages/omnigent-desktop) on AUR:

```sh
shelly install aur omnigent-desktop
```

For a clean-chroot build with tests enabled:

```sh
sudo pacman -S --needed devtools
shelly install aur omnigent-desktop --chroot --check
```

Shelly 3.1.6's GUI does not use its saved `AurInstallUseChroot` setting;
use the explicit CLI option for clean-chroot builds. Enable **Run checks**
for both GUI installs and updates. Native GUI builds can additionally use
`~/.config/shelly/shellybuild.conf`:

```toml
[build]
check = true

[sandbox]
enabled = true
```

The native Landlock sandbox restricts access to home files outside permitted
build paths. It still permits shared `/tmp` and network access; it is not a
clean chroot. This recipe keeps npm and Electron caches inside its build
directory so it needs no extra home-directory permissions.

Launch **Omnigent** from the applications menu or run `omnigent-desktop`.
The name of the launcher avoids colliding with the upstream Python CLI.
The app's own binary updater is disabled; Shelly/pacman manages updates.

Earlier desktop releases were named `omnigent`. The replacement/conflict is
limited to those releases (through 0.17.0-2), leaving future CLI packages free
to use that name.

The bundled runtime preserves upstream's packaged-app behavior; using system
Electron changes `app.isPackaged` and would require additional source patches.
Its `/opt/omnigent-desktop` location follows the
[Arch Electron packaging guidelines](https://wiki.archlinux.org/title/Electron_package_guidelines#Directory_structure).
Namcap's generic ELF-path warning for `/opt` is expected for this layout;
upstream prebuilt libraries can also retain unused links and symbols.

Shelly flags the Node/pnpm commands in this source recipe. `pnpm install`
downloads dependencies using upstream's frozen lockfile; the build commands
execute upstream build tools, and `node --test` runs local OIDC tests. These
warnings describe build-time code execution, not install scriptlets. Review
them with the recipe; the package does not suppress Shelly's checks.

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
