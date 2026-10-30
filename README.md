# 🚀 OpenCV Prebuilt Binaries

[![GitHub Release](https://img.shields.io/github/v/release/mh-bagheri/opencv-prebuilt?style=flat-square&color=success)](https://github.com/mh-bagheri/opencv-prebuilt/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](https://opensource.org/licenses/MIT)

Pre-compiled Release builds of [OpenCV](https://opencv.org/) engineered for cross-platform Continuous Integration (CI) pipelines and deterministic local C++ development.

* **Zero Compile Overhead:** Replace ten to fifteen minutes of source compilation on every CI run with a download and extraction step that finishes in under five seconds.
* **Dependency Isolation:** Extract binaries directly into a repository-local library/opencv directory to avoid polluting global system paths and guarantee reproducible builds.
* **Installer Ready:** Structured for seamless post-build staging and standalone CPack distribution across Windows (NSIS exe), macOS (Drag-and-Drop dmg), and Linux (Debian deb).

## 📦 Supported Platforms & Release Assets

Download the precompiled archive for your target operating system and processor architecture from the [Releases page](../../releases).

| Operating System | Architecture | Camera Backend | Format | Asset Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **Debian Linux** | x64 | Video4Linux2 | tar.gz | opencv-version-debian-x64 |
| **Debian Linux** | ARM64 | Video4Linux2 | tar.gz | opencv-version-debian-arm64 |
| **Red Hat Linux** | x64 | Video4Linux2 | tar.gz | opencv-version-redhat-x64 |
| **Red Hat Linux** | ARM64 | Video4Linux2 | tar.gz | opencv-version-redhat-arm64 |
| **Windows** | x64 | DirectShow / MSMF | zip | opencv-version-windows-x64 |
| **Windows** | ARM64 | DirectShow / MSMF | zip | opencv-version-windows-arm64 |
| **macOS** | ARM64 | AVFoundation | tar.gz | opencv-version-macos-arm64 |

The Debian packages target Ubuntu, Mint, and Raspberry Pi (AArch64), while Red Hat packages support RHEL, Fedora, and CentOS. Windows builds support both Intel/AMD processors and Snapdragon X (Windows on ARM) systems, and macOS builds target Apple Silicon M-Series hardware.

## 📂 Recommended Project Layout

Placing the extracted OpenCV distribution inside a local library/opencv directory at the root of your repository keeps third-party headers and shared libraries self-contained:

```text
MyProject/
├── CMakeLists.txt
├── include/            # Application C++ headers (.h)
├── src/                # Application C++ sources (.cpp)
└── library/
    └── opencv/         # Extracted OpenCV headers, CMake configs & binaries
```

## ⚡ Environment Setup & CI Integration

Use the scripts below in your local terminal or GitHub Actions workflows to download and normalize the folder structure into library/opencv.

### 🍎 macOS (Apple Silicon / ARM64)

```bash
mkdir -p library && cd library
if [ -d "opencv" ]; then rm -rf opencv; fi
wget https://github.com/MH-Bagheri/OpenCV-Prebuilt/releases/download/v4.10.0/opencv-4.10.0-macos-arm64.tar.gz
tar -zxvf opencv-4.10.0-macos-arm64.tar.gz

if [ -d "opencv-4.10.0-macos-arm64/opencv" ]; then
    mv opencv-4.10.0-macos-arm64/opencv ./opencv
    rm -rf opencv-4.10.0-macos-arm64
elif [ -d "opencv-4.10.0-macos-arm64" ]; then
    mv opencv-4.10.0-macos-arm64 ./opencv
fi
rm -f opencv-4.10.0-macos-arm64.tar.gz
cd ..
```

### 🐧 Linux (Debian & Red Hat — x64 & ARM64)

Swap the archive name with the Red Hat or ARM64 variant when targeting RHEL/Fedora or AArch64 devices:

```bash
mkdir -p library && cd library

# --- Option A: Debian/Ubuntu x64 (Intel / AMD) ---
if [ -d "opencv" ]; then rm -rf opencv; fi
wget https://github.com/MH-Bagheri/OpenCV-Prebuilt/releases/download/v4.10.0/opencv-4.10.0-debian-x64.tar.gz
tar -zxvf opencv-4.10.0-debian-x64.tar.gz
if [ -d "opencv-4.10.0-debian-x64/opencv" ]; then
    mv opencv-4.10.0-debian-x64/opencv ./opencv
    rm -rf opencv-4.10.0-debian-x64
elif [ -d "opencv-4.10.0-debian-x64" ]; then
    mv opencv-4.10.0-debian-x64 ./opencv
fi
rm -f opencv-4.10.0-debian-x64.tar.gz

# --- Option B: Debian/Ubuntu ARM64 (AArch64 / Raspberry Pi) ---
# if [ -d "opencv" ]; then rm -rf opencv; fi
# wget https://github.com/MH-Bagheri/OpenCV-Prebuilt/releases/download/v4.10.0/opencv-4.10.0-debian-arm64.tar.gz
# tar -zxvf opencv-4.10.0-debian-arm64.tar.gz
# if [ -d "opencv-4.10.0-debian-arm64/opencv" ]; then
#     mv opencv-4.10.0-debian-arm64/opencv ./opencv
#     rm -rf opencv-4.10.0-debian-arm64
# elif [ -d "opencv-4.10.0-debian-arm64" ]; then
#     mv opencv-4.10.0-debian-arm64 ./opencv
# fi
# rm -f opencv-4.10.0-debian-arm64.tar.gz

cd ..
```

### 🪟 Windows (PowerShell — x64 & ARM64)

Run the PowerShell block matching your target Windows architecture:

```powershell
New-Item -ItemType Directory -Force -Path "library"
cd library
$ProgressPreference = 'SilentlyContinue'

# For ARM64, replace 'windows-x64' with 'windows-arm64' below:
if (Test-Path "opencv") { Remove-Item -Path "opencv" -Recurse -Force }
Invoke-WebRequest -Uri "https://github.com/MH-Bagheri/OpenCV-Prebuilt/releases/download/v4.10.0/opencv-4.10.0-windows-x64.zip" -OutFile "opencv.zip"
Expand-Archive -Path "opencv.zip" -DestinationPath "." -Force

if (Test-Path "opencv-4.10.0-windows-x64\opencv") {
    Move-Item -Path "opencv-4.10.0-windows-x64\opencv" -Destination "." -Force
    Remove-Item -Path "opencv-4.10.0-windows-x64" -Recurse -Force
} elseif (Test-Path "opencv-4.10.0-windows-x64") {
    Rename-Item -Path "opencv-4.10.0-windows-x64" -NewName "opencv"
}

Remove-Item -Path "opencv.zip" -Force
$ProgressPreference = 'Continue'
cd ..
```

#### Windows ARM64 vcpkg Dependency Patch

The Windows ARM64 archive includes strict CMake dependency checks (such as Protobuf) generated by vcpkg. Run this PowerShell snippet from your project root to strip unneeded checks prior to configuring CMake:

```powershell
$cmakePath = "library/opencv/share"
if (Test-Path $cmakePath) {
    Get-ChildItem -Path $cmakePath -Filter "*.cmake" -Recurse | ForEach-Object {
        (Get-Content $_.FullName) -replace '.*find_dependency\(.*', '' | Set-Content $_.FullName
    }
    Write-Host "Stripped missing vcpkg dependencies from OpenCV config."
}
```

## 🛠️ Production CMake & CPack Integration

The CMakeLists.txt pattern below resolves the local OpenCV headers, configures platform RPATH settings, copies shared libraries into the build folder for immediate debugging, and stages binaries for CPack installers:

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyAwesomeApp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 1. Discover Local Prebuilt OpenCV
if(EXISTS "${CMAKE_SOURCE_DIR}/library/opencv")
    list(APPEND CMAKE_PREFIX_PATH "${CMAKE_SOURCE_DIR}/library/opencv")
endif()
find_package(OpenCV REQUIRED)

add_executable(${PROJECT_NAME} src/main.cpp)
target_include_directories(${PROJECT_NAME} PRIVATE${OpenCV_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE${OpenCV_LIBS})

# 2. Configure Runtime Library Search Paths (RPATH)
if(APPLE)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        INSTALL_RPATH "@executable_path"
        BUILD_WITH_INSTALL_RPATH TRUE
    )
elseif(UNIX AND NOT APPLE)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        INSTALL_RPATH "$ORIGIN:$ORIGIN/x86_64-linux-gnu:$ORIGIN/aarch64-linux-gnu"
        BUILD_WITH_INSTALL_RPATH TRUE
    )
endif()

# 3. Post-Build Local Staging & CPack Installer Bundling
if(WIN32)
    # Windows: Glob and stage dynamic libraries (.dll) from /bin
    file(GLOB_RECURSE OPENCV_WIN_DLLS "${CMAKE_SOURCE_DIR}/library/opencv/bin/*opencv_*.dll")
    if(OPENCV_WIN_DLLS)
        foreach(OPENCV_WIN_DLL ${OPENCV_WIN_DLLS})
            add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${OPENCV_WIN_DLL}"
                $<TARGET_FILE_DIR:${PROJECT_NAME}>/
            )
        endforeach()
        install(FILES ${OPENCV_WIN_DLLS} DESTINATION ".")
    endif()
elseif(APPLE)
    # macOS: Copy /lib directory to preserve versioned .dylib symlink chains
    if(EXISTS "${CMAKE_SOURCE_DIR}/library/opencv/lib")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory
            "${CMAKE_SOURCE_DIR}/library/opencv/lib"
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/
        )
        install(DIRECTORY "${CMAKE_SOURCE_DIR}/library/opencv/lib/"
            DESTINATION "${PROJECT_NAME}.app/Contents/MacOS"
            FILES_MATCHING PATTERN "*.dylib*"
        )
    endif()
else()
    # Linux: Copy /lib directory (including multiarch subdirectories) to preserve .so symlinks
    if(EXISTS "${CMAKE_SOURCE_DIR}/library/opencv/lib")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory
            "${CMAKE_SOURCE_DIR}/library/opencv/lib"
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/
        )
        install(DIRECTORY "${CMAKE_SOURCE_DIR}/library/opencv/lib/"
            DESTINATION "bin"
            FILES_MATCHING PATTERN "*.so*"
        )
    endif()
endif()
```

## 🔨 Platform Build Commands

Configure and compile your project from a dedicated build directory using the platform-appropriate flags:

```bash
mkdir build && cd build
```

* **Linux (x64 & ARM64):** Standard CMake configuration automatically resolves OpenCV via the appended prefix path.
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release -j$(nproc)
```

* **macOS (Apple Silicon):** Pass any additional framework paths alongside the Release build type.
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release -j$(sysctl -n hw.ncpu)
```

* **Windows (x64):** Point OpenCV_DIR to the library/opencv/lib directory when generating the x64 Visual Studio solution.
```powershell
cmake .. -DOpenCV_DIR="../library/opencv/lib" -A x64
cmake --build . --config Release
```

* **Windows (ARM64):** Pass the root library/opencv path via CMAKE_PREFIX_PATH when targeting ARM64.
```powershell
cmake .. -DCMAKE_PREFIX_PATH="../library/opencv" -A ARM64
cmake --build . --config Release
```

## 🔍 Platform Packaging & Runtime Notes

* **Apple Silicon Signing:** macOS ARM64 requires valid code signatures on all bundled dynamic libraries after deployment tools modify load commands. Run an ad-hoc deep signature on your bundle to prevent launch crashes:
```bash
codesign --force --deep --sign - MyAwesomeApp.app
```
* **Linux Multiarch Isolation:** Debian archives place shared objects inside architecture-specific folders (x86_64-linux-gnu or aarch64-linux-gnu). Include both paths in your binary RPATH or wrapper script LD_LIBRARY_PATH.
* **Linux Camera Access:** If Video4Linux2 camera capture returns a permission error on /dev/video0, add your user to the video group and re-authenticate:
```bash
sudo usermod -aG video $USER
```