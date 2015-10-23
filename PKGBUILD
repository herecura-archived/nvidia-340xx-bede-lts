# vim:set ft=sh et:
# Maintainer : BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Thomas Baechler <thomas@archlinux.org>

_pkgname=nvidia
pkgname=$_pkgname-340xx-bede-lts
pkgver=340.93
_extramodules=4.1-BEDE-LTS-external
pkgrel=3
pkgdesc="NVIDIA 340xx drivers for linux-bede-lts"
arch=('i686' 'x86_64')
url="http://www.nvidia.com/"
makedepends=('linux-bede-lts>=4.1.11' 'linux-bede-lts<4.2' 'linux-bede-lts-headers>=4.1' 'linux-bede-lts-headers<4.2' "nvidia-340xx-utils=$pkgver" "nvidia-340xx-libgl=$pkgver")
conflicts=('nvidia-bede-lts')
provides=('nvidia')
license=('custom')
install=nvidia.install
options=(!strip)

source=(
    "nv-drm.patch"
)
source_i686=("http://download.nvidia.com/XFree86/Linux-x86/$pkgver/NVIDIA-Linux-x86-$pkgver.run")
source_x86_64=("http://download.nvidia.com/XFree86/Linux-x86_64/$pkgver/NVIDIA-Linux-x86_64-$pkgver-no-compat32.run")

sha256sums=('c9986c306f452614fcf23990c55ffe12bdc451bcbd65a5200269f90a722a3d35')
sha256sums_i686=('4a81c158302c595e1e72b5a1812eb3c67c8cf584ca74b1bc24163dad5289d612')
sha256sums_x86_64=('8fb230a7579a15c778ab7c2f160830682919729235beb8ea2b84326528c54843')

[[ "$CARCH" = "i686" ]] && _pkg="NVIDIA-Linux-x86-${pkgver}"
[[ "$CARCH" = "x86_64" ]] && _pkg="NVIDIA-Linux-x86_64-${pkgver}-no-compat32"

prepare() {
    [ -d "$_pkg" ] && rm -rf "$_pkg"
    sh $_pkg.run --extract-only
    cd $_pkg
    # patch if needed
    patch -p0 -i "$srcdir/nv-drm.patch"
}

build() {
    _kernver="$(cat /usr/lib/modules/$_extramodules/version)"
    cd $_pkg/kernel
    make SYSSRC=/usr/lib/modules/$_kernver/build module

    if [[ "$CARCH" = "x86_64" ]]; then
        cd uvm
        make SYSSRC=/usr/lib/modules/"${_kernver}/build" module
    fi
}

package() {
    depends=('linux-bede-lts>=4.1' 'linux-bede-lts<4.2' "nvidia-340xx-utils=$pkgver" "nvidia-340xx-libgl=$pkgver")

    install -Dm644 "$srcdir/$_pkg/kernel/nvidia.ko" \
        "$pkgdir/usr/lib/modules/$_extramodules/$_pkgname/nvidia.ko"

    if [[ "$CARCH" = "x86_64" ]]; then
        install -D -m644 "${srcdir}/${_pkg}/kernel/uvm/nvidia-uvm.ko" \
            "${pkgdir}/usr/lib/modules/${_extramodules}/nvidia-uvm.ko"
    fi

    install -dm755 "$pkgdir/usr/lib/modprobe.d"
    echo "blacklist nouveau" >> "$pkgdir/usr/lib/modprobe.d/$pkgname.conf"
    echo "blacklist nvidiafb" >> "$pkgdir/usr/lib/modprobe.d/$pkgname.conf"

    # gzip all modules
    find "$pkgdir" -name '*.ko' -exec gzip -9 {} \;

    sed -i -e "s/EXTRAMODULES='.*'/EXTRAMODULES='$_extramodules'/" "$startdir/nvidia.install"
}

