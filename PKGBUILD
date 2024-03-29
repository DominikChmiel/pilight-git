# Maintainer: Philipp Schmitt (philipp<at>schmitt<dot>co)

pkgname=pilight-git
_pkgname=pilight
pkgver=v8.1.5.v8.1.5
_pkgver_major=$(sed -n 's/^v\([0-9]\+\)\.*.*$/\1/p' <<< $pkgver)
pkgrel=1
pkgdesc='Home automation with the Raspberry Pi'
arch=('x86_64' 'armv6h' 'armv7h' 'aarch64')
url="http://pilight.org/"
license=('GPL3')
makedepends=('git' 'gcc' 'glibc' 'cmake' 'libunwind' 'libpcap' 'wiringx-git')
depends=('mbedtls' 'luajit' 'lua51')
source=('git+https://github.com/pilight/pilight.git#branch=staging'
        'pilight.service')
sha256sums=('SKIP'
            '25ffe32693a9a68be4234f63248f6e72e1704cbb74646f77672d02ba19e7f179')
conflicts=('pilight')
backup=(etc/pilight/config.json)

pkgver() {
    cd "${srcdir}/${_pkgname}"
    # Why is the last tag skipped?
    local latest_tag=$(git describe --tags `git rev-list --tags --max-count=1`)
    local description=$(git describe | sed 's/^v.*-\(.*-.*\)$/\1/')
    sed -E 's/([^-]*-g)/r\1/;s/-/./g' <<< "${latest_tag}-${description}"
    # git describe --long | sed -E 's/([^-]*-g)/r\1/;s/-/./g'
}

prepare() {
    cd "${srcdir}/${_pkgname}"
    # Fix zlib path
    sed -i 's|\(/usr/lib/\).*/\(libz.so\)|\1\2|g' CMakeLists.txt
}

build() {
    cd "${srcdir}/${_pkgname}"
    # Adapt to GCC 10 defaulting to -fno-common
    export CFLAGS=-fcommon
    cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr .
    make
}

package() {
    cd "${srcdir}/${_pkgname}"

    make DESTDIR="$pkgdir/" install
    install -Dm644 "${srcdir}/pilight.service" "${pkgdir}/usr/lib/systemd/system/pilight.service"

    # Remove pilight-provided service
    rm "${pkgdir}/etc/systemd/system/pilight.service"

    # Fix paths
    cd "${pkgdir}"
    mv usr/sbin/* usr/bin/
    rm -r usr/sbin
    #mv usr/lib/pilight/libpilight.so.${_pkgver_major} usr/lib/libpilight.so.${_pkgver_major}
    ln -s usr/lib/libpilight.so.${_pkgver_major} usr/lib/libpilight.so
    mkdir usr/local/lib
    mv usr/lib/pilight usr/local/lib/pilight

    # Cleanup
    rm -rf etc/init.d usr/local/lib/pilight/storage/CMakeFiles
}

