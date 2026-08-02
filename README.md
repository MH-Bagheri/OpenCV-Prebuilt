# 🚀 OpenCV Prebuilt Binaries

[![GitHub Release](https://img.shields.io/github/v/release/mh-bagheri/opencv-prebuilt?style=flat-square&color=success)](https://github.com/mh-bagheri/opencv-prebuilt/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/github/actions/workflow/status/mh-bagheri/opencv-prebuilt/build.yml?style=flat-square)](https://github.com/mh-bagheri/opencv-prebuilt/actions)

Pre-compiled Release builds of [OpenCV](https://opencv.org/) designed specifically for standard cross-platform CI (Continuous Integration) pipelines.

> **Why use this?**
> Stop wasting 10–15 minutes compiling OpenCV from source on every single CI run. Use these binaries to download and extract the pre-built libraries in **under 5 seconds**, drastically speeding up your automated testing and deployment workflows.

---

## 📦 Available Packages

Download the archive for your target architecture directly from our [Releases page](../../releases). 

| Target OS / Architecture | Asset Name | Format |
| :--- | :--- | :--- |
| **Debian x64 (Ubuntu/Mint)** | `opencv-<version>-debian-x64.tar.gz` | `.tar.gz` |
| **Debian ARM64** | `opencv-<version>-debian-arm64.tar.gz` | `.tar.gz` |
| **Red Hat x64 (RHEL/Fedora/CentOS)** | `opencv-<version>-redhat-x64.tar.gz` | `.tar.gz` |
| **Red Hat ARM64** | `opencv-<version>-redhat-arm64.tar.gz` | `.tar.gz` |
| **Windows x64** | `opencv-<version>-windows-x64.zip` | `.zip` |
| **Windows ARM64** | `opencv-<version>-windows-arm64.zip` | `.zip` |
| **macOS ARM64** | `opencv-<version>-macos-arm64.tar.gz` | `.tar.gz` |

---

## ⚡ GitHub Actions Integration

Easily integrate these binaries into your GitHub Actions (or any other CI provider) by downloading and extracting them directly in your workflow.

### 🐧 Linux (Debian / Red Hat) & 🍎 macOS

```yaml
- name: Download Prebuilt OpenCV
  run: |
    mkdir -p library
    wget https://github.com/mh-bagheri/opencv-prebuilt/releases/download/v4.10.0/opencv-4.10.0-debian-x64.tar.gz
    tar -zxvf opencv-4.10.0-debian-x64.tar.gz -C library
    rm opencv-4.10.0-debian-x64.tar.gz
```

*💡 **Note**: For Red Hat or macOS environments, simply swap the URL with the `redhat-x64.tar.gz` or `macos-arm64.tar.gz` asset respectively.*

### 🪟 Windows (PowerShell)

```yaml
- name: Download Prebuilt OpenCV
  shell: pwsh
  run: |
    New-Item -ItemType Directory -Force -Path "library"
    Invoke-WebRequest -Uri "https://github.com/mh-bagheri/opencv-prebuilt/releases/download/v4.10.0/opencv-4.10.0-windows-x64.zip" -OutFile "opencv.zip"
    Expand-Archive -Path "opencv.zip" -DestinationPath "library"
    Remove-Item -Path "opencv.zip"
```

---

## 🛠️ Linking with CMake

Once downloaded and extracted in your CI pipeline, you can easily point CMake to the prebuilt binaries. Just pass the path to the extracted OpenCV directory as `OpenCV_DIR` when configuring your project, or set it directly in your `CMakeLists.txt`:

```cmake
# 1. Point CMake to the downloaded library directory
set(OpenCV_DIR "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv/build")

# 2. Find the package
find_package(OpenCV REQUIRED)

# 3. Link against your executable
add_executable(MyAwesomeApp main.cpp)
target_link_libraries(MyAwesomeApp PRIVATE ${OpenCV_LIBS})
```

*💡 **Note**: The exact path to the `build` directory or `OpenCVConfig.cmake` file may vary slightly depending on the OS archive structure. Adjust the `OpenCV_DIR` path accordingly.*