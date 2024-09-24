cmake_minimum_required(VERSION 3.19)
project(NVIDIA_SGEMM_PRACTICE LANGUAGES CXX CUDA) # languages cxx cuda表示项目中要用到c++和cuda两种语言
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(CUDA REQUIRED)

# ensure cuda is available
include(CheckLanguage)
check_language(CUDA) # 这两行命令用于验证cuda是否有用

set(CMAKE_CXX_STANDARD 20)
set(CUDA_COMPUTE_CAPABILITY 80)

# in debug mode, add debug symbols to device code
# this disables most optimizations and kills performance
add_compile_options("$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:CUDA>>:-G;-src-in-ptx>")
# add_compile_options("--ptxas-options=-v")

# Configure header file search paths
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${PROJECT_SOURCE_DIR}/src)
# Configure the source file path to be compiled
aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC) # 将目录 ${PROJECT_SOURCE_DIR}/sr 的值作为一个变量赋值给 SRC

message(STATUS "cuda version: " ${CUDA_VERSION_STRING}) # 打印日志
message(STATUS "cuda library: " ${CUDA_LIBRARIES}) # 打印日志
message(STATUS "cuda cublas library: " ${CUDA_CUBLAS_LIBRARIES}) # 打印日志

# generate executable
add_executable(sgemm sgemm.cu ${SRC}) # 基于sgemm.cu 文件和 ${SRC} 文件 编译生成可执行文件 sgemm
# 用于指定cuda的compute architectures, 主要是用于编译电脑的显卡比需要部署的显卡更好的时候，如果采用默认的计算架构的话，可能在部署的电脑上无法执行
set_target_properties(sgemm PROPERTIES CUDA_ARCHITECTURES ${CUDA_COMPUTE_CAPABILITY}) 
target_link_libraries(sgemm ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES})

add_executable(cuBLAS_sgemm cuBLAS_sgemm.cu )
set_target_properties(sgemm PROPERTIES CUDA_ARCHITECTURES ${CUDA_COMPUTE_CAPABILITY})
target_link_libraries(cuBLAS_sgemm ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES})

add_executable(simplest_kernel simplest_kernel.cu)
set_target_properties(sgemm PROPERTIES CUDA_ARCHITECTURES ${CUDA_COMPUTE_CAPABILITY})
target_link_libraries(simplest_kernel ${CUDA_LIBRARIES})
