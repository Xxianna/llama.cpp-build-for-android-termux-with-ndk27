# llama.cpp-build-for-android-termux-with-ndk27
源码：https://github.com/ggerganov/llama.cpp.git

## 当前功能
- 支持armv8a设备，编译来自ndk。在termux中直接使用
- 使用cpu运行



## 说明

单纯编译时候遇到很多坑，遂开此仓库

- 例如在termux中试图编译vulkan版本。但clang18编译出错，gcc下载太慢
- windows下ndk编译似乎出现奇奇怪怪的问题
- 在WSL中编译时，参考https://github.com/ggerganov/llama.cpp/blob/master/docs/android.md ，下载ndk配好位置即可
```
mkdir buildandroid
cd buildandroid
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod -DGGML_STATIC=1 ..
make
cd bin
```
- 为了方便开了静态编译，并且指定openmp静态链接
```
# if (GGML_OPENMP)
#     find_package(OpenMP)
#     if (OpenMP_FOUND)
#         message(STATUS "OpenMP found")

#         add_compile_definitions(GGML_USE_OPENMP)

#         list(APPEND GGML_EXTRA_LIBS_PRIVATE OpenMP::OpenMP_C OpenMP::OpenMP_CXX)

#         if (GGML_MUSA)
#             list(APPEND GGML_EXTRA_INCLUDES     "/usr/lib/llvm-10/include/openmp")
#             list(APPEND GGML_EXTRA_LIBS_PRIVATE "/usr/lib/llvm-10/lib/libomp.so")
#         endif()
#     else()
#         message(WARNING "OpenMP not found")
#     endif()
# endif()

list(APPEND GGML_EXTRA_LIBS_PRIVATE -fopenmp -static-openmp)
```
- 如果不静态编译，有人遇到这样的问题
  - https://github.com/ggerganov/llama.cpp/issues/8436
  - https://github.com/ggerganov/llama.cpp/issues/8428
- 但并非不能解决。单纯把.so移动到目录下是无效的，这方面linux不像windows。但再通过```export LD_LIBRARY_PATH=./```可以使用
- 此外，可根据```ldd llama-cli```的输出查看当前程序依赖的其他库地址以及还缺什么库。安卓的动态库很多在```/system/lib64```下，但需要root才能访问













