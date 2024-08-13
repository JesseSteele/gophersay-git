# gophersay
## The talking gopher
Gopher talkback written in Go for Linux

- Built on [gophersay](https://github.com/jessesteele/gophersay), using a `git` repository as source for the local compiler
- This may be useful as an example when needing to compile from a `git` repo
- The package will be `gophersay-git`, but the command installed to the system will still be `gophersay` just as from the [gophersay](https://github.com/jessesteele/gophersay) repository

## Create the simple Linux install package for `gophersay` (via `gophersay-git` package)
This is a guide to create an installer package for the `gophersay` command on:
1. Arch (Manjaro, Black Arch, et al)

Working examples for each already resides in this repository

### Create and install the `gophersay-git` package directly from this repo

| **Arch** :$ (& Manjaro, Black Arch)

```console
git clone https://github.com/JesseSteele/gophersay-git.git
cd gophersay/arch
makepkg -si
```

## Detailed instructions per architecture
Instructions explain each in detail to create these packages from scratch...

- These instructions presume you can access [gophersay.go](https://github.com/JesseSteele/gophersay/blob/main/gophersay.go)

### I. Arch Linux Package (`gophersay-git-1.0.0-1-x86_64.pkg.tar.zst`)
*Arch package directory structure:*

| **`arch/`** :

```
arch/
└─ PKGBUILD
```

- Create directory: `arch`
- In `arch/` create file: `PKGBUILD`

| **`arch/PKGBUILD`** :

```
# Maintainer: Jesse Steele <codes@jessesteele.com>
pkgname=gophersay-git
pkgver=1  # Must not be empty (can be anything), later replaced with the pkgver() function, getting the version from git so this does not need to be re-written on every release
pkgrel=1
pkgdesc="Gopher talkback written in Go for Linux"
url="https://github.com/JesseSteele/gophersay"
arch=('x86_64')     # Go is newer and may not work on older systems, so not 'any'
license=('GPL')
depends=('go')      # Depends on the 'go' package to build the binary
replaces=('gophersay' 'gophersay-tar')

# Custom variable "should" start with _
# Not necessary, but may keep code clean (can remove this, then $_cmdname replace with 'gophersay' everywhere)
_cmdname=gophersay

source=("$_cmdname.repo::git+https://github.com/JesseSteele/$_cmdname.git") # gophersay.repo is just some name we can call it; '$_cmdname.repo::' could be omitted and we could use $_cmdname or 'gophersay'
sha256sums=('SKIP') # We skip the hash since are cloning from git, so security is already dealt with, so the hash won't properly check on a repo/directory anyway

# Dynamically set pkgver= variable based on unique source versioning
# Can go anywhere in PKGBUILD file, but usually variables are first, then functions after
pkgver() {
  cd "$pkgdir"
    ( set -o pipefail
      git describe --long --tags --abbrev=7 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
      git describe --long --abbrev=7 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
      printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
  )
}

build() {
  
  # Get into the root, where our to-be-compiled files are
  cd "$srcdir"

  # Move our files into place from wherever we know they are inside the repo
  cp "$pkgdir/$_cmdname.go" .  # Because we used 'something-here::' in source=
  #cp $pkgname/$_cmdname.go .  # This would also work if we omit 'something-here::' from source=

  # Compile the Go binary
  go build -o "$_cmdname" "$_cmdname.go"
}

package() {
  install -Dm755 "$srcdir/$_cmdname" "$pkgdir/usr/bin/$_cmdname"
}
```

- Place file `gophersay.go` in the same directory as `PKGBUILD`
- Build package:
  - Navigate to directory `arch/`
  - Run this, then the package will be built, then installed with `pacman`:

| **Build & install Arch package** :$ (in one command)

```console
makepkg -si
```

- Use this to build and install in two steps:

| **Build, *then* install Arch package** :$ (first line produces the `.pkg.tar.zst` file for repos or manual install)

```console
makepkg
sudo pacman -U gophersay-git-1.0.0-1-x86_64.pkg.tar.zst
```

- Special notes about Arch:
  - We don't need to resolve any dependencies, we can omit the `-s` flag with `makepkg`
    - This package only needs `bash` as a dependency, which should already be installed merely to execute the script
      - `depends=('bash')` is redundant and only serves as an example in `PKGBUILD`
  - The name of the directory containing the package files does not matter
  - `PKGBUILD` is the instruction file, not a directory as might be expected with other package builders
  - `makepkg` must be run from the same directory containing `PKGBUILD`
  - The `.pkg.tar.zst` file will appear inside the containing directory

| **Remove Arch package** :$ (optional)

```console
sudo pacman -R gophersay-git
```
