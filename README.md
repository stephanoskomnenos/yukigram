# Yukigram: patch-based [Telegram Desktop][tdesktop] fork

[tdesktop]: https://github.com/telegramdesktop/tdesktop

Minus the bullshit, plus the features.

- [Full feature/bug list]
- [Contributing notes]
- [LLM usage policy (tldr: NO.)][LLM usage policy]
- [Past versions and migration guides]
- [Telegram channel]

[Full feature/bug list]: ./FEATURES.md
[Contributing notes]: ./CONTRIBUTING.md
[LLM usage policy]: ./CONTRIBUTING.md#fuck-off-with-your-llms
[Past versions and migration guides]: ./ARCHEOLOGY.md
[Telegram channel]: https://telegram.me/yukigram

## Patches?

Most clients (all of them?) are _forks_,
and most forks don't bother rebasing their changes over and over,
and instead opt for introducing merge commits
to be on par with upstream changes.
This makes history and changes harder to track and audit.
Patch-based approach is, in a way,
a rebase-workflow but without commits,
and with [range-diff] included,
allowing to more precisely track how patches evolve over time
and understand code easier.

Please inspect these patches yourselves to make sure that you really trust the code.

Some more information can be found in [Contributing notes].

[range-diff]: https://git-scm.com/docs/git-range-diff

## Installation

> Jump to
> [Flatpak][with-flatpak],
> [Nix][with-nix],
> [Nura][on-nura],
> [your favorite distro][distro-packages],
> [prebuilt binaries][from-binaries],
> [building from source][from-source].

[with-flatpak]: #with-flatpak
[with-nix]: #with-nix
[on-nura]: #on-nura-or-alpine
[distro-packages]: #distro-packages
[from-binaries]: #from-binaries
[from-source]: #from-source

Do not blindly trust our binary caches.
They are built by GitHub Actions,
and are supposed to be reproducible, though.

Windows and macOS are *not* supported yet.

Consider using isolated packages
and reviewing and refining their permissions
(with e.g. [Flatseal] for flatpak
or with [custom Nixpak options] for nixpak)
instead of plain binaries.

[Flatseal]: https://github.com/tchx84/Flatseal
[custom Nixpak options]: #custom-options

### with flatpak

```shell
flatpak remote-add --user --if-not-exists yukigram https://yukigram.github.io/yukigram/index.flatpakrepo
flatpak install --user io.github.yukigram
```

### with nix

#### binary caches setup

Add `yukigram-nixos-binary-cache:JY9MpP2ESUmPx3cfIpcSRpBK9HQ1/mzHemsvjv1aiYU=`
to `extra-trusted-public-keys`.

Add `https://yukigram.github.io/yukigram`
to `extra-substituters`.

Add `yukigram-official.cachix.org-1:PmmKVD/46LWDxfPWKol4rvoqvcdLqFq0aTtG/E1gdA8=`
to `extra-trusted-public-keys`
for previous and development versions.

Add `https://yukigram-official.cachix.org`
to `extra-substituters`
for previous and development versions.

If you use Nix to manage nix config,
activate a configuration with these settings at least once
before adding Yukigram to packages.

If you supply these options as command-line-arguments,
make sure to run builds from a trusted user,
otherwise binary caches won't work.

Binary caches are only built for tagged releases.
Track `release` branch instead of `master`
if your pinning tool does not support tracking tags.

Recommended flake input formats are:
- `github:yukigram/yukigram/release` for latest releases,
- `github:yukigram/yukigram/v6.7.8.1` for pinned releases,
- `github:yukigram/yukigram/beta` for latest pre-release.

Pre-release Nix builds are published
only to Cachix and release artifacts.

#### with nixpak

without flakes:
```nix
(import <yukigram> {}).packages.default
```

with flakes:
```nix
inputs.yukigram.packages.${system}.default
```

`default` can be replaced with `nixpak` for explicitness.

#### without nixpak

without flakes:
```nix
(import <yukigram> {}).packages.nonisolated
```

with flakes:
```nix
inputs.yukigram.packages.${system}.nonisolated
```

#### custom options

Custom nixpak options are required to launch Yukigram nixpak under X11.

```nix
let
  d = import <yukigram> {}; # without flakes
  d = inputs.yukigram.d.${system}; # with flakes
  yukigram = d.override (prev: {
    # customize nixpak
    nixpak.yukigram = prev.nixpak.yukigram.override {
      appId = "io.github.yukigram.backed";
      customNixpakConfig = {
        bubblewrap.sockets.x11 = true;
      }
    }

    # or bring your own patches (disables binary caches)
    packages.nonisolated = prev.packages.nonisolated.override (prev: {
      unwrapped = prev.unwrapped.overrideAttrs (prev: {
        patches = prev.patches ++ [./my-cool.patch];
      };
    };

    # or bring your own nixpkgs (may disable binary caches)
    sources.nixpkgs = <nixpkgs>;

    # or use a custom instance of nixpkgs (may disable binary caches)
    inputs.pkgs = import <nixpkgs> {};

    # or make nonisolated package the default one
    packages.default = prev.packages.nonisolated;
  });
in yukigram.packages.default;
```

## Windows release builds

Windows remains unsupported for end users,
but this repository contains a manual GitHub Actions release build
for producing Windows artifacts when the current patchset builds there.
The workflow intentionally builds Telegram Desktop from source,
applies `tdesktop/cur/*.patch`, records upstream/patchset hashes,
and uploads a portable artifact to a GitHub Release.

To run it from a checkout with GitHub CLI access:

```shell
gh workflow run windows-release.yml \
  -f upstream_ref=v7.2.8 \
  -f architecture=x64 \
  -f configuration=Release \
  -f devel=false \
  -f use_dependency_cache=false \
  -f release_tag=v7.2.8.0-rc.1
```

The current patchset targets Telegram Desktop `v7.2.8`.
Its dependency preparation includes the libvpx fix for NASM 3.x.
Keep `upstream_ref` aligned with the patchset when updating from upstream.

The default toolchain check expects
Windows SDK `10.0.26100.0` and MSVC toolset `14.44`.
If the hosted runner does not provide them,
the workflow should fail early before preparing dependencies.
When run from a tag ref, `release_tag` may be left empty and the current
tag is used. The release tag is checked before cloning or building.

### on Nura or Alpine

Temporarily disabled due to build issues (last version v7.1.5.0).
They will be restored later, but in a separate repo.
Past versions can be installed from released `.apk` files.

### distro packages

Unofficial packages that seem correct and maintained:

- AUR package [`yukigram-desktop`][aur-yukigram-desktop] (from source, releases)
- AUR package [`yukigram-desktop-clang`][aur-yukigram-desktop-clang] (from source, releases, with clang)
- AUR package [`yukigram-bin`][aur-yukigram-bin] (binary, releases, x86_64-only)

[aur-yukigram-desktop]: https://aur.archlinux.org/packages/yukigram-desktop
[aur-yukigram-desktop-clang]: https://aur.archlinux.org/packages/yukigram-desktop-clang
[aur-yukigram-bin]: https://aur.archlinux.org/packages/yukigram-bin

### manual installation

#### from binaries

Download `binary-$ARCH` from releases
and unpack it to `/usr/local/` or `~/.local/`.

Automatic updates are unsupported with this approach.

#### from source

Check your package manager's manual
on how to add patches to packages
and package Telegram Desktop (of correct version)
with your selection of patches.

The recommended way of applying patches
for building for repositories
is by using `patch`.
See [#applying-with-patch]
in contributing notes for details.

[#applying-with-patch]: CONTRIBUTING.md#applying-with-patch

## Licensing

This project follows the [REUSE specification][reuse-spec].

Patches for Telegram Desktop (under `tdesktop/cur/`)
are provided under [GPL-3.0-or-later]
with [OpenSSL exception][sqlitestudio-OpenSSL-exception].

GitHub Actions workflows and flatpak manifest
are provided under [MIT].

Nix package definitions
are provided under [CC0-1.0] or [Unlicense] at your option.

This file is licensed under [CC0-1.0].

See [`REUSE.toml`](./REUSE.toml) for some details.

Additionally, `.svg` files embedded inside patches in `tdesktop/cur/`, namely

- Telegram/Resources/art/yukigram.svg
- Telegram/Resources/icons/info/edit/group_manage_actions_topbar.svg
- Telegram/Resources/icons/info/edit/group_manage_admins_topbar.svg

are provided under [CC0-1.0].

[reuse-spec]: https://reuse.software/spec-3.3/
[GPL-3.0-or-later]: ./LICENSES/GPL-3.0-or-later.txt
[sqlitestudio-OpenSSL-exception]: ./LICENSES/LicenseRef-sqlitestudio-OpenSSL-exception.txt
[CC0-1.0]: ./LICENSES/CC0-1.0.txt
[Unlicense]: ./LICENSES/Unlicense.txt
[MIT]: ./LICENSES/MIT.txt
[`REUSE.toml`]: ./REUSE.toml
