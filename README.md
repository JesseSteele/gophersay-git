# gophersay-git
## The talking gopher
Gopher talkback written in Go for Linux

- Built on [gophersay](https://github.com/jessesteele/gophersay), using a `git` repository as source for the local compiler
- This may be useful as an example when needing to compile from a `git` repo
- The package will be `gophersay-git`, but the command installed to the system will still be `gophersay` just as from the [gophersay](https://github.com/jessesteele/gophersay) repository

## Create the simple Linux install package for `gophersay` (via `gophersay-git` package)
This is a guide to create an installer package for the `gophersay` command on:
1. Arch (Manjaro, Black Arch, et al)
2. Debian (Ubuntu, Kali, Mint, et al)
3. RPM (OpenSUSE, RedHat/CentOS, Fedora, et al)

Working examples for each already resides in this repository

### Create and install the `gophersay-git` package directly from this repo

| **Arch** :$ (& Manjaro, Black Arch)

```console
git clone https://github.com/JesseSteele/gophersay-git.git
cd gophersay-git/arch
makepkg -si
```

| **Debian** :$ (& Ubuntu, Kali, Mint)

```console
git clone https://github.com/JesseSteele/gophersay-git.git
sudo apt-get update
sudo apt-get install dpkg-dev debhelper golang-go
cd gophersay-git/deb/build
sudo dpkg-buildpackage -us -uc
cd debian
dpkg-deb --build gophersay
sudo dpkg -i gophersay.deb
```

| **RedHat/CentOS** :$ (& Fedora)

```console
git clone https://github.com/JesseSteele/gophersay-git.git
sudo dnf update
sudo dnf install rpm-build rpmdevtools go
cp -rf gophersay-git/rpm/rpmbuild ~/
rpmbuild -ba ~/rpmbuild/SPECS/gophersay-git.spec
ls ~/rpmbuild/RPMS/noarch/
sudo rpm -i ~/rpmbuild/RPMS/noarch/gophersay-1.0.0-1.noarch.rpm  # Change filename if needed
rm -rf ~/rpmbuild
```

| **OpenSUSE** :$ (& Tumbleweed)

```console
git clone https://github.com/JesseSteele/gophersay-git.git
cd gophersay-git/rpm
sudo zypper update
sudo zypper install rpm-build rpmdevtools go
cp -r rpmbuild ~/
rpmbuild -ba ~/rpmbuild/SPECS/gophersay-git.spec
ls ~/rpmbuild/RPMS/noarch/
sudo rpm -i ~/rpmbuild/RPMS/noarch/gophersay-1.0.0-1.noarch.rpm  # Change filename if needed
rm -rf ~/rpmbuild
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
url="https://github.com/JesseSteele/gophersay-git"
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
  cd "$_cmdname.repo"
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
  cp "$_cmdname.repo/$_cmdname.go" .  # Because we used 'something-here::' in source=
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

### II. Debian Package (`gophersay.deb`)
*Debian "**maintainer**" build directory structure:*

| **`deb/`** :

```
deb/
└─ build/
   └─ debian/
      ├─ source/
      │  └─ format
      ├─ compat
      ├─ control
      ├─ copyright
      ├─ changelog
      ├─ watch
      ├─ install
      └─ rules
```

#### Create Mainainer Package Director Structure
- Create directories: `deb/build/debian`
- In `debian/` create file: `control`
  - Learn about the [Debian source package template control file](https://www.debian.org/doc/debian-policy/ch-controlfields.html#debian-source-package-template-control-files-debian-control)

| **`deb/build/debian/control`** :

```
Source: gophersay-git
Section: games
Priority: optional
Maintainer: Jesse Steele <codes@jessesteele.com>
Homepage: https://github.com/JesseSteele/gophersay-git
Vcs-Git: https://github.com/JesseSteele/gophersay
Build-Depends: debhelper (>= 10), golang-go
Standards-Version: 3.9.6

Package: gophersay-git
#Version: 1.0.0 # No! Inherited from `debian/changelog`
Architecture: all
Depends: bash (>= 4.0)
Description: Gopher talkback written in Go for Linux
```

- In `debian/` create file: `compat`

| **`deb/build/debian/compat`** : (`debhelper` minimum version)

```
10
```

- In `debian/` create file: `changelog`

| **`deb/build/debian/changelog`** : (optional, for listing changes)

```
gophersay-git (1.0.0-1) stable; urgency=low

  * First release

 -- Jesse Steele <codes@jessesteele.com>  Thu, 1 Jan 1970 00:00:00 +0000
```

- In `debian/` create file: `copyright`

| **`deb/build/debian/copyright`** : (optional, may be legally wise)

```
Format: http://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: gophersay-git
Source: https://github.com/JesseSteele/gophersay-git

Files: *
Copyright: 2024, Jesse Steele <codes@jessesteele.com>
License: GPL-3+
```

- In `debian/` create file: `watch`

| **`deb/build/debian/watch`** : (dynamic version based on Git)

```
version=4
https://github.com/JesseSteele/gophersay .*/v?(\d\S*)\.tar\.gz
```

- In `debian/` create file: `rules`
  - Make it executable with :$ `chmod +x debian/rules`

| **`deb/build/debian/rules`** : (build compiler)

```
#!/usr/bin/make -f

%:
	dh $@

override_dh_auto_build:
    git clone https://github.com/JesseSteele/gophersay
    cp gophersay/gophersay.go .
	go build -o gophersay gophersay.go
	rm -rf gophersay

override_dh_auto_install:
	install -D -m 0755 gophersay $(DESTDIR)/usr/bin/gophersay

```

- In `debian/` create file: `install`

| **`deb/build/debian/install`** : (places files in the `.deb` directory structure)

```
gophersay /usr/bin
```

- Create subdirectory: `deb/build/debian/source/`
- In `debian/source/` create file: `format`

| **`deb/build/debian/source/format`** : (source package format)

```
3.0 (quilt)
```

#### Build the Package Directories
- Install the `dpkg-dev`, `debhelper` & `golang-go` packages

| **Install Debian `dpkg-dev` package** :$

```console
sudo apt-get update
sudo apt-get install dpkg-dev debhelper golang-go
```

- Prepare package builder:
  - Navigate to directory `deb/build/`
  - Run this, then the package builder & repo packages will be created:

| **Prepare the Debian package builder** :$

```console
sudo dpkg-buildpackage -us -uc  # Create the package builder
```

- Note what just happened
  - Everything just done to this point is called "**maintainer**" work in the Debian world
  - Basic repo packages *and also* the package `DEBIAN/` builder structure were greated
  - At this point, one could navigate up one directory to `deb/` and run `sudo dpkg -i gophersay_1.0.0-1_all.deb` and the package would be installed, *but we won't do this*
  - The command has also been created at `/usr/bin/gophersay`
    - Once installed with `sudo dpkg -i` (later) this can be removed the standard way with `sudo apt-get remove gophersay`
  - This is the new, just-created directory structure for the standard Debian package builder:

| **`deb/build/debian/`** :

```
deb/build/debian/
          └─ gophersay/
             ├─ DEBIAN/
             │  ├─ control
             │  └─ md5sums
             └─ usr/
                └─ bin/
                   └─ gophersay
```

- Build package:
  - Navigate to directory `deb/build/debian/`
    - :$ `cd debian`
  - Run this, then the package will be built, then installed:

| **Build, *then* install Debian package** :$

```console
dpkg-deb --build gophersay  # Create the .deb package
sudo dpkg -i gophersay.deb  # Install the package
```

- Special notes about Debian
  - This Debian builder relies on GitHub [releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)
    - Eg the `watch` file looks for a `.tar.gz` file for the repo, which is only available if releases are also issued
      - `https://github.com/JesseSteele/gophersay .*/v?(\d\S*)\.tar\.gz`
    - Releases are not automatically issued on GitHub, but are an important part of software intended for production
  - The `deb/build/` directory can be anything, but we want it for housekeeping...
    - `dpkg-buildpackage` will create a laundry list of files as peers to this directory in `deb/`
  - The `debian/control` file builds `DEBIAN/control`, but uses different fields
    - Fields after the empty line in this `debian/control` example will not be recognized by the `dpkg-buildpackage` builder, but will supply information for `DEBIAN/control` in the final `.deb` package
  - The `rules` script will compile `gophersay` from `gophersay.go`
  - The `install` script will place the compiled `gophersay` binary at `usr/bin/gophersay` inside the package
    - This is why we don't need to place the binary at `usr/bin/gophersay` manually
  - Note `usr/local/bin/` won't work for CLI command files because Debian packages expect binary commands to go in `/usr/bin/`
    - Debian can install *directories*—but *cannot install any **file***—under `usr/local/`
    - Trying to install a file will return an [error from the package manager](https://unix.stackexchange.com/questions/409800/) since it expects directories, but only finds a file
  - The standard package build files (for `dpkg-deb --build`) will appear at `deb/build/debian/gophersay/DEBIAN/`
    - So from `deb/build/debian/` one could run `dpkg-deb --build gophersay` to create the `.deb` package at `deb/build/debian/gophersay.deb`
  - The standard package installer will appear at `deb/gophersay_1.0.0-1_all.deb`

| **Remove Debian package** :$ (optional)

```console
sudo apt-get remove gophersay
```

### III. RPM Package (`gophersay-1.0.0-1.noarch.rpm`)
*RPM package directory structure:*

| **`rpm/`** :

```
rpm/
└─ rpmbuild/
   └─ SPECS/
      └─ gophersay-git.spec
```

- Create directories: `rpm/rpmbuild/SPECS`
- In `SPECS/` create file: `gophersay-git.spec`

| **`rpm/rpmbuild/SPECS/gophersay-git.spec`** :

```
Name:           gophersay-git
Version:        1.0.0
Release:        1%{?dist}
Summary:        The talking gopher

License:        GPL
URL:            https://github.com/JesseSteele/gophersay-git
#Source0:        gophersay.go

BuildArch:      noarch
BuildRequires:  git, go
Requires:       bash

%description
Gopher talkback written in Go for Linux

%prep
git clone https://github.com/JesseSteele/gophersay
cd gophersay

%build
go build -o gophersay gophersay.go

%install
mkdir -p %{buildroot}/usr/bin
install -D -m 0755 gophersay %{buildroot}/usr/bin/gophersay

%files
/usr/bin/gophersay

%changelog
* Thu Jan 01 1970 Jesse Steele <codes@jessesteele.com> - 1.0.0-1
- Something started, probably with v1.0.0-1
```

- Install the `rpm-build`, `rpmdevtools` & `go` packages

| **RedHat/CentOS** :$

```console
sudo dnf update
sudo dnf install rpm-build rpmdevtools go
```

| **OpenSUSE** :$

```console
sudo zypper update
sudo zypper install rpm-build rpmdevtools go
```

- Build package:
  - Navigate to directory `rpm/`
  - Run the following commands:

| **Build, *then* install RPM package** :$

```console
cp -r rpmbuild ~/
rpmbuild -ba ~/rpmbuild/SPECS/gophersay-git.spec                     # Create the .rpm package
ls ~/rpmbuild/RPMS/noarch/                                        # Check the .rpm filename
sudo rpm -i ~/rpmbuild/RPMS/noarch/gophersay-1.0.0-1.noarch.rpm  # Install the package (filename may be different)
```

- Special notes about RPM:
  - RPM requires the build be done from `~/rpmbuild/`
  - The resulting `.rpm` fill will be at: `~/rpmbuild/RPMS/noarch/gophersay-1.0.0-1.noarch.rpm`
    - This file might actually have a different name, but should be in the same directory (`~/rpmbuild/RPMS/noarch/`)
  - `noarch` means it works on any architecture
    - This part of the filename was set in the `.spec` file with `BuildArch: noarch`
  - If you get `changelog` or `bad date` error, then consider yourself normal

| **Remove RedHat/CentOS package** :$ (optional)

```console
sudo dnf remove gophersay
```

| **Remove OpenSUSE package** :$ (optional)

```console
sudo zypper remove gophersay
```