cmake_minimum_required(VERSION 3.20)
project(SafeFlowDecomposition VERSION 1.0.0 LANGUAGES CXX)

# --- Global C++20 Enforcement ---
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- High-Performance Optimization & Hardware Intrinsics ---
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    # Native tuning for HFT-grade performance
    add_compile_options(-O3 -march=native -mtune=native)
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
    
    # Enable AVX-512 (Crucial for the vectorized SPFA backend)
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-mavx512f" COMPILER_SUPPORTS_AVX512)
    if(COMPILER_SUPPORTS_AVX512)
        add_compile_options(-mavx512f -mavx512dq -mavx512bw)
        add_definitions(-DHAS_AVX512)
    endif()
    
    # Memory Safety Sanitizers for Debugging
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-fsanitize=address,undefined -g)
        add_link_options(-fsanitize=address,undefined)
    endif()
endif()

# --- Google Test Integration (Automated Fetch) ---
include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# --- Include Directories ---
# Using project-relative paths to handle the new "Safe Flow" structure
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/graphs)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/baseline)

# --- Build Targets ---

# 1. The Core Library (The Optimized Infrastructure)
file(GLOB_RECURSE LIB_SOURCES "src/*.cpp")
add_library(safe_flow_core STATIC ${LIB_SOURCES})

# 2. The Benchmark Executable
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/benchmarks/benchmark_safe_flow.cpp")
    add_executable(benchmark_safe_flow benchmarks/benchmark_safe_flow.cpp)
    target_link_libraries(benchmark_safe_flow PRIVATE safe_flow_core)
endif()

# 3. Unit Tests
enable_testing()
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests/test_safe_flow.cpp")
    add_executable(safe_flow_tests 
        tests/test_safe_flow.cpp
        # Add other relevant test files here
    )
    target_link_libraries(safe_flow_tests PRIVATE safe_flow_core gtest_main)
    add_test(NAME AllTests COMMAND safe_flow_tests)
endif()