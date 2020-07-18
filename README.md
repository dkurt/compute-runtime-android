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

2. Copy the binaries

  Create container
  ```bash
  docker run intel-opencl
  ```

  Check container ID
  ```bash
  $ docker container ls --all
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
310ebb294f49        intel-opencl        "/bin/bash"              15 seconds ago      Exited (0) 13 seconds ago                        goofy_snyder0
  ```

  Copy the binaries
  ```bash
  docker cp 310ebb294f49:/ocl/intel-opencl.tar.gz ./
  ```

3. On device

  * Create a folder `/system/vendor/Khronos/OpenCL/vendors/` and copy `intel.icd` to it.
  * Make sure that the rest of `*.so` libraries are in `LD_LIBRARY_PATH`.
