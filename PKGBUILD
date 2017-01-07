# $Id$
# Maintainer: Tom Gundersen <teg@jklm.no>
# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Contributor: judd <jvinet@zeroflux.org>
# SELinux Maintainer: Nicolas Iooss (nicolas <dot> iooss <at> m4x <dot> org)
# SELinux Contributor: Timothée Ravier <tim@siosm.fr>
# SELinux Contributor: Nicky726 <nicky726@gmail.com>

pkgbase=util-linux-selinux
pkgname=(util-linux-selinux libutil-linux-selinux)
_pkgmajor=2.29
pkgver=${_pkgmajor}
pkgrel=2
pkgdesc="SELinux aware miscellaneous system utilities for Linux"
url="https://www.kernel.org/pub/linux/utils/util-linux/"
arch=('i686' 'x86_64')
groups=('selinux')
# SELinux package maintenance note:
#   ArchLinux base packages have a build-time cyclic dependency because
#   systemd depends on libutil-linux and util-linux depends on libudev
#   provided by libsystemd (FS#39767).  To break this cycle, make
#   util-linux-selinux depend on systemd at build time.
makedepends=('systemd' 'python' 'libselinux')
license=('GPL2')
options=('strip' 'debug')
validpgpkeys=('B0C64D14301CC6EFAEDF60E4E4B71D5EEC39C284')  # Karel Zak
source=("https://www.kernel.org/pub/linux/utils/util-linux/v$_pkgmajor/${pkgbase/-selinux}-$pkgver.tar."{xz,sign}
        pam-{login,common,su}
        '0001-sfdisk-don-t-be-silent-when-list-non-existing-device.patch'
        '0001-sfdisk-cleanup-dump-error-messages.patch'
        '0001-sfdisk-support-empty-label-use-case.patch'
        '0001-chrt-default-to-SCHED_RR-policy.patch'
        '0001-lsns-Fix-parser-for-proc-pid-stat-which-is-including.patch')
md5sums=('07b6845f48a421ad5844aa9d58edb837'
         'SKIP'
         '4368b3f98abd8a32662e094c54e7f9b1'
         'a31374fef2cba0ca34dfc7078e2969e4'
         'fa85e5cce5d723275b14365ba71a8aad'
         '3fce7192ce1b3d97fdffd0226ed63a90'
         '2f3c061865360170cacda948035fd02d'
         '6d2e3915124938577f0ff18ef701c87f'
         '168c1cb2cfe7d4eddfc6e3f3b19d3ced'
         '68c2076a9a09564ba0c9776540a175fa')

prepare() {
  cd "${pkgbase/-selinux}-$pkgver"

  patch -Np1 <../0001-sfdisk-don-t-be-silent-when-list-non-existing-device.patch
  patch -Np1 <../0001-sfdisk-cleanup-dump-error-messages.patch
  patch -Np1 <../0001-sfdisk-support-empty-label-use-case.patch
  patch -Np1 <../0001-chrt-default-to-SCHED_RR-policy.patch
  patch -Np1 <../0001-lsns-Fix-parser-for-proc-pid-stat-which-is-including.patch
}

build() {
  cd "${pkgbase/-selinux}-$pkgver"

  ./configure --prefix=/usr \
              --libdir=/usr/lib \
              --bindir=/usr/bin \
              --localstatedir=/run \
              --enable-fs-paths-extra=/usr/bin \
              --enable-raw \
              --enable-vipw \
              --enable-newgrp \
              --enable-chfn-chsh \
              --enable-write \
              --enable-mesg \
              --disable-tailf \
              --with-selinux \
              --with-python=3

  make
}

package_util-linux-selinux() {
  conflicts=('util-linux-ng' 'eject' 'zramctl'
             "${pkgname/-selinux}" "selinux-${pkgname/-selinux}")
  provides=("util-linux-ng=$pkgver" 'eject' 'zramctl'
            "${pkgname/-selinux}=${pkgver}-${pkgrel}"
            "selinux-${pkgname/-selinux}=${pkgver}-${pkgrel}")
  depends=('pam-selinux' 'shadow-selinux' 'coreutils-selinux'
           'libsystemd-selinux' 'libutil-linux-selinux')
  optdepends=('python: python bindings to libmount')
  backup=(etc/pam.d/chfn
          etc/pam.d/chsh
          etc/pam.d/login
          etc/pam.d/su
          etc/pam.d/su-l)

  cd "${pkgbase/-selinux}-$pkgver"

  make DESTDIR="$pkgdir" install

  # setuid chfn and chsh
  chmod 4755 "$pkgdir"/usr/bin/{newgrp,ch{sh,fn}}

  # install PAM files for login-utils
  install -Dm644 "$srcdir/pam-common" "$pkgdir/etc/pam.d/chfn"
  install -m644 "$srcdir/pam-common" "$pkgdir/etc/pam.d/chsh"
  install -m644 "$srcdir/pam-login" "$pkgdir/etc/pam.d/login"
  install -m644 "$srcdir/pam-su" "$pkgdir/etc/pam.d/su"
  install -m644 "$srcdir/pam-su" "$pkgdir/etc/pam.d/su-l"

  # TODO(dreisner): offer this upstream?
  sed -i '/ListenStream/ aRuntimeDirectory=uuidd' "$pkgdir/usr/lib/systemd/system/uuidd.socket"

  # adjust for usrmove
  # TODO(dreisner): fix configure.ac upstream so that this isn't needed
  cd "$pkgdir"
  mv {,usr/}sbin/* usr/bin
  rmdir sbin usr/sbin

  ### runtime libs are shipped as part of libutil-linux
  rm "$pkgdir"/usr/lib/lib*.{a,so}*

  ### tailf has been deprecated for a while. let's not include it anymore.
  rm \
    "$pkgdir"/usr/bin/tailf \
    "$pkgdir"/usr/share/bash-completion/completions/tailf \
    "$pkgdir"/usr/share/man/man1/tailf.1
}

package_libutil-linux-selinux() {
  pkgdesc="util-linux-selinux runtime libraries"
  provides=('libblkid.so' 'libfdisk.so' 'libmount.so' 'libsmartcols.so' 'libuuid.so'
            "${pkgname/-selinux}=${pkgver}-${pkgrel}")
  depends=('libselinux')
  conflicts=("${pkgname/-selinux}")

  make -C "${pkgbase/-selinux}-$pkgver" DESTDIR="$pkgdir" install-usrlib_execLTLIBRARIES
}
