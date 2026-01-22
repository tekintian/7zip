# 7-Zip on GitHub

7-Zip website: [7-zip.org](https://7-zip.org)

**版本**: 25.01 (2025-08-03)

本仓库: [github.com/tekintian/7zip](https://github.com/tekintian/7zip) | 作者: [dev.tekin.cn](https://dev.tekin.cn)

本仓库包含 7-Zip 25.01 的源代码，支持跨平台构建（Linux/macOS/Windows）。

## 构建产物说明

### Linux (x86_64)

| 产物 | 说明 | 支持的格式 | 构建路径 |
|------|------|-----------|---------|
| `7za` | 控制台版本 (轻量级) | 7z, xz, cab, zip, gzip, bzip2, tar | `CPP/7zip/Bundles/Alone/_o/7za` |
| `7zz` | 控制台版本 (完整版) | **所有格式** (包括 ZSTD, LZFSE, RAR 等) | `CPP/7zip/Bundles/Alone2/_o/7zz` |
| `7z.so` | 动态库 (7z 格式) | 7z (解压/压缩) | `CPP/7zip/Bundles/Format7zF/_o/7z.so` |
| `7zdec` | C 语言解码器 | 7z (仅解压, LZMA/Copy 方法) | `C/Util/7z/_o/7zdec` |

**注意**：
- `7za` 禁用了 ZSTD 和 LZFSE 格式支持（通过 `ZIP_FLAGS=-DZ7_ZIP_LZFSE_DISABLE` 编译标志）
- `7zz` 支持所有格式，包括 ZSTD 和 LZFSE
- 输出目录：`_o/`

### macOS (x86_64)

| 产物 | 说明 | 支持的格式 | 构建路径 |
|------|------|-----------|---------|
| `7za` | 控制台版本 (轻量级) | 7z, xz, cab, zip, gzip, bzip2, tar | `CPP/7zip/Bundles/Alone/_o/7za` |
| `7zz` | 控制台版本 (完整版) | **所有格式** (包括 ZSTD, LZFSE, RAR 等) | `CPP/7zip/Bundles/Alone2/_o/7zz` |
| `7z.so` | 动态库 (7z 格式) | 7z (解压/压缩) | `CPP/7zip/Bundles/Format7zF/_o/7z.so` |
| `7zdec` | C 语言解码器 | 7z (仅解压, LZMA/Copy 方法) | `C/Util/7z/_o/7zdec` |

**注意**：
- `7za` 禁用了 ZSTD 和 LZFSE 格式支持（通过 `ZIP_FLAGS=-DZ7_ZIP_LZFSE_DISABLE` 编译标志）
- `7zz` 支持所有格式，包括 ZSTD 和 LZFSE
- macOS 构建目标：`x86_64-apple-macos10.15`，最低支持 macOS 10.15
- 输出目录：`_o/`

### Windows (x64)

| 产物 | 说明 | 支持的格式 | 构建路径 |
|------|------|-----------|---------|
| `7za.exe` | 控制台版本 (轻量级) | 7z, xz, cab, zip, gzip, bzip2, tar | `CPP/7zip/Bundles/Alone/x64/7za.exe` |
| `7zr.exe` | 控制台版本 (仅 7z) | 仅 7z 格式 | `CPP/7zip/Bundles/Alone7z/x64/7zr.exe` |

**注意**：
- `7za.exe` 默认禁用了 ZSTD 和 LZFSE 格式支持
- 输出目录：`x64/` (由 `PLATFORM=x64` 参数控制)

## 技术细节

### 支持的压缩方法

**核心方法**：
- `LZMA2` (0x21) - 7z 默认方法，高压缩比，快速解压
- `LZMA` (0x03 01) - 传统 LZMA 方法
- `Copy` (0x00) - 无压缩
- `Delta` (0x03) - 差分编码

**过滤器**：
- `BCJ` (0x04) - x86 可执行代码过滤器
- `BCJ2` (0x03 01 1B) - x86 4 流过滤器
- `PPC` (0x05) - PowerPC (大端)
- `IA64` (0x06) - Itanium
- `ARM` (0x07) - ARM (小端)
- `ARM64` (0x0A) - ARM64
- `RISCV` (0x0B) - RISC-V

**其他方法**：
- `PPMd` (0x04 01) - PPMd 压缩
- `BZip2` (0x04 02) - BZip2 压缩
- `Deflate` (0x04 01 08) - ZIP Deflate
- `Deflate64` (0x04 01 09) - ZIP Deflate64

**扩展方法** (仅在 7zz 完整版支持)：
- `ZSTD` (0x04 01 5D) - Zstandard 压缩
- `WavPack` (0x04 01 61) - 音频压缩
- `PPMd-zip` (0x04 01 62) - ZIP PPMd

### 加密方法

| 方法 | 标识符 | 说明 |
|------|--------|------|
| AES-128 | 0x06 F0 01 10 | AES-128 ECB/CBC/CFB/OFB/CTR |
| AES-192 | 0x06 F0 01 40 | AES-192 ECB/CBC/CFB/OFB/CTR |
| AES-256 | 0x06 F0 01 80 | AES-256 ECB/CBC/CFB/OFB/CTR |
| 7zAES | 0x06 F1 07 01 | AES-256 + SHA-256 (7z 默认) |
| ZipCrypto | 0x06 F1 01 01 | ZIP 传统加密 |
| Rar29AES | 0x06 F1 03 03 | RAR29 AES-128 + SHA-1 |

### 编译选项

**变量说明**：
- `PLATFORM` - 目标平台：x64, x86, arm64, arm, ia64
- `USE_ASM` - 启用汇编优化 (需要 Asmc 或 UASM)
- `DISABLE_RAR=1` - 完全禁用 RAR 支持
- `DISABLE_RAR_COMPRESS=1` - 仅禁用 RAR 解压 (仍可查看文件列表)

**macOS 特定选项**：
```bash
export CFLAGS="-target x86_64-apple-macos10.15 -mmacosx-version-min=10.15"
export LDFLAGS="-target x86_64-apple-macos10.15 -mmacosx-version-min=10.15"
```

**Windows 特定选项**：
```bash
nmake -f makefile PLATFORM=x64
```

### 版本差异

| 特性 | 7za | 7zz | 7zr | 7zdec |
|------|-----|-----|-----|-------|
| 文件大小 | 小 | 大 | 最小 | 最小 |
| 格式支持 | 有限 (不含 ZSTD/LZFSE) | 所有格式 | 仅 7z | 仅 7z (LZMA) |
| 编码语言 | C++ | C++ | C++ | C |
| 动态链接 | 否 | 否 | 否 | 否 |
| RAR 支持 | 是 | 是 | 否 | 否 |

### 许可证信息

- **7-Zip 核心代码**: GNU LGPL
- **LZMA SDK**: 公有领域
- **unRAR 代码**: 专有许可 (限制性较弱，可用于解压 RAR)
- **部分代码**: BSD 3-clause 许可证

### 内存要求

**7z 解码**：
- 打开档案：临时池（头部数据） + 主池（文件列表）
- 解压：临时池（LZMA 结构） + 主池（压缩块 + BCJ2 缓冲区约 15%）
- 不为压缩块分配内存（需用户分配缓冲区）

### 构建系统

| 平台 | Makefile | 构建工具 | 输出目录 |
|------|----------|---------|---------|
| Linux | `makefile.gcc` | GNU make | `_o/` |
| macOS | `makefile.gcc` | GNU make | `_o/` |
| Windows | `makefile` | NMake | `x64/` (由 PLATFORM 决定) |

### 已知限制

1. **7zdec (C 解码器)**：
   - 仅支持 LZMA 和 Copy 方法 + BCJ/BCJ2 过滤器
   - 只读取文件名、大小、修改时间和 CRC
   - 文件名从 UTF-16 转换为 UTF-8

2. **7za (轻量版)**：
   - 不支持 ZSTD 压缩
   - 不支持 LZFSE (macOS 专用格式)

3. **汇编优化**：
   - 需要安装 Asmc 或 UASM (Linux/macOS)
   - JWasm 不支持 AES 指令，会回退到 C 实现

### 相关链接

- **batch7z**: [github.com/tekintian/batch7z](https://github.com/tekintian/batch7z) - 7z 批量打包备份工具，支持自动化备份和压缩
- 7-Zip 官方: [www.7-zip.org](https://www.7-zip.org)
- LZMA SDK: [www.7-zip.org/sdk.html](https://www.7-zip.org/sdk.html)
- Asmc: [github.com/nidud/asmc](https://github.com/nidud/asmc)
- UASM: [github.com/Terraspace/UASM](https://github.com/Terraspace/UASM)
- p7zip (旧版): [sourceforge.net/projects/p7zip](http://sourceforge.net/projects/p7zip/)
