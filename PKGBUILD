# Maintainer: artoo <artoo@artixlinux.org>
# Maintainer: Chris Cromer <cromer@artixlinux.org>
# Contributor: williamh <williamh@gentoo.org>

_url=https://gitea.artixlinux.org/artix
_extra=1.3
_alpm=2.2

pkgname=openrc
pkgver=0.62.2
pkgrel=1
pkgdesc="OpenRC is a dependency-based init system that works with the system-provided init program"
arch=('x86_64')
url="https://github.com/OpenRC/openrc"
license=('BSD-2-Clause')
makedepends=('git' 'meson')
depends=(
    # 'audit' #'libaudit.so'
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
    'etc/openrc/conf.d/localmount'
    'etc/openrc/conf.d/netmount'
    'etc/openrc/conf.d/bootmisc'
    'etc/openrc/conf.d/dmesg'
    'etc/openrc/conf.d/devfs'
    'etc/openrc/conf.d/killprocs'
    'etc/openrc/conf.d/swap'
    'etc/openrc/conf.d/agetty.tty'{1,2,3}
)
source=(
    "git+${url}.git#tag=${pkgver}"
    'openrc.logrotate'
    'sysctl.conf'
    'openrc-user.pam'
    "git+${_url}/openrc-extra.git#tag=${_extra}"
    "git+${_url}/alpm-hooks.git#tag=${_alpm}"
    "openrc-agetty-meson-conf-d.patch::https://github.com/OpenRC/openrc/pull/850/commits/e3961a81809ed8d0e594402b012ee685e4ad970f.patch"
)

prepare() {
    cd "${pkgname}"
    # apply patch from the source array (should be a pacman feature)
    local src
    for src in "${source[@]}"; do
        src="${src%%::*}"
        src="${src##*/}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src"
    done
}

check(){
    meson test -C build --print-errorlogs
}

build(){
    pushd "${pkgname}"
    patch -N -p1 -i ../../mh-init.patch
    popd

    local _meson_options=()
    _meson_options+=(
        --sbindir=/usr/bin
        --libexecdir=/usr/lib
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

    arch-meson "${pkgname}" build "${_meson_options[@]}"

    meson compile -C build
}

package() {
    meson install -C build --destdir "${pkgdir}"

    install -Dm644 "${srcdir}/${pkgname}"/support/sysvinit/inittab "${pkgdir}/etc/openrc/inittab"

    # user pam
    install -d "$pkgdir"/etc/pam.d
    install -m755 "$srcdir"/openrc-user.pam "$pkgdir"/etc/pam.d/openrc-user

    install -Dm644 "${srcdir}/${pkgname}".logrotate "${pkgdir}"/etc/logrotate.d/"${pkgname}"

    # license
    install -Dm644 "${pkgname}"/LICENSE "${pkgdir}"/usr/share/licenses/"${pkgname}"/LICENSE

    ####

    install -d "${pkgdir}"/usr/lib/{openrc/cache,binfmt.d,sysctl.d}

    # sysctl defaults
    install -m755 "${srcdir}"/sysctl.conf "${pkgdir}"/usr/lib/sysctl.d/50-default.conf

    # openrc extra
    # env -C "${pkgname}-extra" patch -N -p1 -i ../../"${pkgname}"-extra.patch

    make -C "${pkgname}"-extra DESTDIR="${pkgdir}" SYSCONFDIR="/etc/openrc" \
         install_kmod install_sysusers install_tmpfiles

    # pacman hooks
    make -C alpm-hooks DESTDIR="${pkgdir}" install_openrc

    # setup misc
    while read srv lvl; do
      ln -s /etc/openrc/init.d/$srv "${pkgdir}"/etc/openrc/runlevels/$lvl
    done < ../extra/runlevels

    install -Dm644 ../extra/conf.d/* "${pkgdir}"/etc/openrc/conf.d/

    # remove suport dir
    # rm -r "${pkgdir}"/usr/share/openrc
}
