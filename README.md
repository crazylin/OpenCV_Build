# OpenCV Auto-Build Repository

[![Build and Release OpenCV](https://github.com/crazylin/OpenCV_Build/actions/workflows/build_and_release.yml/badge.svg)](https://github.com/crazylin/OpenCV_Build/actions/workflows/build_and_release.yml)

This project uses GitHub Actions to automatically compile multiple versions of OpenCV (for both shared and static linkage) and publish them to a single GitHub Release.

## ✨ Features

- **Multi-Version Support**: Automatically compiles a matrix of specified OpenCV versions.
- **Shared & Static Linking**: Generates both shared libraries (`.dll`) and static libraries (`.lib`) for each version.
- **Cross-Compiler Compatibility**: Uses the appropriate Visual Studio version (VS2022 / VS2019) for each OpenCV version to ensure compatibility.
- **Automated Releases**: After compilation, all artifacts are bundled into a single GitHub Release for easy downloading.
- **Fully Automated**: The workflow can be triggered manually or run automatically on push to the `main` branch.

## 🚀 How to Use

### 1. Trigger the Workflow

You can start the build process in two ways:

- **Manual Trigger**:
    1.  Navigate to the **Actions** tab of this repository.
    2.  Select the **Build and Release OpenCV** workflow from the sidebar.
    3.  Click the **Run workflow** button and confirm the run.
- **Automatic Trigger**:
    1.  Fork this repository.
    2.  Push any changes to the `main` or `master` branch of your own repository.

### 2. Download the Artifacts

After the workflow completes successfully, all compiled files will be uploaded to GitHub Releases.

1.  Navigate to the **Releases** page of this repository.
2.  Find the latest release (e.g., `OpenCV Builds (Run <number>)`).
3.  Download the desired `.zip` archives from the **Assets** section.

The file naming convention is as follows:
- `opencv-<version>-win64-<vc_version>-static.zip`: Statically linked libraries
- `opencv-<version>-win64-<vc_version>.zip`: Shared (dynamically linked) libraries

## 🛠️ Customization

You can easily fork this project and customize it to your needs.

1.  **Fork** this repository.
2.  Edit the `.github/workflows/build_and_release.yml` file.

### Modify Build Versions

In the `jobs.build.strategy.matrix` section, you can add, remove, or change the OpenCV versions, operating systems, or linkage types to be built.

```yaml
# .github/workflows/build_and_release.yml

# ...
    strategy:
      fail-fast: false
      matrix:
        version_info:
          - { opencv_version: '4.8.0', os: windows-latest, vc_version: 'vc17' } # VS2022
          - { opencv_version: '4.5.5', os: windows-latest, vc_version: 'vc17' }
          - { opencv_version: '3.4.20', os: windows-2019, vc_version: 'vc16' } # VS2019
          - { opencv_version: '2.4.13', os: windows-2019, vc_version: 'vc16' }
        linkage: [shared, static]
# ...
```

> **Note**: The workflow automatically handles legacy versions like `2.4.13` (which do not have a separate `opencv_contrib` repository) and adjusts the build process accordingly.

### Modify CMake Parameters

In the `Configure CMake` step, you can add or modify CMake parameters to fit your specific requirements (e.g., enabling or disabling certain modules).

```yaml
# .github/workflows/build_and_release.yml

# ...
    - name: Configure CMake
      # ...
      run: |
        # ...
        CMAKE_FLAGS="-D CMAKE_BUILD_TYPE=Release \
          -D BUILD_SHARED_LIBS=${BUILD_SHARED_LIBS_FLAG} \
          # Add your custom CMake flags here...
        "
        # ...
        cmake ../opencv $CMAKE_FLAGS
# ...
```

## ⚙️ Workflow Explained

The core of this repository is the GitHub Actions workflow located at `.github/workflows/build_and_release.yml`. It is divided into two main jobs:

1.  **`build` Job**:
    - Uses a build matrix to start a parallel job for each combination of OpenCV version and linkage type.
    - Checks out the corresponding source code for `opencv` and `opencv_contrib`.
    - Configures build options using CMake and compiles the code.
    - Packages the resulting files (headers, libraries, binaries) into a `.zip` archive.
    - Uploads each `.zip` file as a build artifact.

2.  **`release` Job**:
    - This job depends on the successful completion of all `build` jobs.
    - It downloads all build artifacts uploaded by the `build` jobs.
    - It creates a single GitHub Release, tagged with the build run number.
    - It uploads all the downloaded `.zip` archives as assets to this release.

## 📝 License

The scripts in this project are licensed under the [MIT License](LICENSE). OpenCV itself is licensed under the Apache 2 License. 