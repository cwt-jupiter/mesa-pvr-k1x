# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgname=mesa-pvr-k1x
pkgdesc="Mesa wrapper for PVR DDK 23.2 blobs (SpacemiT K1-x)"
_tag=v2.0.4
pkgver=24.0.1+ddk24.2+bianbu_${_tag}
pkgrel=1
arch=('riscv64')
makedepends=('git' 'python-mako' 'xorgproto'
              'libxml2' 'libx11'  'libvdpau' 'libva' 'elfutils' 'libxrandr'
              'wayland-protocols' 'meson' 'ninja' 'glslang' 'pandoc-cli')
depends=('mesa' 'img-gpu-k1x')
url="https://github.com/Icenowy/aosc-os-pvr/tree/master/ddk232"
license=('MIT AND Khronos AND SGI-Free-Software-License-B AND Boost-permissive')
_srcname=mesa3d-${_tag}
source=("${_srcname}.tar.gz::https://gitee.com/bianbu-linux/mesa3d/repository/archive/${_tag}.tar.gz"
        'https://github.com/Icenowy/aosc-os-pvr/raw/refs/heads/master/ddk242/mesa-pvr-ddk242/autobuild/patches/0001-redirect-glapi.patch'
        'https://github.com/Icenowy/aosc-os-pvr/raw/refs/heads/master/ddk242/mesa-pvr-ddk242/autobuild/patches/0002-change-LIBGL_ALWAYS_SOFTWARE-to-PVRGL_ALWAYS_SOFTWAR.patch'
        'https://github.com/Icenowy/aosc-os-pvr/raw/refs/heads/master/ddk242/mesa-pvr-ddk242/autobuild/patches/0003-Revert-gbm-add-gbm_bo_blit.patch'
        'https://github.com/Icenowy/aosc-os-pvr/raw/refs/heads/master/ddk242/mesa-pvr-ddk242/autobuild/patches/0004-gbm-backend-ize.patch'
        'https://github.com/Icenowy/aosc-os-pvr/raw/refs/heads/master/ddk242/mesa-pvr-ddk242/autobuild/patches/0005-add-dri-alias-for-spacemit.patch')

sha256sums=('be182bc5d7ebe08e1608d53b9225ded260c4420ec95fd154ba00d17bbebce85b'
            'f1a6c30b87106e7f57e7dc4c69478fdf0332580e529cf72c450caf08cef897c7'
            '95412206b60605b19e85cab24d9e3c37cf41ec90cfc85f67bb3b85e156ed1798'
            '40382e81fa143167b6c511f8846701d2998adae6eae5db5de312683904929124'
            'df3e46ed38335b813881feb90fc3a748f0d75daf548c78f42566b29f8d56479d'
            'ec1d09d6db7c485bc10544a680d59a6bede33dc0aefaa3609ea160e430f8b548')

prepare() {
    # although removing _build folder in build() function feels more natural,
    # that interferes with the spirit of makepkg --noextract
    if [  -d _build ]; then
        rm -rf _build
    fi

    sed -i 's/\r$//' ${srcdir}/${_srcname}/src/mesa/main/formats.csv;

    local _patchfile
    for _patchfile in "${source[@]}"; do
        _patchfile="${_patchfile%%::*}"
        _patchfile="${_patchfile##*/}"
        [[ $_patchfile = *.patch ]] || continue
        echo "Applying patch $_patchfile..."
        patch -F3 --directory="${_srcname}" --forward --strip=1 --input="${srcdir}/${_patchfile}"
    done
}

build () {
    cd ${srcdir}
    mkdir _build
    cd _build
    meson setup ${srcdir}/${_srcname} --prefix=/usr -Ddri-drivers-path=/usr/lib/pvr-dri \
        -Dglvnd=true -Dglvnd-vendor-name=pvr \
        -Dgallium-drivers=pvr -Dvulkan-drivers=pvr \
        -Dglx=disabled -Dllvm=disabled -Dgbm=enabled
    ninja $NINJAFLAGS -C .
}

package() {
    mkdir -p ${pkgdir}/usr/lib ${pkgdir}/usr/share/glvnd/egl_vendor.d
    DESTDIR=${srcdir}/tmp ninja $NINJAFLAGS -C ${srcdir}/_build install
    cp -a ${srcdir}/tmp/usr/lib/*pvr* ${pkgdir}/usr/lib/
    cp -r ${srcdir}/tmp/usr/lib/gbm ${pkgdir}/usr/lib/
    ln -sf pvr_gbm.so ${pkgdir}/usr/lib/gbm/spacemit_gbm.so
    cp ${srcdir}/tmp/usr/share/glvnd/egl_vendor.d/50_pvr.json ${pkgdir}/usr/share/glvnd/egl_vendor.d/40_pvr.json
    mkdir -p ${pkgdir}/usr/share/licenses/${pkgname}
    pandoc -f rst -t plain ${srcdir}/${_srcname}/docs/license.rst -o ${pkgdir}/usr/share/licenses/${pkgname}/LICENSE
}
