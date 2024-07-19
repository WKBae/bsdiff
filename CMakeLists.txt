cmake_minimum_required(VERSION 3.15 FATAL_ERROR)
project(bsdiff VERSION 0.1)

# Specify the C standard
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED True)

# Find BZip2 library
find_package(BZip2 REQUIRED)

# Download CPM.cmake
file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.40.0/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=7b354f3a5976c4626c876850c93944e52c83ec59a159ae5de5be7983f0e17a2a
)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

# Helper macro to check the result and fail if not found
macro(check_or_fail var_name)
    if (NOT ${var_name})
        message(FATAL_ERROR "Missing required ${var_name}")
    endif()
endmacro()

# Check for required headers
include(CheckIncludeFiles)
check_include_files("fcntl.h;limits.h;stddef.h;stdint.h;stdlib.h;string.h;unistd.h" HAVE_REQUIRED_HEADERS)
check_or_fail(HAVE_REQUIRED_HEADERS)

# Check for required types
include(CheckTypeSize)
check_type_size("int64_t" HAVE_INT64_T)
check_or_fail(HAVE_INT64_T)
check_type_size("off_t" HAVE_OFF_T)
check_or_fail(HAVE_OFF_T)
check_type_size("size_t" HAVE_SIZE_T)
check_or_fail(HAVE_SIZE_T)
check_type_size("uint8_t" HAVE_UINT8_T)
check_or_fail(HAVE_UINT8_T)

# Check for required functions
include(CheckSymbolExists)
check_symbol_exists("malloc" "stdlib.h" HAVE_MALLOC)
check_or_fail(HAVE_MALLOC)
check_symbol_exists("memset" "string.h" HAVE_MEMSET)
check_or_fail(HAVE_MEMSET)

# Define source files for the executables
add_executable(bsdiff bsdiff.c)
add_executable(bspatch bspatch.c)

# Set compiler flags for executables
target_compile_definitions(bsdiff PRIVATE BSDIFF_EXECUTABLE)
target_compile_definitions(bspatch PRIVATE BSPATCH_EXECUTABLE)

# Link BZip2 library to executables
target_link_libraries(bsdiff BZip2::BZip2)
target_link_libraries(bspatch BZip2::BZip2)

# Define source files for the library
add_library(libbsdiff STATIC bsdiff.c bspatch.c)

# Remove the "lib" prefix from the library name
set(CMAKE_STATIC_LIBRARY_PREFIX "")

# Set compiler flags for the library
target_compile_options(libbsdiff PRIVATE ${CMAKE_C_FLAGS})

# Install targets
install(TARGETS bsdiff bspatch libbsdiff
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# Install headers and additional files
install(FILES
    bsdiff.h
    bspatch.h
    LICENSE
    README.md
    DESTINATION include
)
