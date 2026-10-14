[English](README.md) | 简体中文

# RustGrid

> 用 Rust 打造的 Navicat 风格数据库管理工具 —— 原生、轻量、零学习成本。

RustGrid 是一款使用 **Rust + GPUI** 构建的跨平台（Windows / macOS / Linux）数据库管理工具。
它的界面布局和交互习惯对齐 Navicat，连接树、对象列表、工具栏和弹窗都与你熟悉的操作一致，
迁移毫无成本。

## 界面预览

<table>
  <tr>
    <td width="50%">
      <img src="docs/img/img1_cn.png" width="400" alt="连接树与对象列表">
      <p align="center"><b>1.</b> 连接树与对象列表</p>
    </td>
    <td width="50%">
      <img src="docs/img/img2_cn.png" width="400" alt="表数据网格">
      <p align="center"><b>2.</b> 表数据网格 —— 分页浏览，支持就地编辑</p>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <img src="docs/img/img3_cn.png" width="400" alt="备份与还原">
      <p align="center"><b>3.</b> 备份与还原 —— <code>.rgbak</code> 容器</p>
    </td>
    <td width="50%">
      <img src="docs/img/img4_cn.png" width="400" alt="SQL 编辑器">
      <p align="center"><b>4.</b> SQL 编辑器 —— 语法高亮、补全与结果</p>
    </td>
  </tr>
</table>

## 亮点

- **原生，而非 WebView** —— GPUI 元素由 GPU 直接渲染，不依赖浏览器引擎，单个二进制。
- **兼容 Navicat** —— 相同的连接树、对象面板、工具栏与弹窗。
- **可扩展的驱动层** —— 驱动与引擎解耦，新增数据库无需改动界面。
- **Apache-2.0** —— 允许免费商用。

## 支持的数据库

| 引擎                       | 状态           |
| -------------------------- | -------------- |
| MySQL / MariaDB            | 已支持         |
| SQLite                     | 已支持         |
| SQL Server                 | 已支持         |
| PostgreSQL                 | 已支持         |
| Oracle                     | 已支持         |
| DB2、达梦 等               | 通过通用 ODBC  |

## 功能

- **连接管理** —— 新建 / 编辑 / 删除、连接、密码提示；密码在磁盘上加密存储
  （XChaCha20-Poly1305），配置文件中不保存明文。
- **数据库** —— 枚举数据库，新建 / 编辑 / 删除（字符集 + 排序规则）。
- **表与视图** —— 浏览表与视图，分页网格并支持就地编辑，以及删除 / 清空 / 截断 / 重命名。
- **SQL 编辑器** —— 语法高亮、关键字 / 表名补全、美化、解释，并可保存命名查询。
- **导入导出向导** —— Excel、CSV、TXT。
- **用户与例程** —— 账户与权限管理，以及存储例程、视图设计器。
- **备份与还原** —— RustGrid 自有的 `.rgbak` 容器。
- **主题与语言** —— 浅色 / 深色 / 跟随系统，英文 / 简体中文。

## 从源码构建

前置要求：Rust stable（edition 2024，工具链已在 `rust-toolchain.toml` 中固定）。

```bash
git clone https://github.com/yisier/rustgrid.git
cd rustgrid
cargo run -p rustgrid-app
```

生成的二进制文件名为 `RustGrid`。

在 Windows 上需要链接器和 C 编译器（sqlx 的 `ring` 依赖）。安装了 MSVC Build Tools 时
默认工具链即可，否则请使用 GNU 工具链：

```powershell
$env:RUSTUP_TOOLCHAIN="stable-x86_64-pc-windows-gnu"
$env:CARGO_TARGET_X86_64_PC_WINDOWS_GNU_LINKER="gcc"
$env:CC="gcc"
cargo run -p rustgrid-app
```

## Windows 离线绿色版

离线版通过 `offline` 构建特性关闭启动检查更新、在线升级和 GitHub 入口，保留内网数据库连接、
SSH/代理通道、查询、导入导出和备份功能。完整解压后运行 `RustGrid.exe` 即可。

| 目标环境或连接方式 | 需要准备的内容 |
| --- | --- |
| Windows 10 专业版 20H2 **64 位** | 完整的系统组件和能正常运行 Direct3D 的显卡驱动；已核查的系统接口均由该版本提供 |
| SQL Server、MySQL、MariaDB、PostgreSQL、SQLite、Oracle | 驱动已内置，无需另装 ODBC、数据库客户端或 Oracle Instant Client |
| DB2、达梦等通过通用 ODBC 连接的数据库 | 预先离线安装对应厂商的 **64 位 ODBC 驱动** |
| 使用私有 CA、客户端证书或 Oracle wallet 的连接 | 将实际连接需要的证书文件复制到内网机器，并配置本地路径 |

按下方 MSVC 静态运行库方式构建的包不需要另装 Visual C++ Redistributable、Rust、Java、.NET、
Windows SDK 或 `fxc.exe`。图标、语言、SQL 语法资源和 Release 着色器均已内嵌，字体使用系统字体。
所需的 ICU、DirectWrite、DXGI 等为 Windows 系统组件；Windows 10 20H2 已覆盖这些接口要求
（[ICU](https://learn.microsoft.com/en-us/windows/win32/intl/international-components-for-unicode--icu-)、
[DXGI](https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_6/nn-dxgi1_6-idxgifactory6)、
[系统滚动条设置](https://learn.microsoft.com/en-us/uwp/api/windows.ui.viewmanagement.uisettings.autohidescrollbars)）。
当前启动验证环境为 Windows 11 x64；Windows 10 20H2 的结论来自导入表及后端源码核查，
实际部署仍需在目标机器验证启动和目标数据库连接。

配置保存在 `%APPDATA%\rustgrid\config`，不会随程序目录自动迁移。迁移时退出程序并复制整个配置目录，
加密密码文件与 `secret.key` 需配套保留。离线升级时退出程序，再用新的离线包替换程序。

### 构建离线版

构建机需要 Rust **1.95 或更新版本**（锁定的 GPUI 依赖使用了 1.95 稳定的 `cold_path`）、
MSVC C++ 工具和 Windows SDK。以下命令在 x64 Visual Studio Developer PowerShell 中运行；
这些工具只用于构建，不需要安装到内网使用电脑。

```powershell
$env:RUSTFLAGS = "-C target-feature=+crt-static"
$fxc = Get-ChildItem "${env:ProgramFiles(x86)}\Windows Kits\10\bin" -Filter fxc.exe -Recurse |
    Where-Object { $_.FullName -match '\\x64\\fxc\.exe$' } |
    Sort-Object FullName -Descending | Select-Object -First 1
if (-not $fxc) { throw "Windows SDK fxc.exe was not found" }
$env:GPUI_FXC_PATH = $fxc.FullName
cargo +1.95.0 build --locked --release --target x86_64-pc-windows-msvc -p rustgrid-app --features offline
```

产物为 `target/x86_64-pc-windows-msvc/release/RustGrid.exe`。发布时随附 `LICENSE` 和部署说明，
压缩包放入不纳入版本控制的 `dist/` 目录。构建依赖已缓存时，可额外添加 Cargo 的 `--offline` 参数；
该参数只禁止构建时下载依赖，程序的离线行为由 `--features offline` 决定。

## 项目结构（Cargo workspace）

- `crates/rustgrid-core` —— 与引擎无关的领域层：驱动 / 连接 trait 与模型。
- `crates/rustgrid-mysql` —— MySQL 与 MariaDB 驱动。
- `crates/rustgrid-sqlite` —— SQLite 驱动。
- `crates/rustgrid-sqlserver` —— SQL Server 驱动。
- `crates/rustgrid-postgresql` —— PostgreSQL 驱动。
- `crates/rustgrid-oracle` —— Oracle 驱动（Oracle 官方纯 Rust `oracledb` thin 驱动，无需 Instant Client）。
- `crates/rustgrid-tunnel` —— 共享的 SSH / SOCKS5 / HTTP 隧道，供网络驱动复用。
- `crates/rustgrid-odbc` —— 通用 ODBC 驱动（DB2、达梦 等）。
- `crates/rustgrid-backup` —— `.rgbak` 备份容器。
- `crates/rustgrid-export` —— 表数据导出器（`.xlsx` / `.csv` / `.sql` / `.txt`）。
- `crates/rustgrid-import` —— Excel / CSV / TXT 源文件读取。
- `crates/rustgrid-config` —— 带版本号的设置 / 连接配置，以及加密的密钥存储。
- `crates/rustgrid-app` —— GPUI 桌面应用（二进制 `RustGrid`）。

## 开源协议

采用 **Apache License 2.0** 协议，允许商用。完整协议文本见 [`LICENSE`](LICENSE)。

## 状态

项目处于早期阶段，迭代很快 —— 功能、界面与磁盘存储格式都可能发生变化，恕不另行通知。
