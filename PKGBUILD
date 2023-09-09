# Maintainer: Kisaragi Hiu <mail@kisaragi-hiu.com>

# upstream provides a normal build (with AVX2 instructions) and a
# "baseline" build (without). This is accomplished through Zig by
# setting CPU_TARGET to haswell for AVX2, or nehalem for without?
pkgbase=bun
pkgname=(bun bun-baseline)
pkgver=1.0.0
pkgrel=1
# set in package_*
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
  # Clone submodules once
  cd ${pkgbase}
  # We also want the WebKit submodule
  # Instead of fiddling with the "submodule" target in Makefile, just do this
  git submodule update --init --recursive --progress --depth=1 --checkout
  npm install --ignore-scripts --omit=dev
  cd ..
  # Copy for each package like the emacs PKGBUILD
  cp --reflink=auto -ar ${pkgbase} ${pkgbase}-baseline
}

build () {
  # These errors are on by default but triggered during build.
  # Turn them off.
  # https://github.com/oven-sh/bun/issues/4732
  local _errors_off="UWS_INCLUDE=\
-Wno-implicit-function-declaration \
-Wno-implicit-int \
-Wno-incompatible-function-pointer-types"
  local _jsc="JSC_BASE_DIR=${HOME}/webkit-build"
  cd ${pkgbase}
  # TODO build bun-webkit
  # when WEBKIT_RELEASE_DIR_LTO exists, we probably don't need to set
  # JSC_BASE_DIR
  make jsc

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
