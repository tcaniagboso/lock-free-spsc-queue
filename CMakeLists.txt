cmake_minimum_required(VERSION 3.16)

project(lock_free_spsc_queue LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(lf_queue_bench
        src/main.cpp
)

target_include_directories(lf_queue_bench
        PRIVATE
        include
        benchmark
)

target_compile_options(lf_queue_bench PRIVATE
        -O3
        -march=native
)

target_link_libraries(lf_queue_bench PRIVATE pthread)