# Digital archeology #

## Warnings ##

Make a backup of your data!
Downgrading versions is not supported.

### up to and including 7.1.5.0 ###

7.1.5.0 is the last version to have Nura builds.
They won't be around for quite a while.

In the meantime, you can use Flatpak builds instead
and carry over data by renaming
`${XDG_DATA_HOME:-~/.local/share}/io.github.yukigram`
to `~/.var/app/io.gitub.yukigram/data/io.github.yukigram`.

To migrate development builds,
add `.devel` to both `io.github.yukigram`s in the name above.

### before 6.8.3.2 ###

Pre-release versions prior to 6.8.3.2
had a fallback to release paths
and could corrupt production tdata.

6.8.3.2 is the first fully-devel-aware version
that will *not* corrupt tdata
when development and release versions are run side-by-side.

Nura builds are devel-oblivious.

### before 6.8.2.2 ###

6.8.2.2 is the first version to have release all artifacts attached
and permanently saved.

Previous releases' binaries are probably lost media,
as they were only ever cached on Pages and Cachix
and were overwritten a dozen times.

### before 6.7.8.1 ###

Versions prior to 6.7.8.1 are NOT devel-aware.
Running devel-oblivious development versions
*will* corrupt production version's tdata.

To make your installation devel-aware,

1. Close Yukigram.
1. Update Yukigram to a recent enough version.
1. Rename `~/.TelegramDesktop` folder
to `${XDG_DATA_HOME:-~/.local/share}/io.github.yukigram`
(nonisolated package that was used for a dozen years).
1. Go to `${XDG_DATA_HOME:-~/.local/share}` (nonisolated package).
and rename `Yukigram` folder to `io.github.yukigram`.
1. Go to `~/.var/app/io.github.yukigram/data` (flatpak or nixpak)
and rename `Yukigram` folder to `io.github.yukigram`.
1. Go to `~/.var/app/io.github.yukigram.devel/data` (devel, flatpak or nixpak)
and rename `Yukigram` folder to `io.github.yukigram.devel`.
1. Go to `~/.var/app/io.github.yukigram.devel/data` (old nixpak with customNixpakConfig)
and rename `io.github.yukigram` folder to `io.github.yukigram.devel` if that folder doesn't yet exist.
1. Run Yukigram again.

Nura builds are devel-oblivious.

Running Nix-built devel-oblivious development versions
without `customNixpakConfig` or `appId` is *not* supported.

### up to and including 6.4.1.1 ###

This is the last version to be based on 64Gram.

Versions of Yukigram up to and including this one
are deprecated and should no longer be used.

64Gram-style "Enhanced settings" are not supported in Yukigram anymore,
and different settings backends makes it difficult to convert between these automatically.
Make a note of your enchanced settings and toggle them on a new version manually.

Past versions can be found at [yukigram-legacy] repo.

[yukigram-legacy]: https://github.com/yukigram/yukigram-legacy


## How to install ##

Read the warnings above
that are relevant for the version at hand
and proceed carefully.

### with flatpak ###

Download `flatpak-$ARCH` release asset and unpack it.

```shell
flatpak install --user .../flatpak-$ARCH io.github.yukigram
```

Substitute `io.github.yukigram`
with `io.github.yukigram.devel`
for development builds.

### with nix ###

Recent previous and development versions are cached at Cachix.

For historic versions,
download `nixos-$ARCH` release asset,
start an HTTP server of choice on unpacked contents,
and plug its address to `extra-substituters`.

### with Nura ###

Install .apk files from selected release.
