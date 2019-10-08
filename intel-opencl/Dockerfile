FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install -y \
        git \
        cmake \
        wget \
        unzip \
        bison \
        flex \
        build-essential \
        python-minimal \
        patchelf \
        pkg-config

RUN git config --global user.email "you@example.com" && \
    git config --global user.name "Your Name"

# Clone repositories
RUN git clone -b igc-1.0.2652 https://github.com/intel/intel-graphics-compiler igc --depth 1
RUN git clone -b android_fix https://github.com/dkurt/gmmlib --depth 1
RUN mkdir runtime && git clone -b fix_android https://github.com/dkurt/compute-runtime runtime/ --depth 1

RUN git clone -b release/9.x https://github.com/llvm/llvm-project llvm-project --depth 1
RUN git clone -b ocl-open-90 https://github.com/intel/opencl-clang llvm-project/llvm/projects/opencl-clang --depth 1
RUN git clone -b llvm_release_90 https://github.com/KhronosGroup/SPIRV-LLVM-Translator llvm-project/llvm/projects/llvm-spirv --depth 1
RUN git clone https://github.com/intel/llvm-patches llvm_patches --depth 1
RUN mv llvm-project/clang llvm-project/llvm/tools/

# Download NDK
RUN wget https://dl.google.com/android/repository/android-ndk-r20-linux-x86_64.zip && \
    unzip android-ndk-r20-linux-x86_64.zip && \
    rm android-ndk-r20-linux-x86_64.zip

# Compile native gmmlib (need for ocloc)
RUN cd gmmlib && mkdir build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release .. && \
    make -j8 install

# Compile native IGC (need for ocloc)
RUN mkdir -p build_native_igc && cd build_native_igc && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
    ../igc && make -j8 install

RUN cd build_native_igc && make -j4 && make install

#
# Build compute runtime (libigdfcl.so and intel.icd)
#

RUN mv runtime compute-runtime && mkdir runtime && mv compute-runtime runtime/compute-runtime

# Compile ocloc
RUN cd runtime && mkdir build_ocloc && cd build_ocloc && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DSKIP_ALL_ULT=ON \
      -DSKIP_UNIT_TESTS=ON ../compute-runtime && \
      make -j$(nproc) ocloc

RUN cd runtime && mkdir build && cd build && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_TOOLCHAIN_FILE=/android-ndk-r20/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=x86_64 \
      -DSKIP_ALL_ULT=ON \
      -DSKIP_UNIT_TESTS=ON ../compute-runtime && \
    make -j$(nproc); exit 0
# RUN cd /runtime/build && cp /runtime/build_ocloc/bin/ocloc ./bin && \
#     export LD_LIBRARY_PATH=/usr/local/lib/: && \
#     make -j$(nproc)
#
# #
# # Cross compile IGC
# #
#
# # Apply patches
# RUN sed -i -E 's|linux_resource_linker|/build_native_igc/IGC/llvm/build/src/bin/linux_resource_linker|' llvm-project/llvm/projects/opencl-clang/cl_headers/CMakeLists.txt
# RUN sed -i -E 's|target_link_libraries\(IGA_EXE PUBLIC IGA_SLIB "-lrt"\)|target_link_libraries\(IGA_EXE PUBLIC IGA_SLIB\)|' igc/visa/iga/IGAExe/CMakeLists.txt
# RUN sed -i -E 's|-lrt|\n|' igc/IGC/CMakeLists.txt
# RUN sed -i -E 's|target_link_libraries\(GenX_IR_Exe rt dl\)|target_link_libraries\(GenX_IR_Exe dl\)|' igc/visa/CMakeLists.txt
# RUN sed -i -E 's|llvm-tblgen|/build_native_igc/IGC/llvm/build/src/bin/llvm-tblgen|' llvm-project/llvm/projects/opencl-clang/CMakeLists.txt
# RUN sed -i -E 's|UNIX|UNIX OR ANDROID|' igc/IGC/BiFModule/CMakeLists.txt
# RUN sed -i -E 's|COMMAND clang-tool|COMMAND /build_native_igc/IGC/llvm/build/src/bin/clang|' igc/IGC/BiFModule/CMakeLists.txt
# RUN sed -i -E 's|DEPENDS clang-tool|DEPENDS /build_native_igc/IGC/llvm/build/src/bin/clang|' igc/IGC/BiFModule/CMakeLists.txt
# RUN sed -i -E 's|COMMAND llvm-link|COMMAND /build_native_igc/IGC/llvm/build/src/bin/llvm-link|' igc/IGC/BiFModule/CMakeLists.txt
# RUN sed -i -E 's|\$<TARGET_FILE:\$\{IGC_BUILD__PROJ__ElfPackager\}> -includeSizet -funcList \$\{CMAKE_CURRENT_SOURCE_DIR\}/function_bin.txt \$\{IGC_BUILD__BIF_DIR\}/OCLBiFImpl.bc \$\{IGC_BUILD__BIF_DIR\}/igdclbif.bin|echo 1|' igc/IGC/ElfPackager/CMakeLists.txt
#
# RUN cd build && \
#     mkdir -p NATIVE/bin && \
#     cp /build_native_igc/IGC/llvm/build/src/bin/llvm-tblgen NATIVE/bin && \
#     cp /build_native_igc/IGC/llvm/build/src/bin/clang-tblgen NATIVE/bin && \
#     cmake \
#         -DCMAKE_BUILD_TYPE=Release \
#         -DCMAKE_TOOLCHAIN_FILE=../android-ndk-r20/build/cmake/android.toolchain.cmake \
#         -DANDROID_ABI=x86_64 \
#         -DCMAKE_CXX_FLAGS="-Wno-macro-redefined -Wno-user-defined-warnings -Wno-deprecated" \
#     ../igc && make -j$(nproc) fcl_dll igc_dll
#
#
# # Compile gmmlinb for Android
# RUN cd gmmlib && mkdir build_android && cd build_android && \
#     cmake \
#       -DCMAKE_BUILD_TYPE=Release \
#       -DCMAKE_TOOLCHAIN_FILE=../android-ndk-r20/build/cmake/android.toolchain.cmake \
#       -DANDROID_ABI=x86_64 \
#       -DRUN_TEST_SUITE=OFF .. && \
#     make -j$(nproc)
#
# # Collect the binaries
# RUN mkdir ocl && cd ocl && \
#     cp /build/IGC/Release/libigdfcl.so libigdfcl.so.1 && patchelf --set-soname libigdfcl.so.1 libigdfcl.so.1 && \
#     cp /build/IGC/Release/libigc.so libigc.so.1 && patchelf --set-soname libigc.so.1 libigc.so.1 && \
#     cp /gmmlib/build_android/Source/GmmLib/libigdgmm.so libigdgmm.so.11 && patchelf --set-soname libigdgmm.so.11 libigdgmm.so.11 && \
#     cp /build/IGC/llvm/build/src/lib/libopencl-clang.so . && \
#     cp /runtime/build/bin/libigdrcl.so . && \
#     echo "libigdrcl.so" > intel.icd
