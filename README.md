# A unified cross-compilation environment for thb

This repository defines the toolchains that are going to be used for compiling everything
and containes the submodules with all the external dependencies.

The whole system is tested and runs on macOS and supports building for Linux (on `x86`, `x86_64`, `armv5`, `armhf`,
`arm64`, `mips`, `mipsel` with both `glibc` and `musl`), macOS 10.10+ (on `x86_64` and `arm64`) and Windows (on `x86`
and `x86-64`).

Don't forget to run `git clone --recursive` to get all the submodules. If you already forgot then run `git submodule update --init --recursive`. ;)

## Prerequisites

1. The XCode command line tools:

```shell
xcode-select --install
```

2. The `crosstool-ng` toolchains for targetting Linux and Windows:

Download [pre-built toolchains from GitHub](https://github.com/LoEE/crosstool-ng/releases) and unpack them to `/usr/local`.

Building from source (currently does not work on M1/arm64):
```shell
git clone https://github.com/LoEE/crosstool-ng
cd crosstool-ng
diskutil apfs addVolume disk3 'Case-sensitive APFS' crosstool
./bootstrap
homebrew install binutils
ln -s /opt/homebrew/opt/binutils/bin/gobjcopy /opt/homebrew/bin
CPPFLAGS="-I/opt/homebrew/opt/gettext/include -I/opt/homebrew/opt/ncurses/include" LDFLAGS="-L/opt/homebrew/opt/gettext/lib -L/opt/homebrew/opt/ncurses/lib" LIBS=-lintl LIBTOOL=/opt/homebrew/bin/glibtool ./configure --enable-local
make
./build-all
```

You can run `tail -F log-current` in another terminal to watch the progress of the build in more detail.

TODO: use separate directories for each configuration so we can paralelize the build

3. A 32-bit Linux VM for running a 32-bit LuaJIT binary during LuaJIT cross-compilation
  (modern macOS does not support 32-bit binaries but a 32-bit VM works flawlessly, even on the M1):

```shell
multipass launch -n thb-vm32
multipass mount . thb-vm32
multipass exec thb-vm32 -- sudo $($(command -v greadlink || command -v readlink) -f "$PWD")/setup-multipass-guest
```

4. GNU parallel which speeds the build process up significantly:

```shell
brew install parallel
```

## Building

If you follow the default settings for `crosstool-ng` it should be enough to run

```shell
./init-toolchains
```

to get a directory with all the supported configurations:

```console
$ ls toolchains
linux/            linux-armhf/      linux64/          openwrt-mipsel/   win64/
linux-arm64/      linux-armv5/      linux64-musl/     osx/
linux-arm64-musl/ linux-mipsel/     openwrt-mips/     win32/
```

Next you can run ./build-all to compile all platforms in parallel (this should take less than 2 minutes).

```shell
./build-all
```

## Building `thb`

Building `thb` is now easy, just clone it's repository in same parent folder (or adjust the `toolchains` symlink) and
run `./release` or check out the README in the `thb` repository.

```shell
cd ..
git clone https://github.com/LoEE/thb
cd thb
./release
```
