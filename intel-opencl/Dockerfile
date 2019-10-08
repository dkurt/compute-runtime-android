FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install -y \
        git \
        cmake \
        wget \
        unzip

RUN git clone https://github.com/intel/intel-graphics-compiler igc && \
    mkdir build

RUN wget https://dl.google.com/android/repository/android-ndk-r20-linux-x86_64.zip && \
    unzip android-ndk-r20-linux-x86_64.zip && \
    rm android-ndk-r20-linux-x86_64.zip

RUN apt-get install -y bison flex build-essential

RUN git clone -b release/9.x https://github.com/llvm/llvm-project llvm-project
RUN git clone -b ocl-open-90 https://github.com/intel/opencl-clang llvm-project/llvm/projects/opencl-clang
RUN git clone -b llvm_release_90 https://github.com/KhronosGroup/SPIRV-LLVM-Translator llvm-project/llvm/projects/llvm-spirv
RUN git clone https://github.com/intel/llvm-patches llvm_patches
RUN mv llvm-project/clang llvm-project/llvm/tools/

RUN git config --global user.email "you@example.com" && \
    git config --global user.name "Your Name"

RUN apt-get install -y bison flex build-essential
RUN cd igc && git checkout b3ed5e5

RUN apt-get install -y \
        python-minimal

RUN git clone -b android_fix https://github.com/dkurt/gmmlib

# Compile native gmmlib (need for ocloc)
RUN cd gmmlib && mkdir build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release .. && \
    make -j8 install

# Compile native IGC (need for ocloc)
RUN mkdir -p build_native && cd build_native && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
    ../igc && make -j8 install

#
# Build compute runtime (libigdfcl.so and intel.icd)
# Keep the repo isolated from IGC to be sure that it uses system libs
#
RUN mkdir rnt && cd rnt && git clone -b fix_android https://github.com/dkurt/compute-runtime

RUN apt-get install -y pkg-config

# Compile ocloc
RUN cd rnt && mkdir build_ocloc && cd build_ocloc && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DSKIP_ALL_ULT=ON \
      -DSKIP_UNIT_TESTS=ON ../compute-runtime && \
      make -j8 ocloc

RUN cd rnt && mkdir build && cd build && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_TOOLCHAIN_FILE=/android-ndk-r20/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=x86_64 \
      -DSKIP_ALL_ULT=ON \
      -DSKIP_UNIT_TESTS=ON ../compute-runtime && \
    make -j8; exit 0
RUN cd rnt/build && cp /rnt/build_ocloc/bin/ocloc ./bin && \
    make -j8

#
# Cross compile IGC
#
RUN sed -i -E 's|linux_resource_linker|/build_native/IGC/llvm/build/src/bin/linux_resource_linker|' llvm-project/llvm/projects/opencl-clang/cl_headers/CMakeLists.txt
RUN sed -i -E 's|target_link_libraries\(IGA_EXE PUBLIC IGA_SLIB "-lrt"\)|target_link_libraries\(IGA_EXE PUBLIC IGA_SLIB\)|' igc/visa/iga/IGAExe/CMakeLists.txt
RUN sed -i -E 's|-lrt|\n|' igc/IGC/CMakeLists.txt
RUN sed -i -E 's|target_link_libraries\(GenX_IR_Exe rt dl\)|target_link_libraries\(GenX_IR_Exe dl\)|' igc/visa/CMakeLists.txt
RUN sed -i -E 's|llvm-tblgen|/build_native/IGC/llvm/build/src/bin/llvm-tblgen|' llvm-project/llvm/projects/opencl-clang/CMakeLists.txt
RUN sed -i -E 's|UNIX|UNIX OR ANDROID|' igc/IGC/BiFModule/CMakeLists.txt

RUN sed -i -E 's|COMMAND clang-tool|COMMAND /build_native/IGC/llvm/build/src/bin/clang|' igc/IGC/BiFModule/CMakeLists.txt
RUN sed -i -E 's|DEPENDS clang-tool|DEPENDS /build_native/IGC/llvm/build/src/bin/clang|' igc/IGC/BiFModule/CMakeLists.txt
RUN sed -i -E 's|COMMAND llvm-link|COMMAND /build_native/IGC/llvm/build/src/bin/llvm-link|' igc/IGC/BiFModule/CMakeLists.txt

RUN apt-get install -y nano patchelf

RUN sed -i -E 's|\$<TARGET_FILE:\$\{IGC_BUILD__PROJ__ElfPackager\}> -includeSizet -funcList \$\{CMAKE_CURRENT_SOURCE_DIR\}/function_bin.txt \$\{IGC_BUILD__BIF_DIR\}/OCLBiFImpl.bc \$\{IGC_BUILD__BIF_DIR\}/igdclbif.bin|echo 1|' igc/IGC/ElfPackager/CMakeLists.txt

RUN cd build && rm -rf * && \
    mkdir -p NATIVE/bin && \
    cp /build_native/IGC/llvm/build/src/bin/llvm-tblgen NATIVE/bin && \
    cp /build_native/IGC/llvm/build/src/bin/clang-tblgen NATIVE/bin && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_TOOLCHAIN_FILE=../android-ndk-r20/build/cmake/android.toolchain.cmake \
        -DANDROID_ABI=x86_64 \
        -DCMAKE_CXX_FLAGS="-Wno-macro-redefined -Wno-user-defined-warnings -Wno-deprecated" \
    ../igc

#
# RUN cd igc && \
#     git clone -b android_fix https://github.com/dkurt/gmmlib
#     git clone -b fix_android https://github.com/dkurt/intel-compute-runtime
#
# RUN cd build && \
#     mkdir -p NATIVE/bin && \
#     cp /build_native/IGC/llvm/build/src/bin/llvm-tblgen NATIVE/bin && \
#     cp /build_native/IGC/llvm/build/src/bin/clang-tblgen NATIVE/bin && \
#     cmake \
#       -DCMAKE_BUILD_TYPE=Debug \
#       -DCMAKE_TOOLCHAIN_FILE=../android-ndk-r20/build/cmake/android.toolchain.cmake \
#       -DANDROID_ABI=x86_64 \
#       -DSKIP_ALL_ULT=ON \
#       -DSKIP_UNIT_TESTS=ON \
#       -DRUN_TEST_SUITE=OFF \
#       -DCMAKE_CXX_FLAGS="-Wno-macro-redefined -Wno-user-defined-warnings -Wno-deprecated" \
#     /compute-runtime/
#
RUN cd build && make -j8 fcl_dll igc_dll

RUN cd rnt/build && make install intel.icd

RUN cd gmmlib && mkdir build_android && cd build_android && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_TOOLCHAIN_FILE=../android-ndk-r20/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=x86_64 \
      -DRUN_TEST_SUITE=OFF .. && \
    make -j8

RUN mkdir ocl && cd ocl && \
    cp /build/IGC/Release/libigdfcl.so libigdfcl.so.1 && patchelf --set-soname libigdfcl.so.1 libigdfcl.so.1 && \
    cp /build/IGC/Release/libigc.so libigc.so.1 && patchelf --set-soname libigc.so.1 libigc.so.1 && \
    cp /gmmlib/build_android/Source/GmmLib/libigdgmm.so libigdgmm.so.11 && patchelf --set-soname libigdgmm.so.11 libigdgmm.so.11 && \
    cp /build/IGC/llvm/build/src/lib/libopencl-clang.so . && \
    cp /rnt/build/bin/libigdrcl.so . && \
    cp /rnt/build/intel.icd .
