# Maintainer: Zile995 <stefan.zivkovic995@gmail.com>

pkgname=booster-um-git
pkgver=1.1.r14.g1a753d9
pkgrel=1
pkgdesc="Booster UKI Manager - Simple bash script to manage booster initramfs, vmlinuz and UKI files"
url="https://github.com/Zile995/booster-um"
arch=('any')
license=('GPL3')
backup=(etc/booster-um.yaml)
depends=('booster' 'go-yq' 'systemd' 'systemd-ukify')
optdepends=("sbctl: Sign UKI files")
makedepends=('git')
source=("${pkgname}::git+$url")
md5sums=('SKIP')

pkgver() {
  cd "$pkgname"
  git describe --long --tags --abbrev=7 | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

package() {
  cd "$pkgname"

  # Create config file
  mkdir "${pkgdir}/etc/"
  touch "${pkgdir}/etc/booster-um.yaml"

  # Install the booster-um
  install -Dm0755 booster-um -t "${pkgdir}/usr/bin"

  # Install the scripts
  install -Dm0755 scripts/booster-um-remove  -t "${pkgdir}/usr/share/libalpm/scripts"
  install -Dm0755 scripts/booster-um-install -t "${pkgdir}/usr/share/libalpm/scripts"

  # Install the hooks
  install -Dm0644 hooks/90-booster-um-install.hook      -t "${pkgdir}/usr/share/libalpm/hooks"
  install -Dm0644 hooks/60-booster-um-remove.hook       -t "${pkgdir}/usr/share/libalpm/hooks"
  install -Dm0644 hooks/60-booster-um-ucode-remove.hook -t "${pkgdir}/usr/share/libalpm/hooks"

  # Install the LICENSE
  install -Dm0644 LICENSE -t "${pkgdir}/usr/share/licenses/${pkgname}"
}
