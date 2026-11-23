# deepseek-harness-repo

Repository pacman untuk Arch Linux yang berisi satu paket,
`deepseek-harness-desktop-bin` — versi **desktop** dari DeepSeek Harness.

Paket ini dibuat dengan mengemas ulang file `.deb` resmi dari
[dsh-tauri/deepseek-harness-desktop](https://github.com/dsh-tauri/deepseek-harness-desktop)
menjadi `.pkg.tar.zst`. Tidak ada kompilasi dari source dan tidak memakai
`debtap` — cukup ekstrak `data.tar.*` memakai `bsdtar`.

Satu workflow GitHub Actions jalan tiap hari (dan bisa dipicu manual), lalu:

1. cek rilis terbaru upstream lewat GitHub API;
2. kalau versinya berubah, perbarui `pkgver`, `source`, dan `sha256sums` di `PKGBUILD`;
3. build di container `archlinux:base-devel` sebagai user non-root;
4. buat database pacman dengan `repo-add`;
5. ubah symlink `.db` dan `.files` jadi file biasa (GitHub Releases tidak menyimpan symlink);
6. upload paket + database ke **satu tag tetap bernama `repo`**;
7. commit `PKGBUILD` ke branch `main`.

Karena tag-nya tetap, URL di `pacman.conf` tidak pernah berubah.

## Install

### 1. Tambahkan repo ke `/etc/pacman.conf`

```ini
[deepseek-harness-repo]
SigLevel = Optional TrustAll
Server = https://github.com/adttsonaisse/deepseek-harness-repo/releases/download/repo
```

`SigLevel = Optional TrustAll` diperlukan karena database dan paket di repo ini
tidak ditandatangani kunci PGP. Hanya pakai kalau kamu percaya pada repo ini.

### 2. Sinkronkan dan pasang

```bash
sudo pacman -Syu deepseek-harness-desktop-bin
```

### 3. Jalankan

```bash
deepseek-harness-desktop
```

Atau lewat menu aplikasi (namanya **DSH Tauri**).

Paket ini hanya untuk `x86_64` / `amd64`, mengikuti asset `.deb` upstream.

## Update

Cukup:

```bash
sudo pacman -Syu
```

Kalau cron GitHub telat (wajar pada repo publik), buka tab **Actions** dan klik
**Run workflow**, lalu jalankan `sudo pacman -Syu` lagi.

## Isi release `repo`

| File | Fungsi |
| --- | --- |
| `deepseek-harness-repo.db` | database pacman (isi sama dengan `.db.tar.gz`) |
| `deepseek-harness-repo.db.tar.gz` | database yang dipakai pacman |
| `deepseek-harness-repo.files` | indeks daftar file paket |
| `deepseek-harness-repo.files.tar.gz` | versi terkompresi indeks tersebut |
| `deepseek-harness-desktop-bin-<versi>-1-x86_64.pkg.tar.zst` | paketnya sendiri |

File `.db` dan `.files` sengaja disalin dari `*.tar.gz` menjadi file biasa.
`repo-add` awalnya membuat keduanya sebagai symlink, tetapi GitHub Releases
mengabaikan symlink saat upload.

## Build manual (opsional)

```bash
git clone https://github.com/adttsonaisse/deepseek-harness-repo.git
cd deepseek-harness-repo
makepkg -si
```

Butuh `base-devel` dan `bsdtar` (sudah termasuk di `base`). `makepkg` tidak
boleh dijalankan sebagai root.

Catatan: paket ini meng-`conflicts` dengan `deepseek-harness-desktop` dari AUR
karena memiliki path file yang sama. Lepaskan versi AUR dulu sebelum memasang
paket dari repo ini.

Ini adalah repack pihak ketiga, bukan rilis resmi DeepSeek.
