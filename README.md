# PHP 8.5 for Synology DSM 7 (Web Station backend)

Unofficial package of **PHP 8.5.12** for Synology NAS running **DSM 7.1 or newer**. After you install it,
**PHP 8.5** appears in Web Station next to Synology's own PHP versions, and you can switch the
extensions on and off per profile, the same way as with Synology's PHP.

Current release: **8.5.12-7** · Author: [dev-valley](https://github.com/dev-valley/)

**Download:** [github.com/dev-valley/php85-sbk/releases/latest](https://github.com/dev-valley/php85-sbk/releases/latest)
(all versions: [Releases](https://github.com/dev-valley/php85-sbk/releases))

> Not affiliated with or supported by Synology. Use at your own risk, and keep a backup.

## Which file for which NAS

Check your model's CPU or platform name at
[Synology: What kind of CPU does my NAS have?](https://kb.synology.com/en-global/DSM/tutorial/What_kind_of_CPU_does_my_NAS_have).
Package Center refuses a package built for the wrong architecture. Download the matching file from the
[latest release](https://github.com/dev-valley/php85-sbk/releases/latest).

| File | CPU | Platforms | Example models |
|---|---|---|---|
| `php85_x64-7.1_8.5.12-7.spk` | Intel / AMD 64-bit | apollolake, avoton, braswell, broadwell(nk/nkv2/ntbap), bromolow, cedarview, denverton, epyc7002/7003(ntb), geminilake(nk), grantley, icelaked, kvmx64, purley, r1000(nk), v1000(nk) | DS918+, DS920+, DS923+, DS1517+, DS1621+, DS1821+, DS224+, DS423+, RS-series, Virtual DSM |
| `php85_aarch64-7.1_8.5.12-7.spk` | ARMv8 64-bit | rtd1296, rtd1619b, armada37xx | DS118, DS218, DS418, DS124, DS223, DS223j, DS423, DS120j, DS220j, DS420j |
| `php85_armv7-7.1_8.5.12-7.spk` | ARMv7 32-bit | alpine, alpine4k, armada370, armada375, armada38x, armadaxp, monaco | DS216j, DS218j, DS418j, DS416, DS216play, DS215j |

Model names are examples only. The platform name is what decides.

**Tested (x64):**

| Model | DSM |
|---|---|
| RS818RP+ | 7.2.2 |
| RS820+ | 7.3.2 |
| DS1517+ | 7.2.2 (Web Station integration, all extensions, imagick, benchmark below) |

**ARM packages:** built from the same recipe, but **not tested on real hardware**, so consider them
beta. Feedback is welcome.

SHA-256 (also attached to the release as `SHA256SUMS-8.5.12-7.txt`):
```
9c8b1dec27fe88e69346c2c7a720e2543d7c3d0f4917e0bddf435ae302991e15  php85_x64-7.1_8.5.12-7.spk
638b624d3cbb9682641f4dfa614333f633be568dc67b84e812a8317a2936c5b0  php85_aarch64-7.1_8.5.12-7.spk
0d5d588b80759e7d1ba070309397f30b129d11c92d3ef3f37d61c76b00a523ef  php85_armv7-7.1_8.5.12-7.spk
```

## Installation

1. Install **Web Station** from Package Center.
2. Package Center → **Manual Install** → choose the spk for your CPU. The package isn't signed, so DSM
   shows a warning about the publisher. Confirm it to continue.
3. Web Station → **Script Language Settings** → **Create** → choose **PHP 8.5**.
4. Tick the extensions you need. If an app needs **APCu**, tick **apcu** in the extension list. For the
   two switches at the top of the profile:
   - **"Enable PHP cache": leave it off.** OPcache is always built in and active since PHP 8.5, whatever
     this switch says. The switch loads APCu **and** tries to load OPcache as a separate module, which
     doesn't exist in 8.5. With the switch on, every PHP-FPM start writes a harmless
     `Failed loading Zend extension 'opcache.so'` warning to the log. Ticking apcu **and** turning on
     the switch also logs `Module "apcu" is already loaded`.
   - **"Enable Xdebug": leave it off.** Xdebug isn't included. Turning it on does nothing except write a
     `Failed loading Zend extension 'xdebug.so'` warning to the log.

   Neither warning is shown to visitors (`display_errors` is off).
5. Assign the profile to a web service or portal.

To update, install a newer spk over the existing one without uninstalling first. Your profiles and
their extension selection are kept. Extensions added in a new release show up unticked.

If PHP 8.5 doesn't appear in Web Station, re-register it via SSH:
`sudo /var/packages/WebStation/target/tools/synowebservice plugin-pkg sync php85`

## What's included

- `php85` (CLI), `php85-cgi`, and `php-fpm`, which Web Station uses
- Besides Web Station, the package runs its own PHP-FPM pool on **127.0.0.1:9085** (localhost only), for
  reverse-proxy or CLI setups.

**Compiled in (always on):** Core, ctype, date, dom, fileinfo, filter, hash, json, lexbor, libxml,
mbstring, mysqlnd, pcntl, pcre (with JIT), PDO, random, Reflection, session, SimpleXML, SPL, standard,
tokenizer, uri, xml, xmlreader, xmlwriter, Zend OPcache

**Shared extensions (switchable in Web Station's extension list, 38):** apcu 5.1.28, bcmath, bz2, calendar,
curl, dba (with cdb/flatfile/inifile), exif, ftp, gd, gettext, gmp, iconv, **imagick 3.8.1**, intl, ldap,
mailparse 3.2.0, memcached 3.4.0, mysqli, openssl, pdo_dblib, pdo_mysql, pdo_pgsql, pdo_sqlite, pgsql,
phar, posix, shmop, soap, sockets, sodium, sqlite3, ssh2 1.5.0, sysvmsg, sysvsem, sysvshm, xsl, zip, zlib

Not included: readline (interactive shell only), xdebug (debugger, not meant for production servers), redis, igbinary.

### imagick / ImageMagick

- Bundled **ImageMagick 7.1.2 Q8** (same colour depth as Synology's own imagick), without OpenMP, so
  it's safe inside PHP-FPM workers.
- Formats: JPEG, PNG, GIF, WebP and TIFF can be read and written. **HEIC and AVIF can be read only**
  (decoders libde265 and dav1d, no encoders). Synology's own PHP 8.4 can't write them either.
- No PDF, PostScript or SVG (no Ghostscript or librsvg).
- **Security policy** (ImageMagick's "websafe" policy) in
  `/var/packages/php85/target/etc/ImageMagick-7/policy.xml`: external delegates and the MVG, MSL and URL
  coders are blocked. Limits: 256 MiB RAM, 16,000 px per side, 128 megapixels, 60 s per image. Larger
  images are rejected with a "resource limit" error. Adjust the file if you really need more.
  Updates overwrite it.

## How it was built

- **Framework:** [SynoCommunity spksrc](https://github.com/SynoCommunity/spksrc), with Synology's
  DSM 7.1 toolchains (GCC 8.5.0) for `x64-7.1`, `aarch64-7.1` and `armv7-7.1`
- **Flags:** `-O2` for PHP and every bundled library, without `-march`, so it runs on every CPU of the
  architecture. Release build: no debug, NTS (non-thread-safe), like Synology's own PHP.
- **Extensions** are built with `=shared`. PECL extensions (APCu, imagick, mailparse, memcached, ssh2) are
  built in-tree from their release sources. memcached is built without SASL, igbinary or msgpack.
- **Libraries are bundled** in `/var/packages/php85/target/lib` and found via RPATH, so the package
  doesn't depend on DSM's system libraries:
  OpenSSL 3.5.9, ICU 78.3, curl 8.22.0, libxml2 2.15.4, libxslt 1.1.45, libsodium 1.0.20,
  libzip 1.12.1, SQLite 3.53, ImageMagick 7.1.2, libheif 1.20.2, libde265, dav1d, lcms2, libpq
  (PostgreSQL 17), OpenLDAP (without SASL), libmemcached-awesome, libssh2, FreeTDS, GMP, libgcrypt,
  libjpeg, libpng, libwebp, freetype, libtiff
- **Web Station integration:** INFO `install_provide_packages="PHP"` plus `target/misc/` (`meta.json`,
  `php-fpm.ini`, `default_settings.json`, `extension_list.json`), modeled on Synology's PHP 8.4
  package. `opcache.jit` is disabled by default, as in Synology's packages.

**Performance:** on a DS1517+ (Atom C2538), a CPU benchmark (function calls, arrays, strings, regex, JSON)
runs as fast as Synology's PHP 8.4.27, within measurement noise (about 2.4–2.6 s for both).

## Changelog

- **8.5.12-7:** APCu is listed in Web Station's extension list (tick apcu instead of using the
  "Enable PHP cache" switch). No code changes.
- **8.5.12-6:** imagick 3.8.1 with bundled ImageMagick 7 Q8 and websafe policy
- **8.5.12-5:** built with `-O2` (earlier builds were unoptimized and about 3× slower). Added soap, dba,
  mailparse, pgsql, pdo_pgsql, ldap, memcached, ssh2 and pdo_dblib.
- **8.5.12-4:** extensions as switchable shared modules, php85-cgi, APCu
- **8.5.12-3:** first version usable as a Web Station backend

## Licenses

PHP is distributed under the PHP License. The bundled extensions and libraries are under their own
licenses (OpenSSL: Apache 2.0, ICU: Unicode, curl: curl license, ImageMagick: ImageMagick license,
GMP/libheif/libde265/FreeTDS: LGPL, etc.).
