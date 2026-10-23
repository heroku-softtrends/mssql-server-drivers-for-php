# MS-SQL Server Driver For Ubuntu (20.04, 22.04, 24.04, 26.04) PHP Applications

Pre-built `sqlsrv` / `pdo_sqlsrv` extensions, ODBC `Aptfile`s, and Heroku
installer `packages.json` manifests for running the Microsoft SQL Server PHP
drivers on Heroku. Each driver version is split by Ubuntu stack and PHP
version, matching Heroku's own stack/PHP support matrix.

## Available versions

| Driver  | Ubuntu stack           | Heroku stack | PHP versions   | msodbcsql |
|---------|-------------------------|--------------|----------------|-----------|
| 5.9.0   | 20.04                   | heroku-20    | 7.4, 8.0       | 17        |
| 5.10.0  | 20.04                   | heroku-20    | 7.4, 8.0, 8.1  | 17        |
| 5.11.1  | 22.04                   | heroku-22    | 8.1, 8.2       | 17, 18    |
| 5.13.3  | 24.04                   | heroku-24    | 8.3, 8.4, 8.5  | 17, 18    |
| 5.13.3  | 26.04                   | heroku-26    | 8.4, 8.5       | 18 only   |

All builds are **NTS** (non-thread-safe), matching Heroku's PHP buildpack.

## Folder layout

```
<driver-version>/ubuntu/<ubuntu-version>/
├── msodbcsql17/Aptfile          # ODBC Driver 17 + unixODBC, as .deb URLs
├── msodbcsql18/Aptfile          # ODBC Driver 18 + unixODBC, as .deb URLs
└── php/<php-version>/nts/
    ├── ext-sqlsrv-<driver>_php-<php>.tar.gz       # lib/php/extensions/no-debug-non-zts-<api>/sqlsrv.so
    ├── ext-pdo_sqlsrv-<driver>_php-<php>.tar.gz    # same, pdo_sqlsrv.so
    └── packages.json                               # heroku-sys/ext-* manifest for the Heroku PHP installer
```

Point your app's `composer.json` `repositories` at the relevant
`packages.json` and require `heroku-sys/ext-sqlsrv` /
`heroku-sys/ext-pdo_sqlsrv`, and add one of the `msodbcsql*/Aptfile`s via
`heroku-community/apt`, per the Heroku PHP driver-extension pattern this
repo already uses.

---

## 5.9.0 / 5.10.0 (Ubuntu 20.04) and 5.11.1 (Ubuntu 22.04)

Pre-existing in this repo. Not changed or re-verified as part of this
update — see each folder's own `Aptfile`/`packages.json` for exact package
versions in use.

## 5.13.3 (Ubuntu 24.04, PHP 8.3 / 8.4 / 8.5)

Added to support heroku-24, Heroku's current default stack.

**Verified:**
- Driver binaries are genuine, unmodified Microsoft builds, pulled directly
  from `microsoft/msphpsql` release `v5.13.3`
  (`Linux_5.13.3RTW.tar.gz`, the official Linux binary bundle).
- PHP extension API numbers (the `no-debug-non-zts-XXXXXXXX` directory PHP
  uses to find a compatible `.so`) were read directly out of each binary's
  embedded build string, not guessed:
  - PHP 8.3 → `20230831`
  - PHP 8.4 → `20240924`
  - PHP 8.5 → `20250925`
- `msodbcsql17`/`msodbcsql18` package URLs point at the current latest
  builds Microsoft publishes for Ubuntu 24.04 (checked live against
  `packages.microsoft.com`): `msodbcsql17_17.11.1.1-1` (only version MS
  ships for 24.04) and `msodbcsql18_18.7.1.1-1` (latest of several).

**Not verified — test before relying on this:**
- The Aptfile's unixODBC package set is a best-effort reconstruction, not a
  tested deploy. Ubuntu 24.04 restructured these packages from what the
  20.04/22.04 Aptfiles in this repo use: `libodbc1` → `libodbc2`,
  `odbcinst1debian2` dropped, `libodbcinst2` added. The Aptfile points at
  `libodbc2`, `libodbcinst2`, `odbcinst`, `unixodbc-dev`
  (`2.3.12-1ubuntu0.24.04.1`, confirmed to exist on `archive.ubuntu.com`),
  but this hasn't been run through `heroku buildpacks:add
  heroku-community/apt` against a live heroku-24 dyno.
- No live connection test was run — binaries were verified as authentic and
  structurally correct, not confirmed to connect to a database.

## 5.13.3 (Ubuntu 26.04, PHP 8.4 / 8.5)

Added for heroku-26. **On weaker footing than the 24.04 build above** —
Microsoft's msphpsql has not announced Ubuntu 26.04 support (still at
v5.13.3, no 26.04 mention in any 5.13.x changelog). This bundle exists on
inference, not vendor confirmation:

- The same `.so` files from the 24.04 bundle are reused here — `readelf`
  shows they only require up to `GLIBC_2.34` / `GLIBCXX_3.4.29` and link
  against `libodbc.so.2` / `libodbcinst.so.2`, nothing 24.04-specific.
  glibc/libstdc++ are backward-compatible, so this should load on 26.04,
  but that's inference from the binary, not a confirmed run.
- `msodbcsql18` for Ubuntu 26.04 **is** genuinely published by Microsoft
  (`packages.microsoft.com/ubuntu/26.04/...`, confirmed live), same version
  as 24.04 (`18.7.1.1-1`).
- `msodbcsql17` is **not** published for 26.04 at all (404 on Microsoft's
  repo) — only an 18 Aptfile exists here.
- unixODBC packages: no `ubuntu0.26.04.x`-tagged build was found on
  `archive.ubuntu.com` the way 24.04 has one. The Aptfile uses the newest
  plain version available there (`2.3.14-2`) as a reasonable guess, not a
  confirmed 26.04-tagged package.
- PHP scope is 8.4/8.5 only because
  [Heroku's PHP support page](https://devcenter.heroku.com/articles/php-support)
  doesn't offer PHP 8.1–8.3 on heroku-26 at all.

**Deploy-test this on a real heroku-26 app before production use.**
