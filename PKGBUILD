# Maintainer: adttsonaisse <adttsonaisse@users.noreply.github.com>
# Repack dari .deb resmi dsh-tauri/deepseek-harness-desktop. Tanpa debtap.

pkgname=deepseek-harness-desktop-bin
pkgver=0.34.0
pkgrel=1
pkgdesc='DeepSeek Harness desktop application (repack dari .deb resmi)'
arch=('x86_64')
url='https://github.com/dsh-tauri/deepseek-harness-desktop'
license=('MIT')
depends=(
  'gtk3'
  'webkit2gtk-4.1'
  'libappindicator'
  'systemd-libs'
  'zenity'
  'openssl'
)
conflicts=('deepseek-harness-desktop')
options=('!strip' '!debug')
noextract=("Deepseek.Harness.Desktop-${pkgver}_amd64.deb")
source=("Deepseek.Harness.Desktop-0.34.0_amd64.deb::https://github.com/dsh-tauri/deepseek-harness-desktop/releases/download/v0.34.0/Deepseek.Harness.Desktop_0.34.0_amd64.deb")
sha256sums=('ccffe901afbe08d9da5029823c697a7d715cd4d29041b5798f9a55dde617d059')

_deb="Deepseek.Harness.Desktop-${pkgver}_amd64.deb"

prepare() {
  cd "$srcdir"

  # File .deb adalah arsip ar: keluarkan control.tar.* dan data.tar.*
  bsdtar -xf "$_deb"
}

package() {
  cd "$srcdir"

  if ! compgen -G 'data.tar.*' > /dev/null; then
    echo 'data.tar.* tidak ditemukan setelah ekstraksi .deb' >&2
    return 1
  fi

  bsdtar -xf data.tar.* -C "$pkgdir"

  # Direktori 755, file yang memang executable di .deb tetap executable.
  find "$pkgdir" -type d -exec chmod 755 {} +
  find "$pkgdir" -type f -perm /111 -exec chmod a+x {} +

  # File .desktop dan aset gambar tidak boleh executable.
  find "$pkgdir" -type f \( -name '*.desktop' -o -name '*.png' -o -name '*.svg' -o -name '*.json' \) -exec chmod 644 {} +
}
