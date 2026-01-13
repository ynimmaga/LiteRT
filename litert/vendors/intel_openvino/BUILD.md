# Building OpenVINO Backend with LiteRT on Linux

This document describes how to build **LiteRT with OpenVINO support** on Linux. The resulting build produces OpenVINO-based compiler and dispatch plugins that can be used by applications integrating LiteRT.

---

## Prerequisites

- Linux (x86_64)
- CMake ≥ 3.20
- Clang / Clang++
- OpenVINO Runtime (development package)
- Git

---

## Step 1: Install OpenVINO and Set Up Environment Variables

Install OpenVINO using the official distribution (archive or package manager).

After installation, source the OpenVINO environment:

```bash
source <openvino_install_dir>/setupvars.sh
```

### Verify that the OpenVINO environment is set correctly:

```bash
echo $INTEL_OPENVINO_DIR
```

## Step 2: Clone the LiteRT Repository

```bash
git clone https://github.com/google/LiteRT.git
cd LiteRT
```

## Step 3: Initialize Submodules
LiteRT depends on multiple submodules (including TensorFlow Lite).

```bash
git submodule update --init --recursive
```

## Step 4: Configure the Build
Navigate to the LiteRT source directory:

```bash
cd LiteRT/litert
```

Configure the build with OpenVINO enabled:

```bash
cmake -S . -B build-release \
  -DTFLITE_EXTRA_CMAKE_ARGS="-DXNNPACK_ENABLE_AVX2=ON;-DXNNPACK_ENABLE_AVX512=OFF;-DXNNPACK_ENABLE_AVXVNNI=OFF;-DXNNPACK_ENABLE_VNNI=OFF;-DTFLITE_BUILD_TESTS=OFF;-DBUILD_TESTING=OFF;-DGTEST_HAS_PTHREAD=0" \
  -DCMAKE_BUILD_TYPE=Release \
  -DLITERT_BUILD_TOOLS=OFF \
  -DLITERT_AUTO_BUILD_TFLITE=ON \
  -DLITERT_ENABLE_GPU=OFF \
  -DLITERT_ENABLE_NPU=OFF \
  -DLITERT_DISABLE_KLEIDIAI=ON \
  -DLITERT_HOST_C_COMPILER=/usr/bin/clang \
  -DLITERT_HOST_CXX_COMPILER=/usr/bin/clang++ \
  -DLITERT_ENABLE_OPENVINO=ON \
  -DOpenVINO_DIR=$INTEL_OPENVINO_DIR/runtime/cmake
```

### Configuration Notes
- AVX2 is enabled; AVX-512 and VNNI variants are explicitly disabled
- TensorFlow Lite is built automatically as part of LiteRT
- GPU, NPU, and KleidiAI backends are disabled
- OpenVINO backend is explicitly enabled
- Clang is used as the host compiler

## Step 5: Build LiteRT
```bash
cmake --build build-release --parallel
```

## Step 6: Verify OpenVINO Artifacts
After a successful build, verify that the OpenVINO plugins were generated.

```bash
cd build-release
```

### Check for the following shared libraries:

Compiler Plugin: 
```
build-release/vendors/intel_openvino/compiler/libLiteRTCompilerPlugin_IntelOpenVINO.so
```

Dispatch Plugin:
```
build-release/vendors/intel_openvino/dispatch/libLiteRtDispatch_IntelOpenvino.so
```

If both shared libraries are present, the OpenVINO backend for LiteRT has been built successfully.

## Troubleshooting
- Ensure INTEL_OPENVINO_DIR is set correctly
- Confirm Clang paths match your system installation
- Use cmake --build build-release --verbose for detailed logs
- Clean and reconfigure if switching compilers or OpenVINO versions
