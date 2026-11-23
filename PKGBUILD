# Maintainer: adttsonaisse <adttsonaisse@users.noreply.github.com>
# Paket dari tarball npm resmi @deepseek-ai/dsh (repo ini tidak merilis .deb).

pkgname=deepseek-harness-bin
pkgver=0.2.2.rc5
pkgrel=1
pkgdesc='DeepSeek Harness (dsh): everything-is-a-plugin agent harness CLI'
arch=('any')
url='https://github.com/deepseek-ai/deepseek-harness'
license=('MIT')
depends=('nodejs>=24.9.0')
makedepends=('npm')
options=('!strip')

# pkgver Arch tidak boleh memakai '-', jadi versi npm disimpan sebagai
# 0.2.2.rc5 dan dikembalikan ke 0.2.2-rc.5 di sini.
_npmver=$(printf '%s' "$pkgver" | sed -E 's/\.(alpha|beta|rc)([0-9]+)$/-\1.\2/')

source=("dsh-${_npmver}.tgz::https://registry.npmjs.org/@deepseek-ai/dsh/-/dsh-0.2.2-rc.5.tgz")
sha256sums=('67da5a3c458f46cd278e0eec6ad0e3a44e99fd13087c6f11a1b362268900a52c')

package() {
  cd "$srcdir"

  # Pasang CLI + semua dependency npm-nya ke prefix /usr. Bin /usr/bin/dsh
  # dibuat npm sebagai symlink relatif ke lib/node_modules.
  npm install --global --prefix "$pkgdir/usr" \
    --cache "$srcdir/.npm-cache" \
    --ignore-scripts \
    "$srcdir/dsh-${_npmver}.tgz"

  chmod 755 "$pkgdir/usr/bin/dsh"
}
