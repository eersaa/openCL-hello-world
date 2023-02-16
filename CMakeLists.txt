cmake_minimum_required(VERSION 3.25)
project(hello-world)

find_package(OpenCL REQUIRED)

add_executable(${PROJECT_NAME} hello-world.cpp)

add_library(compiler_flags INTERFACE)
target_compile_features(compiler_flags INTERFACE cxx_std_23)

set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)

target_link_libraries(${PROJECT_NAME} OpenCL::OpenCL compiler_flags)

# Add OpenCL kernel file to the target
set_source_files_properties(hello-world.cl PROPERTIES HEADER_ONLY TRUE)
add_custom_target(kernel_header DEPENDS hello-world.cl)
add_dependencies(hello-world kernel_header)

# Copy OpenCL kernel file to the binary directory
add_custom_command(
    TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
    ${CMAKE_CURRENT_SOURCE_DIR}/hello-world.cl
    ${CMAKE_CURRENT_BINARY_DIR}/hello-world.cl
    )
