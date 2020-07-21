# Intel(R) Graphics Compute Runtime for OpenCL(TM) for Android(TM)

[![CI](https://github.com/dkurt/compute-runtime-android/workflows/CI/badge.svg?branch=master)](https://github.com/dkurt/compute-runtime-android/actions?query=branch%3Amaster)

This repository contains building instructions for Intel's OpenCL for Android x86 OS (64 bit).

**NOTE**: This is a temporal experimental repository in a preview state.
Use provided instructions or built binaries at your own risk.

Official repository: https://github.com/intel/compute-runtime

Download runtime: https://drive.google.com/file/d/1uJr2oq33FAchpF4j8LmVHkai7IzhrLZb/view?usp=sharing

## Build instructions

1. Build in Docker:

  ```bash
  docker build -t intel-opencl intel-opencl
  ```

2. Copy the binaries:

  ```bash
  docker run -v $(pwd):/mnt intel-opencl cp /ocl/intel-opencl-android-x86_64.tar.gz /mnt
  ```

  You will find an archive in a current working directory.

3. On device

  * Create a folder `/system/vendor/Khronos/OpenCL/vendors/` and copy `intel.icd` to it.
  * Make sure that the rest of `*.so` libraries are in `LD_LIBRARY_PATH`.
