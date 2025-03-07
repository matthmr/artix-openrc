# Maintainer: artoo <artoo@artixlinux.org>
# Maintainer: Chris Cromer <cromer@artixlinux.org>
# Contributor: williamh <williamh@gentoo.org>

_url=https://gitea.artixlinux.org/artix
_extras=1.2
_alpm=1.7

pkgname=openrc
pkgver=0.56
pkgrel=1
pkgdesc="Gentoo's universal init system"
arch=('x86_64')
url="https://github.com/OpenRC/openrc"
license=('BSD-2-Clause')
makedepends=('git' 'meson')
depends=(
    'bash'
    'glibc'
    'inetutils'
    'libcap' 'libcap.so'
    'pam' 'libpam.so'
    'psmisc'
    'perl'
)
optdepends=(
    'networkmanager-openrc: networkmanager init script'
    'elogind-openrc: elogind init script'
)
provides=(
    'init-rc'
    'svc-manager'
    'librc.so'
    'libeinfo.so'
)
conflicts=('init-rc' 'svc-manager')
replaces=(openrc-{deptree2dot,{bash,zsh}-completions})
backup=(
    'etc/openrc/rc.conf'
    'etc/openrc/conf.d/hostname'
    'etc/openrc/conf.d/modules'
    'etc/openrc/conf.d/hwclock'
    'etc/openrc/conf.d/etmpfiles-dev'
    'etc/openrc/conf.d/etmpfiles-setup'
    'etc/openrc/conf.d/agetty.tty'{1,2,3}
)
source=(
    "${pkgname}-${pkgver}.tar.gz::${url}/archive/refs/tags/${pkgver}.tar.gz"
    'openrc.logrotate'
    'sysctl.conf'
    "git+${_url}/openrc-extra.git#tag=${_extras}"
    "git+${_url}/alpm-hooks.git#tag=${_alpm}"
)
sha256sums=('a06b530290057637eab17fc943cbf79c0335eb734ba71ece38b9f3acd8a341d4'
            '0b44210db9770588bd491cd6c0ac9412d99124c6be4c9d3f7d31ec8746072f5c'
            '874e50bd217fef3a2e3d0a18eb316b9b3ddb109b93f3cbf45407170c5bec1d6d'
            '88c2ddad5ac5d347962ce9805a0ed7a4f1737aaafa3d6a8c0a7a55009ce5fef1'
            '6b89db32c61731ae970a7043907c08c51df3aa6b0fb9a527e70a2b6346150511')


check(){
    meson test -C build --print-errorlogs
}

build(){
    pushd "${pkgname}-${pkgver}"
    patch -N -p1 -i ../../mh-init.patch
    popd

    local _meson_options=()
    _meson_options+=(
        --sbindir=/usr/bin
        -Dbash-completions=true
        -Dbranding='"Linux"'
        -Dos=Linux
        # -Drootprefix=/usr
        -Dpam=true
        -Dpkg_prefix=''
        -Dpkgconfig=true
        -Dselinux=disabled
        -Dsysconfdir=/etc/openrc
        -Dzsh-completions=true

        --bindir=/usr/bin
        -Dshell=/bin/bash
        -Dsysvinit=true
        -Dnewnet=false
        -Daudit=disabled
    )

    arch-meson "${pkgname}-${pkgver}" build "${_meson_options[@]}"

    meson compile -C build
}

package() {
    meson install -C build --destdir "${pkgdir}"

    install -Dm644 "${srcdir}/${pkgname}-${pkgver}"/support/sysvinit/inittab "${pkgdir}/etc/openrc/inittab"

    install -Dm644 "${srcdir}/${pkgname}".logrotate "${pkgdir}"/etc/logrotate.d/"${pkgname}"

    # license
    install -Dm644 "${pkgname}-${pkgver}"/LICENSE "${pkgdir}"/usr/share/licenses/"${pkgname}"/LICENSE

    ####

    install -d "${pkgdir}"/usr/lib/{openrc/cache,binfmt.d,sysctl.d}

    # sysctl defaults
    install -m755 "${srcdir}"/sysctl.conf "${pkgdir}"/usr/lib/sysctl.d/50-default.conf

    # openrc extra
    # env -C "${pkgname}-extra" patch -N -p1 -i ../../"${pkgname}"-extra.patch

    make -C "${pkgname}"-extra DESTDIR="${pkgdir}" SYSCONFDIR="/etc/openrc" \
         install_kmod install_sysusers install_tmpfiles install_agetty

    # pacman hooks
    make -C alpm-hooks DESTDIR="${pkgdir}" install_openrc

    # setup misc
    while read srv lvl; do
      ln -s /etc/openrc/init.d/$srv "${pkgdir}"/etc/openrc/runlevels/$lvl
    done < ../extra/runlevels

    # agetty fuckery
    for i in 4 5 6; do
      rm -fv "${pkgdir}"/etc/openrc/conf.d/"agetty.tty$i"
      rm -fv "${pkgdir}"/etc/openrc/conf.d/"agetty.tty$i"
      rm -fv "${pkgdir}"/etc/openrc/runlevels/default/"agetty.tty$i"
    done

    # remove suport dir
    # rm -r "${pkgdir}"/usr/share/openrc
}
