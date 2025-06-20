# Filename rules in cuda backend:
#
# * Use .cu/.cuh if code contains device code, and .cpp/.h if not.
# * Device-only kernel code should be put in kernels/ subdir.
# * Files in kernels/ subdir should not include files outside.
target_sources(
  mlx
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/allocator.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/copy.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/device.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/eval.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/event.cu
          ${CMAKE_CURRENT_SOURCE_DIR}/fence.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/kernel_utils.cu
          ${CMAKE_CURRENT_SOURCE_DIR}/primitives.cu
          ${CMAKE_CURRENT_SOURCE_DIR}/slicing.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/utils.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/worker.cpp)

# Enable defining device lambda functions.
target_compile_options(mlx
                       PRIVATE "$<$<COMPILE_LANGUAGE:CUDA>:--extended-lambda>")

# Compute capability 7 is required for synchronization between CPU/GPU with
# managed memory. TODO: Add more architectures for potential performance gain.
set(MLX_CUDA_ARCHITECTURES
    "70;80"
    CACHE STRING "CUDA architectures")
message(STATUS "CUDA architectures: ${MLX_CUDA_ARCHITECTURES}")
set_target_properties(mlx PROPERTIES CUDA_ARCHITECTURES
                                     "${MLX_CUDA_ARCHITECTURES}")

# Use fixed version of CCCL.
FetchContent_Declare(
  cccl
  URL "https://github.com/NVIDIA/cccl/releases/download/v2.8.1/cccl-v2.8.1.zip")
FetchContent_MakeAvailable(cccl)
target_include_directories(mlx BEFORE PRIVATE "${cccl_SOURCE_DIR}/include")

# Use fixed version of NVTX.
FetchContent_Declare(
  nvtx3
  GIT_REPOSITORY https://github.com/NVIDIA/NVTX.git
  GIT_TAG v3.1.1
  GIT_SHALLOW TRUE
  SOURCE_SUBDIR c EXCLUDE_FROM_ALL)
FetchContent_MakeAvailable(nvtx3)
target_link_libraries(mlx PUBLIC $<BUILD_INTERFACE:nvtx3-cpp>)

# Make cuda runtime APIs available in non-cuda files.
find_package(CUDAToolkit REQUIRED)
target_include_directories(mlx PRIVATE ${CUDAToolkit_INCLUDE_DIRS})

# Suppress nvcc warnings on MLX headers.
target_compile_options(mlx PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:-Xcudafe
                                   --diag_suppress=997>)
