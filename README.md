# 🚀 OpenCV Prebuilt Binaries

[![GitHub Release](https://img.shields.io/github/v/release/mh-bagheri/opencv-prebuilt?style=flat-square&color=success)](https://github.com/mh-bagheri/opencv-prebuilt/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](https://opensource.org/licenses/MIT)

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
    mkdir -p library && cd library
    if [ -d "opencv" ]; then rm -rf opencv; fi
    wget https://github.com/mh-bagheri/opencv-prebuilt/releases/download/v4.10.0/opencv-4.10.0-debian-x64.tar.gz
    tar -zxvf opencv-4.10.0-debian-x64.tar.gz
    if [ -d "opencv-4.10.0-debian-x64/opencv" ]; then
        mv opencv-4.10.0-debian-x64/opencv ./opencv
        rm -rf opencv-4.10.0-debian-x64
    elif [ -d "opencv-4.10.0-debian-x64" ]; then
        mv opencv-4.10.0-debian-x64 ./opencv
    fi
    rm -f opencv-4.10.0-debian-x64.tar.gz
```

*💡 **Note**: For Red Hat or macOS environments, simply swap the URL and folder names with the `redhat-x64.tar.gz` or `macos-arm64.tar.gz` asset respectively.*

### 🪟 Windows (PowerShell)

```yaml
- name: Download Prebuilt OpenCV
  shell: pwsh
  run: |
    New-Item -ItemType Directory -Force -Path "library"
    cd library
    $ProgressPreference = 'SilentlyContinue'
    if (Test-Path "opencv") { Remove-Item -Path "opencv" -Recurse -Force }
    Invoke-WebRequest -Uri "https://github.com/mh-bagheri/opencv-prebuilt/releases/download/v4.10.0/opencv-4.10.0-windows-x64.zip" -OutFile "opencv.zip"
    Expand-Archive -Path "opencv.zip" -DestinationPath "." -Force
    if (Test-Path "opencv-4.10.0-windows-x64\opencv") {
        Move-Item -Path "opencv-4.10.0-windows-x64\opencv" -Destination "." -Force
        Remove-Item -Path "opencv-4.10.0-windows-x64" -Recurse -Force
    } elseif (Test-Path "opencv-4.10.0-windows-x64") {
        Rename-Item -Path "opencv-4.10.0-windows-x64" -NewName "opencv"
    }
    Remove-Item -Path "opencv.zip" -Force
```

---

## 🛠️ Linking with CMake

Once downloaded and extracted in your CI pipeline, you can easily point CMake to the prebuilt binaries. Just pass the path to the extracted OpenCV directory as `OpenCV_DIR` when configuring your project, or set it directly in your `CMakeLists.txt`:

```cmake
# 1. Point CMake to the local extracted OpenCV directory
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv")
    list(APPEND CMAKE_PREFIX_PATH "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv")
endif()

# 2. Find and Link the Package
find_package(OpenCV REQUIRED)
add_executable(MyAwesomeApp main.cpp)
target_include_directories(MyAwesomeApp PRIVATE ${OpenCV_INCLUDE_DIRS})
target_link_libraries(MyAwesomeApp PRIVATE ${OpenCV_LIBS})

# 3. Post-Build: Automatically Copy Dynamic Libraries to Output Directory
if(WIN32)
    # Windows dynamic libraries are located in the /bin directory
    file(GLOB_RECURSE OPENCV_RUNTIME_LIBS "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv/bin/*opencv_*.dll")
    if(OPENCV_RUNTIME_LIBS)
        foreach(OPENCV_LIB ${OPENCV_RUNTIME_LIBS})
            add_custom_command(TARGET MyAwesomeApp POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${OPENCV_LIB}"
                $<TARGET_FILE_DIR:MyAwesomeApp>/
            )
        endforeach()
    endif()
else()
    # macOS and Linux dynamic libraries use symlinks in the /lib directory.
    # We use copy_directory to natively preserve these symlinks instead of globbing.
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv/lib")
        add_custom_command(TARGET MyAwesomeApp POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory 
            "${CMAKE_CURRENT_SOURCE_DIR}/library/opencv/lib" 
            $<TARGET_FILE_DIR:MyAwesomeApp>/
        )
    endif()
endif()
```