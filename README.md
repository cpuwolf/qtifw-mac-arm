# qtifw-mac-arm

在 GitHub Actions（`macos-14` / Apple Silicon）上编译 [Qt Installer Framework](https://github.com/qtproject/installer-framework) **4.11.0**，产出 arm64 工具二进制。

## 用法

打 `v*` tag 并 push 后自动构建，并发布到 GitHub Release：

```bash
git tag v4.11.0-1
git push origin v4.11.0-1
```

结束后在 **Releases** 下载 `qtifw-4.11.0-macos-arm64.zip`（同时保留 Actions artifact：`qtifw-4.11.0-macos-arm64`）。

产物包含：`binarycreator`、`repogen`、`archivegen`、`devtool`、`installerbase`（已 `strip`）。

## 构建说明

| 项 | 取值 |
| --- | --- |
| 源码 | `qtproject/installer-framework` @ `4.11.0` |
| Runner | `macos-14`（arm64） |
| Qt | **static** 6.8.3（[`AllanChain/install-qt-static@v6.8.3-1`](https://github.com/AllanChain/install-qt-static)） |
| 补编模块 | `qt5compat`（预编译包不含，源码补装进同一 prefix） |
| 构建系统 | **qmake** + `make`（4.11 无 CMake） |
| 依赖 | Homebrew `xz` / `p7zip` / `ninja`；`IFW_LZMA_LIBRARY` 指向静态 `liblzma.a` |

按官方 INSTALL：用 static Qt 编 IFW，工具与其打出的安装包可避免依赖外部 Qt frameworks。打包步骤会用 `otool -L` 检查产物不链 `Qt*.framework` / `libQt6*.dylib`。

## 已知限制

- 预编译 Qt 使用 `-no-feature-accessibility` / `-optimize-size`，与 IFW INSTALL 推荐配置不完全一致；若编译缺 accessibility API，需换 tag 或自建 static Qt。
- 解压后需写 `qt_static/bin/qt.conf`（`Prefix = ..`），否则 `qmake` 找不到 `macx-clang`。
- 产物仍可能依赖系统或 Homebrew 的非 Qt 库（如 iconv）；目标是去掉 **Qt** 动态依赖。
- 预编译包为 universal（x86_64+arm64）；本 workflow 以 `QMAKE_APPLE_DEVICE_ARCHS=arm64` 只产出 arm64 IFW 工具。

## 相对草稿的修正

原稿不能直接跑通，主要差异：

1. **不要用 CMake / `-DIFW_STATIC=ON`**：4.11.0 根目录只有 `installerfw.pro`，Coin CI 也是 `qmake` + `make`。
2. **用预编译 static Qt**，不用 aqt 动态桌面包。
3. **补编 `qt5compat`** 进 static prefix（上游包未包含）。
4. **需要 `brew install xz`**，并以 `IFW_LZMA_LIBRARY` 链静态 `liblzma.a`。
5. **Release 产物对二进制做 `strip`**，并用 `otool` 验收无 Qt dylib。

## 许可证

本仓库工作流脚本按仓库 `LICENSE`（GPL-3.0）分发。QtIFW 源码与产物遵循其上游许可证（见 QtIFW `LICENSES/`）。
