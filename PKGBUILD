# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgname=mesa-pvr-k1x
pkgdesc="Mesa wrapper for PVR DDK 23.2 blobs (SpacemiT K1-x)"
pkgver=23.2+bianbu1.0_alpha2
pkgrel=1
arch=('riscv64')
makedepends=('git' 'python-mako' 'xorgproto'
              'libxml2' 'libx11'  'libvdpau' 'libva' 'elfutils' 'libxrandr'
              'wayland-protocols' 'meson' 'ninja' 'glslang' )
depends=('mesa' 'img-gpu-k1x')
url="https://www.mesa3d.org"
license=('custom')
_tag=v1.0.15
_srcname=mesa3d-${_tag}
source=("${_srcname}.tar.gz::https://gitee.com/bianbu-linux/mesa3d/repository/archive/${_tag}.tar.gz"
        '0001-redirect-glapi.patch'
        '0002-change-LIBGL_ALWAYS_SOFTWARE-to-PVRGL_ALWAYS_SOFTWAR.patch'
        '0003-Revert-gbm-add-gbm_bo_blit.patch'
        '0004-gbm-backend-ize.patch'
        '0005-add-dri-alias-for-spacemit.patch')

sha256sums=('f6be25a12d8c61a496cac1a95cd185f9d4195be4197e14bcf030a96f1baa7d16'
            'ebb6f66cb1dcfa29542fa220977462352febebadf0ba90befda7366d52cbc757'
            '96c57bc809767e1e8c88bfaa2c63f1c2d936853b1041339489939f15d455bd19'
            '77e89ada90f43acdedc5787e3a044d2662db3f94108d7002d0da079cb940fe10'
            '02f1d8818f0fab6b0aba855234cd0586a1feb53002c0e762233436fd2e6bf4f8'
            '0fc340700cb5d72d95ac1272fc66c581db3ad1db7f09a6c9e4bf0b08a52e52ac')

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
        patch --directory="${_srcname}" --forward --strip=1 --input="${srcdir}/${_patchfile}"
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
}
