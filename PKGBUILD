# Maintainer: so5iso4ka <so5iso4ka@icloud.com>

pkgname=freesmlauncher
pkgver=1.3.3
pkgrel=1
pkgdesc='Minecraft launcher with ability to manage multiple instances.'
arch=('i386' 'amd64' 'arm64' 'armhf' 'riscv64')
url='https://website-freesmlauncher.vercel.app/'
license=('GPL-3')
depends=(
  'libqt6core5compat6'
  'libqt6core6'
  'libqt6network6'
  'libqt6networkauth6'
  'libqt6svg6'
  'libqt6widgets6'
  'libqt6xml6'
  'qt6-image-formats-plugins'
)
makedepends=(
  'cmake'
  'extra-cmake-modules'
  'g++'
  'gcc'
  'git'
  'libgl1-mesa-dev'
  'libqt6core5compat6-dev'
  'openjdk-17-jdk'
  'qt6-base-dev'
  'qt6-networkauth-dev'
  'qtchooser'
  'scdoc'
  'zlib1g-dev'
)
optdepends=(
  'flite: narrator support'
  'java-runtime=17: support for Minecraft versions >= 1.17 and <= 1.20.4'
  'java-runtime=21: support for Minecraft versions >= 1.20.5'
  'java-runtime=8: support for Minecraft versions <= 1.16'
  'x11-xserver-utils: xrandr is needed to support Minecraft versions <= 1.12'
  's!gamemode: support for GameMode'
  's!mangohud: HUD overlay for FPS and temperatures'
)
source=(
  "https://github.com/FreesmTeam/FreesmLauncher/releases/download/sequoia-${pkgver}/FreesmLauncher-sequoia-${pkgver}.tar.gz"
  'gcc-armv7-fix.patch'
  'copyright'
)
sha256sums=('0a9814a6ab69fbc6f49ed80a1632a3da2d3527b9ccd23c549a08459d2e486f7e'
            '42394447d4b52c9329ff45f3c700c0eb2090a5803c5de010587d64294c37420f'
            '25de382d392c05a1835b9d9a2ea29e6c7c8cdebbb8c098244d377f6fbed8b9e3')

# allow for ARM support
#TODO: makedeb's hard-coding for x86-64 has been fixed in a future makedeb version
#TODO: these 8 lines make this script match the behavior of future makedeb. When it releases, remove this
CARCH="$(dpkg --print-architecture)"
CHOST="$(uname -m)-pc-linux-gnu"
CFLAGS=${CFLAGS/-march=x86-64/}
CXXFLAGS=${CXXFLAGS/-march=x86-64/}
CFLAGS=${CFLAGS/-mtune=generic/}
CXXFLAGS=${CXXFLAGS/-mtune=generic/}
CFLAGS=${CFLAGS/-fcf-protection/}
CXXFLAGS=${CXXFLAGS/-fcf-protection/}

# if the user hasn't specified a tuning/architecture, specify our own minimal defaults to cover the earliest CPUs
if [[ ${CFLAGS} != *"-mtune"* && ${CFLAGS} != *"-march"* ]]; then
  case "${CARCH}" in
    amd64)
      CFLAGS+=" -march=x86-64 -fcf-protection"
      CXXFLAGS+=" -march=x86-64 -fcf-protection"
      ;;
    i386)
      CFLAGS+=" -march=i686"
      CXXFLAGS+=" -march=i686"
      ;;
    arm64)
      CFLAGS+=" -march=armv8-a"
      CXXFLAGS+=" -march=armv8-a"
      ;;
    armhf)
      CFLAGS+=" -march=armv7-a+fp"
      CXXFLAGS+=" -march=armv7-a+fp"
      ;;
    riscv64)
      CFLAGS+=" -march=rv64imafdc"
      CXXFLAGS+=" -march=rv64imafdc"
      ;;
  esac
fi

prepare() {
  # workaround https://gcc.gnu.org/bugzilla/show_bug.cgi?id=64860
  # more info: https://github.com/FreesmLauncher/FreesmLauncher/issues/128
  if [[ "$(uname -m)" = armv7* ]]; then
    echo "GCC / ARMv7 fix is needed for this architecture, applying gcc-armv7-fix.patch"
    patch --directory="FreesmLauncher-sequoia-${pkgver}" --forward --strip=1 --input="${srcdir}/gcc-armv7-fix.patch"
  else
    echo "GCC / ARMv7 fix is not needed for this architecture, skipping gcc-armv7-fix.patch"
  fi
}

build() {
  cd "${srcdir}/FreesmLauncher-sequoia-${pkgver}"
  cmake -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX="/usr" \
    -DLauncher_BUILD_PLATFORM="debian" \
    -DLauncher_APP_BINARY_NAME="${pkgname}" \
    -DLauncher_ENABLE_JAVA_DOWNLOADER=ON \
    -DENABLE_LTO=ON \
    -Bbuild -S.
  cmake --build build
}

check() {
  cd "${srcdir}/FreesmLauncher-sequoia-${pkgver}/build"
  ctest . -E Task  # Skip unreliable Task test
}

package() {
  cd "${srcdir}/FreesmLauncher-sequoia-${pkgver}/build"
  DESTDIR="${pkgdir}" cmake --install .
  mkdir -p "${pkgdir}/usr/share/doc/${pkgname}"
  cp -v "${srcdir}/copyright" "${pkgdir}/usr/share/doc/${pkgname}/copyright"
}

# vim: set sw=2 expandtab:
