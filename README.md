# OpenCV 自动构建仓库

[![Build and Release OpenCV](https://github.com/crazylin/OpenCV_Build/actions/workflows/build_and_release.yml/badge.svg)](https://github.com/crazylin/OpenCV_Build/actions/workflows/build_and_release.yml)

这是一个使用 GitHub Actions 自动编译多个 OpenCV 版本（包括动态和静态链接库）并将其发布到 GitHub Releases 的项目。

## ✨ 功能特性

- **多版本支持**: 自动编译多个指定的 OpenCV 版本。
- **动态与静态链接**: 为每个版本同时生成动态链接库 (`.dll`) 和静态链接库 (`.lib`)。
- **跨编译环境**: 针对不同 OpenCV 版本使用合适的 Visual Studio 版本（VS2022 / VS2019）进行编译，以确保兼容性。
- **自动发布**: 编译完成后，所有产物会自动打包并发布到同一个 GitHub Release 中，方便下载。
- **完全自动化**: 工作流可以手动触发，也可以在推送到 `main` 分支时自动运行。

## 🚀 如何使用

### 1. 触发工作流

您可以通过以下两种方式启动构建流程：

- **手动触发**:
    1.  进入本仓库的 **Actions** 页面。
    2.  在左侧选择 **Build and Release OpenCV** 工作流。
    3.  点击 **Run workflow** 按钮，然后再次确认运行。
- **自动触发**:
    1.  `fork` 本仓库。
    2.  将任何更改推送到您自己仓库的 `main` 或 `master` 分支。

### 2. 下载产物

工作流执行成功后，所有编译好的文件都会被上传到 GitHub Releases。

1.  进入本仓库的 **Releases** 页面。
2.  找到最新的 Release（例如 `OpenCV Builds (Run <number>)`）。
3.  在 **Assets** 部分下载您需要的 `.zip` 压缩包。

文件名格式如下：
- `opencv-<版本号>-win64-<VC版本>-static.zip`: 静态链接库
- `opencv-<版本号>-win64-<VC版本>.zip`: 动态链接库

## 🛠️ 自定义

您可以轻松地 fork 本项目并根据自己的需求进行定制。

1.  **Fork** 本仓库。
2.  编辑 `.github/workflows/build_and_release.yml` 文件。

### 修改编译版本

在 `jobs.build.strategy.matrix.include` 部分，您可以增、删、改需要编译的 OpenCV 版本、操作系统或链接方式。

```yaml
# .github/workflows/build_and_release.yml

# ...
    strategy:
      fail-fast: false
      matrix:
        include:
          - opencv_version: '4.8.0'
            os: windows-latest # 使用 VS2022
            linkage: shared
            build_shared_libs: ON
          - opencv_version: '4.8.0'
            os: windows-latest
            linkage: static
            build_shared_libs: OFF
          # 在这里添加更多版本...
# ...
```

### 修改 CMake 参数

在 `Configure CMake` 步骤中，您可以添加或修改 CMake 参数以满足您的特定需求（例如，启用/禁用特定模块）。

```yaml
# .github/workflows/build_and_release.yml

# ...
    - name: Configure CMake
      # ...
      run: |
        cmake ../opencv \
          -D CMAKE_BUILD_TYPE=Release \
          -D BUILD_SHARED_LIBS=${{ matrix.build_shared_libs }} \
          -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
          # 在这里添加您的 CMake 参数...
# ...
```

## ⚙️ 工作流详解

本仓库的核心是位于 `.github/workflows/build_and_release.yml` 的 GitHub Actions 工作流。它主要分为两个阶段：

1.  **`build` 任务**:
    - 使用构建矩阵 (`matrix`) 为每个定义的 OpenCV 版本和链接类型组合启动一个并行的构建任务。
    - 检出对应版本的 `opencv` 和 `opencv_contrib` 源码。
    - 使用 CMake 配置构建选项并执行编译。
    - 将编译好的文件（头文件、库文件、二进制文件）打包成 `.zip` 文件。
    - 将每个 `.zip` 文件作为构建产物 (artifact) 上传。

2.  **`release` 任务**:
    - 此任务依赖于所有 `build` 任务的成功完成。
    - 它会下载所有由 `build` 任务上传的构建产物。
    - 创建一个以构建运行号命名的单一 GitHub Release。
    - 将所有下载的 `.zip` 文件作为资源 (assets) 上传到这个 Release 中。

## 📝 许可证

本项目的脚本代码使用 [MIT License](LICENSE) 授权。OpenCV 本身使用 Apache 2 License。 