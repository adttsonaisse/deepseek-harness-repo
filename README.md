# deepseek-harness-repo

Repository pacman untuk Arch Linux yang berisi satu paket, `deepseek-harness-bin`
(perintah CLI `dsh`).

Paket ini dibuat dari tarball npm resmi [`@deepseek-ai/dsh`](https://www.npmjs.com/package/@deepseek-ai/dsh),
yaitu kanal distribusi [`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness).
Repo upstream **tidak merilis file `.deb`** — rilisnya hanya berupa tag tanpa asset,
jadi pengemasannya mengikuti cara distribusi aslinya lewat npm.

Satu workflow GitHub Actions jalan tiap hari (dan bisa dipicu manual), lalu:

1. baca dist-tag `latest` dari registry npm;
2. kalau versinya berubah, normalisasi versi ke format `pkgver` Arch, lalu perbarui
   `pkgver`, `source`, dan `sha256sums` di PKGBUILD (reset `pkgrel` ke `1`);
3. build di container `archlinux:base-devel` sebagai user non-root, memasang CLI
   beserta dependency-nya ke `/usr/lib/node_modules`;
4. buat database pacman dengan `repo-add`;
5. ubah symlink `.db` dan `.files` jadi file biasa (GitHub Releases tidak menyimpan symlink);
6. upload paket + database ke **satu tag tetap bernama `repo`**.

Karena tag-nya tetap, URL di `pacman.conf` tidak pernah berubah.

## Install

### 1. Tambahkan repo ke `/etc/pacman.conf`

```ini
[deepseek-harness-repo]
SigLevel = Optional TrustAll
Server = https://github.com/adttsonaisse/deepseek-harness-repo/releases/download/repo
```

`SigLevel = Optional TrustAll` diperlukan karena paket dan database di repo ini
tidak ditandatangani kunci PGP. Hanya pakai kalau kamu percaya pada repo ini.

### 2. Sinkronkan dan pasang

```bash
sudo pacman -Syu deepseek-harness-bin
```

### 3. Pakai

```bash
dsh web
```

Perintah itu menyalakan Web UI di `http://127.0.0.1:3080`. Tambahkan `--no-open`
kalau tidak mau browser terbuka otomatis.

## Update

Cukup:

```bash
sudo pacman -Syu
```

Kalau cron GitHub telat (hal yang wajar pada repo publik), buka tab **Actions**
repo ini dan klik **Run workflow**, lalu jalankan `sudo pacman -Syu` lagi.

## Catatan versi

Versi npm seperti `0.2.2-rc.5` tidak sah sebagai `pkgver` Arch (hyphen dilarang),
jadi disimpan sebagai `0.2.2.rc5`. PKGBUILD mengubahnya kembali ke bentuk npm
saat mengunduh tarball. Untuk versi `alpha`/`beta` polanya sama:
`0.2.3-alpha.2` → `0.2.3.alpha2` → `0.2.3-alpha.2`.

Workflow mengikuti dist-tag **`latest`**, bukan `alpha`. Dist-tag `alpha`
mungkin lebih baru tetapi belum menjadi versi default `npx`.

## Isi release `repo`

| File | Fungsi |
| --- | --- |
| `deepseek-harness-repo.db` | database pacman (isi sama dengan `.db.tar.gz`) |
| `deepseek-harness-repo.db.tar.gz` | database yang dipakai pacman |
| `deepseek-harness-repo.files` | indeks daftar file paket |
| `deepseek-harness-repo.files.tar.gz` | versi terkompresi indeks tersebut |
| `deepseek-harness-bin-<versi>-<rel>-any.pkg.tar.zst` | paketnya sendiri |

`arch=('any')` karena paket ini membawa prebuild native untuk Linux x86_64
dan aarch64 sekaligus, sehingga bisa dipakai di keduanya. Dependency native
hanya memakai `glibc`/`gcc-libs` yang sudah termasuk base Arch.

## Build manual (opsional)

```bash
git clone https://github.com/adttsonaisse/deepseek-harness-repo.git
cd deepseek-harness-repo
makepkg -si
```

Butuh `base-devel`, `nodejs` (>= 24.9.0), dan `npm`. `makepkg` tidak boleh
dijalankan sebagai root.

Ini adalah pengemasan pihak ketiga, bukan rilis resmi DeepSeek.
