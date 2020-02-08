cmake_minimum_required(VERSION 3.1)
project(lodepng)

# define target to install and link tests against
add_library(lodepng
  lodepng.h
  lodepng.cpp)
target_include_directories(lodepng
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)
add_library(lodepng_util
  lodepng_util.h
  lodepng_util.cpp)
target_link_libraries(lodepng_util PUBLIC lodepng)
add_executable(pngdetail
  pngdetail.cpp)
target_link_libraries(pngdetail PRIVATE lodepng_util)

# ALIAS same as in configure file
# the ALIAS can be used to create examples which use the same syntax as a client
# application, which uses `find_package(lodepng CONFIG)`
# create the alias lodepng::lodepng
add_library(lodepng::lodepng ALIAS lodepng)
add_library(lodepng::lodepng_util ALIAS lodepng_util)

option(ENABLE_TESTING "enable creation of unittest" OFF)
if(ENABLE_TESTING)
  enable_testing()
  add_executable(lodepng_unittest lodepng_unittest.cpp)
  # add compiler flags for test target only for GCC and Clang
  target_compile_options(lodepng_unittest PRIVATE
    $<BUILD_INTERFACE:
      $<$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>:
        -Werror -Wall -pedantic>>
    )

  add_test(test_lodepng_unittest lodepng_unittest)
  target_link_libraries(lodepng_unittest PRIVATE lodepng_util)
endif()

option(BUILD_EXAMPLES "build examples using lodepng" OFF)
if(BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

option(BUILD_BENCHMARK "build benchmark for lodepng, requires SDL2" OFF)
if(BUILD_BENCHMARK)
  add_executable(lodepng_benchmark lodepng_benchmark.cpp)
  target_link_libraries(lodepng_benchmark PRIVATE lodepng::lodepng)
  # SDL2 Dependency
  find_package(SDL2 CONFIG REQUIRED)
  if (TARGET SDL2::SDL2)
    message(STATUS "using TARGET SDL2::SDL2")
    target_link_libraries(lodepng_benchmark PUBLIC SDL2::SDL2)
  else()
    message(STATUS "no TARGET SDL2::SDL2, or SDL2, using variables")
    target_include_directories(lodepng_benchmark PUBLIC $<BUILD_INTERFACE:${SDL2_INCLUDE_DIRS}>)
    target_link_libraries(lodepng_benchmark PUBLIC ${SDL2_LIBRARIES})
  endif()
  add_custom_target(benchmark COMMAND lodepng_benchmark)
endif()

# create install target
include(GNUInstallDirs)
install(
  TARGETS
  lodepng
  lodepng_util
  pngdetail
  EXPORT lodepng-targets
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} # set include path for installed library target
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )
install(
  FILES
  lodepng.h
  lodepng_util.h
  DESTINATION include
  )

# Include module for fuctions
# - 'write_basic_package_version_file'
# - 'configure_package_config_file'
include(CMakePackageConfigHelpers)

# generate and install termcolor-config.cmake file
# Configure '<PROJECT-NAME>-config.cmake'
configure_package_config_file(
  "cmake/config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/generated/lodepng-config.cmake"
  INSTALL_DESTINATION "lib/cmake/lodepng"
)
# install config file
install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/generated/lodepng-config.cmake"
  DESTINATION "lib/cmake/lodepng"
)
# install targets file
install(
  EXPORT "lodepng-targets"
  NAMESPACE "lodepng::"
  DESTINATION "lib/cmake/lodepng"
)
