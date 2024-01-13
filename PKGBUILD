# Maintainer: Zile995 <stefan.zivkovic995@gmail.com>

pkgname=booster-um-git
pkgver=1.5.1.r0.g6581394
pkgrel=1
pkgdesc="Booster UKI Manager - A simple bash script to manage UKI files generated by booster and systemd-ukify"
url="https://github.com/Zile995/booster-um"
arch=('any')
license=('GPL3')
provides=("${pkgname%-*}")
conflicts=("${pkgname%-*}")
backup=(etc/booster-um.yaml)
depends=('booster' 'go-yq' 'systemd' 'systemd-ukify' 'util-linux')
optdepends=("sbctl: Sign the UKI files"
            "sbsigntools: Sign the UKI files with sbsign"
            "efibootmgr: Manage EFI entries")
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

  # Install the helper scripts
  for script in scripts/*; do
    install -Dm0755 "$script" -t "${pkgdir}/usr/share/booster-um"
  done

  # Install the libaplm scripts
  for libalpm_script in libalpm/scripts/*; do
    install -Dm0755 "$libalpm_script" -t "${pkgdir}/usr/share/libalpm/scripts"
  done

  # Install the libalpm hooks
  for libalpm_hook in libalpm/hooks/*; do
    install -Dm0644 "$libalpm_hook" -t "${pkgdir}/usr/share/libalpm/hooks"
  done

  # Install completions
  install -Dm644 "completions/zsh/_${pkgname%-*}" -t "${pkgdir}/usr/share/zsh/site-functions/"
  install -Dm644 "completions/bash/${pkgname%-*}" -t "${pkgdir}/usr/share/bash-completion/completions"

  # Install the LICENSE
  install -Dm0644 LICENSE -t "${pkgdir}/usr/share/licenses/${pkgname}"
}
