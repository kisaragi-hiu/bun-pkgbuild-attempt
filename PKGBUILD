# upstream provides a normal build (with AVX2 instructions) and a
# "baseline" build (without). This is accomplished through Zig by
# setting CPU_TARGET to haswell for AVX2, or nehalem for without?
pkgbase=bun
pkgname=(bun bun-baseline)
pkgver=1.0.0
pkgrel=1
# set in package_* functions
# pkgdesc=""
arch=('x86_64')
url="https://bun.sh"
# FIXME: MIT but with JavaScriptCore (WebKit, LGPL2) statically linked
# TODO: needs installing the license file
license=('MIT')
depends=()
makedepends=(
  'zig' 'npm'
  'llvm' 'clang' 'lld'
  rust # for lol-html
  ccache cmake esbuild git go libiconv libtool make
  ninja pkg-config python sed unzip
)
# Cannot use the release tarball from GitHub as we need to fetch submodules
source=(
  "${pkgbase}::git+https://github.com/oven-sh/bun.git#tag=bun-v${pkgver}"
)
sha256sums=('SKIP')

prepare () {
  cd ${pkgbase}

  # There is a C++ flag that is not correctly included as part of a string
  patch -p0 < ../jsc_incorrect_argument.patch

  # Clone submodules once
  #
  # We also want the WebKit submodule, which `make submodule` doesn't
  # provide
  git submodule update --init --recursive --progress --depth=1 --checkout
  npm install --ignore-scripts --omit=dev
  cd ..
  # Copy for each package like the emacs PKGBUILD
  cp --reflink=auto -ar ${pkgbase} ${pkgbase}-baseline
}

build () {
  cd ${pkgbase}
  # "jsc" depends on the "headers" target, which needs "node-fallbacks"
  # and other targets built by the "vendor" target
  make node-fallbacks
  # `make jsc` fails with FileNotFound without this
  make identifier-cache

  # The "jsc" task is broken down because the last part of it requires
  # some care beforehand
  make jsc-build jsc-copy-headers

  # The LTO folder is where the Makefile expects JSC to be built to
  # I haven't figured out how to get an LTO build, meanwhile just use
  # the non-LTO folder
  local _jsc="WEBKIT_RELEASE_DIR_LTO=$(realpath src/bun.js/WebKit/WebKitBuild/Release/)"
  # FIXME: this still fails with issues around libatomic.a and finding
  # the ICU library
  make "$_jsc" jsc-bindings

  # These errors are on by default but triggered during build.
  # Turn them off.
  # https://github.com/oven-sh/bun/issues/4732
  local _errors_off="UWS_INCLUDE=\
-Wno-implicit-function-declaration \
-Wno-implicit-int \
-Wno-incompatible-function-pointer-types"
  make \
    "$_errors_off" \
    bun

  # cd ${pkgbase}-${pkgver}-baseline
  # make baseline
}

package_bun() {
  pkgdesc="abc"
  cd ${pkgbase}
}
package_bun-baseline() {
  pkgdesc="abc (no AVX instructions)"
  provides=(bun)
  conflicts=(bun)

  cd ${pkgbase}-baseline
}
