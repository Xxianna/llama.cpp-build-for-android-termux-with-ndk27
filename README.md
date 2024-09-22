# llama.cpp-build-for-android-termux-with-ndk27
源码：https://github.com/ggerganov/llama.cpp.git

## 当前功能
- 支持armv8a设备，编译来自ndk。在termux中直接使用
- 使用cpu运行
- ggml的vulkan版本，使用vulkan运行，已在8gen3和kirin990上测试 **无法工作**



## 说明 v0.1-cpu

单纯编译时候遇到很多坑，遂开此仓库。本仓库是在wsl下使用ndk27编译的

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

## 说明 v0.2-ggmlvk
- 问题：找不到vulkan_library
  - 将安卓api版本改到大于24
- 问题：找不到vulkan/vulkan.h
  - 在cmake里find_package(vulkan)前把vulkan头文件地址设为windows vulkan sdk的最新的头文件地址
- 问题：编译时，vulkan-shaders-gen运行不了
  - 只设置了ndk的交叉编译，只对arm安卓产生可执行程序。需要索引到vulkan-shaders-gen目录，先编译一个当前平台版本并加入PATH
- 问题：加载模型失败，不能运行
  - 无解。该版本作废。gpu加速建议使用ggml的kompute方案（也是vulkan，不知道对8g3/kirin990是否可行，可能结果一样）或ncnn（在8g3上效率常常比cpu高，但输出略有差别，以及模型转换较困难
```
 cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-29 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod -DGGML_VULKAN=1 ..
```
- 参考链接：https://coderfan.net/deploy-llama-cpp-in-android-system.html#:~:text=1.%20vulkan-
![image](https://github.com/user-attachments/assets/607a6d5f-4ffa-4caa-ab41-556328edfa30)

 
## 说明 v0.3-ggml-kompute-vulkan
- xxd也需要另外设编译器编译
- .comp->.spv->.h过程可能出问题，着色器编译
- 问题较多，未产生成果

## 附：
目前用最新版chrome，特性全开，手机上也不能运行mlc或者huggingface提供的webgpullm。端侧异构道阻且长











