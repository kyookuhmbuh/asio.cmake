# asio (https://github.com/chriskohlhoff/asio) CMake support

project(
  asio
  LANGUAGES CXX
  VERSION 1.36.0
)

# Sources

include(FetchContent)

FetchContent_Declare(
  asio
  GIT_REPOSITORY https://github.com/chriskohlhoff/asio.git
  GIT_TAG asio-1-36-0
)

FetchContent_MakeAvailable(asio)

# Header-Only library

add_library(asio_header_only INTERFACE)
add_library(asio::asio_header_only ALIAS asio_header_only)

target_compile_features(asio_header_only INTERFACE cxx_std_20)

file(GLOB_RECURSE asio_ALL_HEADERS ${asio_SOURCE_DIR}/asio/include/*)

target_sources(
  asio_header_only
  PUBLIC
    FILE_SET HEADERS
    BASE_DIRS ${asio_SOURCE_DIR}/asio/include
    FILES
      ${asio_ALL_HEADERS}
)

# Separate Compilation (static) library

add_library(asio_separate_compilation STATIC)
add_library(asio::asio_separate_compilation ALIAS asio_separate_compilation)

target_compile_definitions(
  asio_separate_compilation
  PUBLIC
    ASIO_SEPARATE_COMPILATION
)

target_link_libraries(
  asio_separate_compilation
  PUBLIC
    asio::asio_header_only
)

target_sources(
  asio_separate_compilation
  PRIVATE
    ${asio_SOURCE_DIR}/asio/src/asio.cpp
)

# Separate Compilation (shared) library

add_library(asio_separate_compilation_shared SHARED)
add_library(
  asio::asio_separate_compilation_shared
  ALIAS asio_separate_compilation_shared
)

target_compile_definitions(
  asio_separate_compilation_shared
  PUBLIC
    ASIO_SEPARATE_COMPILATION
    ASIO_DYN_LINK
)

target_link_libraries(
  asio_separate_compilation_shared
  PUBLIC
    asio::asio_header_only
)

target_sources(
  asio_separate_compilation_shared
  PRIVATE
    ${asio_SOURCE_DIR}/asio/src/asio.cpp
)

# Installation

include(GNUInstallDirs)

install(
  TARGETS
    asio_header_only
    asio_separate_compilation
    asio_separate_compilation_shared
  EXPORT asio_targets
  FILE_SET HEADERS
)

install(
  EXPORT asio_targets
  FILE asioConfig.cmake
  NAMESPACE asio::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/asio
)
