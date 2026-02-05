# asio (https://github.com/chriskohlhoff/asio) CMake support

cmake_minimum_required(VERSION 3.23)

project(
  asio
  LANGUAGES CXX
  VERSION 1.36.0
)

# Sources

include(FetchContent)

FetchContent_Declare(
  asio_repo
  GIT_REPOSITORY https://github.com/chriskohlhoff/asio.git
  GIT_TAG asio-1-36-0
  GIT_SHALLOW TRUE
)

FetchContent_MakeAvailable(asio_repo)

# Header-Only library

add_library(${PROJECT_NAME} INTERFACE)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_20)

target_compile_definitions(
  ${PROJECT_NAME}
  INTERFACE
    ASIO_STANDALONE
    ASIO_NO_DEPRECATED
)

# Fix warning : "Please define _WIN32_WINNT or _WIN32_WINDOWS appropriately."
# https://stackoverflow.com/a/71452642/23061769
if(WIN32 AND CMAKE_SYSTEM_VERSION)
  macro(get_win32_winnt outvar)
    string(REGEX MATCH "^([0-9]+)\\.([0-9]+)" ${outvar} ${CMAKE_SYSTEM_VERSION})
    math(
      EXPR ${outvar} "(${CMAKE_MATCH_1} << 8) + ${CMAKE_MATCH_2}"
      OUTPUT_FORMAT HEXADECIMAL
    )
  endmacro()

  if(NOT DEFINED _WIN32_WINNT)
    get_win32_winnt(ver)
    set(_WIN32_WINNT ${ver})
    message(STATUS "Set _WIN32_WINNT=${_WIN32_WINNT}")
  endif()

  target_compile_definitions(
    ${PROJECT_NAME}
    INTERFACE
      _WIN32_WINNT=${_WIN32_WINNT}
      WIN32_LEAN_AND_MEAN
  )
endif()

file(
  GLOB_RECURSE asio_ALL_HEADERS
  ${asio_repo_SOURCE_DIR}/asio/include/*.hpp
  ${asio_repo_SOURCE_DIR}/asio/include/*.ipp
)

target_sources(
  ${PROJECT_NAME}
  INTERFACE
    FILE_SET HEADERS
    BASE_DIRS ${asio_repo_SOURCE_DIR}/asio/include
    FILES
      ${asio_ALL_HEADERS}
)

set_target_properties(
  ${PROJECT_NAME}
  PROPERTIES
    VERIFY_INTERFACE_HEADER_SETS ON
)

find_package(Threads REQUIRED)
target_link_libraries(asio INTERFACE Threads::Threads)

# Installation

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}_targets
  FILE_SET HEADERS
  COMPONENT ${PROJECT_NAME}
)

install(
  EXPORT ${PROJECT_NAME}_targets
  FILE ${PROJECT_NAME}Config.cmake
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
  COMPONENT ${PROJECT_NAME}
)
