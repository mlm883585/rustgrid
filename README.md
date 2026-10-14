English | [简体中文](README_zh.md)

# RustGrid

> A Navicat-style database GUI built in Rust — native, lightweight, zero learning curve.

RustGrid is a cross-platform (Windows / macOS / Linux) database management tool built with
**Rust + GPUI**. Its layout and interactions follow Navicat, so switching costs nothing:
the connection tree, object lists, toolbars and dialogs all behave the way you already expect.

## Screenshots

<table>
  <tr>
    <td width="50%">
      <img src="docs/img/img1_en.png" width="400" alt="Connection tree and object list">
      <p align="center"><b>1.</b> Connection tree &amp; object list</p>
    </td>
    <td width="50%">
      <img src="docs/img/img2_en.png" width="400" alt="Table data grid">
      <p align="center"><b>2.</b> Table data grid — paginated, in-place editing</p>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <img src="docs/img/img3_en.png" width="400" alt="Backup and restore">
      <p align="center"><b>3.</b> Backup &amp; restore — the <code>.rgbak</code> container</p>
    </td>
    <td width="50%">
      <img src="docs/img/img4_en.png" width="400" alt="SQL editor">
      <p align="center"><b>4.</b> SQL editor — highlighting, completion, results</p>
    </td>
  </tr>
</table>

## Highlights

- **Native, not a WebView** — GPU-rendered GPUI elements, no browser engine, a single binary.
- **Navicat-compatible** — the same connection tree, object panes, toolbars and dialogs.
- **Extensible drivers** — an engine-agnostic driver layer; new engines never touch the UI.
- **Apache-2.0** — free for commercial use.

## Supported databases

| Engine                     | Status           |
| -------------------------- | ---------------- |
| MySQL / MariaDB            | Supported        |
| SQLite                     | Supported        |
| SQL Server                 | Supported        |
| PostgreSQL                 | Supported        |
| Oracle                     | Supported        |
| DB2, Dameng, ...           | via generic ODBC |

## Features

- **Connections** — create / edit / delete, connect, password prompt; passwords are encrypted
  on disk (XChaCha20-Poly1305) and never stored in a profile.
- **Databases** — enumerate, create / edit / delete with charset and collation.
- **Tables & views** — browse tables and views, a paginated grid with in-place editing, and
  drop / empty / truncate / rename operations.
- **SQL editor** — syntax highlighting, keyword/table completion, beautify, explain, and saved
  named queries.
- **Import & export wizards** — Excel, CSV and TXT.
- **Users & routines** — account and privilege management, plus stored-routine and view designers.
- **Backup & restore** — RustGrid's own `.rgbak` container.
- **Themes & languages** — light / dark / system, English / 简体中文.

## Build from source

Prerequisite: Rust stable (edition 2024; the toolchain is pinned in `rust-toolchain.toml`).

```bash
git clone https://github.com/yisier/rustgrid.git
cd rustgrid
cargo run -p rustgrid-app
```

The produced binary is named `RustGrid`.

On Windows a linker and C compiler are required (for sqlx's `ring`). With MSVC Build Tools
installed the default toolchain works; otherwise use the GNU toolchain:

```powershell
$env:RUSTUP_TOOLCHAIN="stable-x86_64-pc-windows-gnu"
$env:CARGO_TARGET_X86_64_PC_WINDOWS_GNU_LINKER="gcc"
$env:CC="gcc"
cargo run -p rustgrid-app
```

## Portable Windows offline edition

The `offline` build feature disables startup update checks, online upgrades and the GitHub link.
Database connections within an isolated network, SSH/proxy tunnels, queries, import/export and
backup remain available. Extract the archive and run `RustGrid.exe`.

| Target or connection | Requirements |
| --- | --- |
| Windows 10 Pro 20H2 **x64** | Intact Windows components and a working Direct3D graphics driver; the reviewed system APIs are available in this version |
| SQL Server, MySQL, MariaDB, PostgreSQL, SQLite, Oracle | Built-in drivers; no separate ODBC driver, database client or Oracle Instant Client |
| Databases accessed through generic ODBC, such as DB2 or Dameng | Install the vendor's **64-bit ODBC driver** offline beforehand |
| Connections using private CAs, client certificates or an Oracle wallet | Copy the required connection-specific files to the target machine and configure their local paths |

The static MSVC build below does not require a separate Visual C++ Redistributable, Rust, Java,
.NET, Windows SDK or `fxc.exe` on the target. Icons, locales, SQL syntax resources and Release
shaders are embedded; fonts come from Windows. ICU, DirectWrite and DXGI are Windows components.
Windows 10 20H2 covers the reviewed API requirements
([ICU](https://learn.microsoft.com/en-us/windows/win32/intl/international-components-for-unicode--icu-),
[DXGI](https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_6/nn-dxgi1_6-idxgifactory6),
[scrollbar settings](https://learn.microsoft.com/en-us/uwp/api/windows.ui.viewmanagement.uisettings.autohidescrollbars)).
Startup has been tested on Windows 11 x64. The Windows 10 20H2 assessment is based on binary
imports and backend source review; validate startup and the intended database connection on the target.

Configuration lives in `%APPDATA%\rustgrid\config`, not beside the executable. To migrate it,
close the app and copy the entire configuration directory, keeping encrypted secrets and
`secret.key` together. To upgrade offline, close the app and replace it with a new offline package.

### Build the offline edition

The build machine needs Rust **1.95 or later** (the locked GPUI dependencies use `cold_path`,
stabilized in 1.95), MSVC C++ tools and the Windows SDK. Run the following in an x64 Visual Studio
Developer PowerShell. These build tools are not needed on the deployment machine.

```powershell
$env:RUSTFLAGS = "-C target-feature=+crt-static"
$fxc = Get-ChildItem "${env:ProgramFiles(x86)}\Windows Kits\10\bin" -Filter fxc.exe -Recurse |
    Where-Object { $_.FullName -match '\\x64\\fxc\.exe$' } |
    Sort-Object FullName -Descending | Select-Object -First 1
if (-not $fxc) { throw "Windows SDK fxc.exe was not found" }
$env:GPUI_FXC_PATH = $fxc.FullName
cargo +1.95.0 build --locked --release --target x86_64-pc-windows-msvc -p rustgrid-app --features offline
```

The executable is `target/x86_64-pc-windows-msvc/release/RustGrid.exe`. Distribute it with `LICENSE`
and deployment instructions; generated archives belong in the ignored `dist/` directory.
Add Cargo's `--offline` flag when build dependencies are already cached. That flag only prevents
build-time downloads; `--features offline` controls the application's offline behavior.

## Project layout (Cargo workspace)

- `crates/rustgrid-core` — engine-agnostic domain: driver/connection traits and models.
- `crates/rustgrid-mysql` — MySQL and MariaDB drivers.
- `crates/rustgrid-sqlite` — SQLite driver.
- `crates/rustgrid-sqlserver` — SQL Server driver.
- `crates/rustgrid-postgresql` — PostgreSQL driver.
- `crates/rustgrid-oracle` — Oracle driver (Oracle's pure-Rust `oracledb` thin driver, no Instant Client).
- `crates/rustgrid-tunnel` — shared SSH / SOCKS5 / HTTP tunnel used by the network drivers.
- `crates/rustgrid-odbc` — generic ODBC driver (DB2, Dameng, ...).
- `crates/rustgrid-backup` — the `.rgbak` backup container.
- `crates/rustgrid-export` — table-data exporters (`.xlsx` / `.csv` / `.sql` / `.txt`).
- `crates/rustgrid-import` — Excel / CSV / TXT source readers.
- `crates/rustgrid-config` — versioned settings/profiles and encrypted secret storage.
- `crates/rustgrid-app` — the GPUI desktop application (binary `RustGrid`).

## License

Licensed under the **Apache License, Version 2.0**. Commercial use is permitted. See the
[`LICENSE`](LICENSE) file for the full text.

## Status

Early and evolving quickly — features, UI and on-disk formats may change without notice.
