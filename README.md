# qtifw-mac-arm

在 GitHub Actions（`macos-14` / Apple Silicon）上编译 [Qt Installer Framework](https://github.com/qtproject/installer-framework) **4.11.0**，产出 arm64 工具二进制。

## 用法

1. 打开仓库 **Actions** → **Build QtIFW 4.11.0 (macOS arm64)**
2. 点击 **Run workflow**
3. 结束后下载 artifact：`qtifw-4.11.0-macos-arm64`

产物包含：`binarycreator`、`repogen`、`archivegen`、`devtool`（已 `strip`）。

## 构建说明

| 项 | 取值 |
| --- | --- |
| 源码 | `qtproject/installer-framework` @ `4.11.0` |
| Runner | `macos-14`（arm64） |
| Qt | 6.8.0 desktop（aqt / `clang_64`） |
| 模块 | `qt5compat`（`qttools` 不是 aqt module） |
| 构建系统 | **qmake** + `make`（4.11 无 CMake） |
| 依赖 | Homebrew `xz`（liblzma，供 libarchive） |

官方文档要求：若要打出「单文件、几乎不依赖外部 Qt」的安装包，需用 **static Qt** 再编 IFW。当前 workflow 使用官方动态库 Qt，工具本身可运行，但会依赖同版本 Qt frameworks。

## 相对草稿的修正

原稿不能直接跑通，主要差异：

1. **不要用 CMake / `-DIFW_STATIC=ON`**：4.11.0 根目录只有 `installerfw.pro`，Coin CI 也是 `qmake` + `make`。
2. **aqt arch 是 `clang_64`**，不是 `macos_arm64`。
3. **模块名是 `qt5compat` / `qttools`**，不是 `qtshade`。
4. **需要 `brew install xz`**，否则 libarchive 相关链接可能失败。
5. **Release 产物对二进制做 `strip`**，避免交付物泄漏大量符号。

## 许可证

本仓库工作流脚本按仓库 `LICENSE`（GPL-3.0）分发。QtIFW 源码与产物遵循其上游许可证（见 QtIFW `LICENSES/`）。
