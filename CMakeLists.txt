# CMakeLists.txt
#
# Copyright 2013, 2014 by
# David Turner, Robert Wilhelm, and Werner Lemberg.
#
# Written by John Cary <cary@txcorp.com>
#
# This file is part of the FreeType project, and may only be used, modified,
# and distributed under the terms of the FreeType project license,
# LICENSE.TXT.  By continuing to use, modify, or distribute this file you
# indicate that you have read the license and understand and accept it
# fully.
#
#
# Say
#
#   cmake CMakeLists.txt
#
# to create a Makefile that builds a static version of the library.
#
# For a dynamic library, use
#
#   cmake CMakeLists.txt -DBUILD_SHARED_LIBS:BOOL=true
#
# For a framework on OS X, use
#
#   cmake CMakeLists.txt -DBUILD_FRAMEWORK:BOOL=true -G Xcode
#
# instead.
#
# For an iOS static library, use
#
#   cmake CMakeLists.txt -DIOS_PLATFORM=OS -G Xcode
#
# or
#
#   cmake CMakeLists.txt -DIOS_PLATFORM=SIMULATOR -G Xcode
#
# Please refer to the cmake manual for further options, in particular, how
# to modify compilation and linking parameters.
#
# Some notes.
#
# . `cmake' will overwrite FreeType's original (top-level) `Makefile' file.
#
# . You can use `cmake' directly on a freshly cloned FreeType git
#   repository.
#
# . `CMakeLists.txt'  is provided as-is since it is not used by the
#   developer team.


cmake_minimum_required(VERSION 3.16)
project(freetype LANGUAGES C)

# Build Freetype as a static library
add_definitions(-DFT2_BUILD_LIBRARY)

# Public include directory
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

# Auto-generated ftconfig.h
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/freetype2)

execute_process(
  COMMAND sed -e "s/FT_CONFIG_OPTIONS_H/<ftoption.h>/" 
              -e "s/FT_CONFIG_STANDARD_LIBRARY_H/<ftstdlib.h>/" 
              -e "s?/undef ?#undef ?"
  INPUT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/builds/unix/ftconfig.in
  OUTPUT_FILE ${CMAKE_CURRENT_BINARY_DIR}/include/freetype2/ftconfig.h
)

include_directories(BEFORE ${CMAKE_CURRENT_BINARY_DIR}/include/freetype2)

# Collect headers
file(GLOB PUBLIC_HEADERS "include/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "include/config/*.h")
file(GLOB PRIVATE_HEADERS "include/internal/*.h")

# Collect sources
file(GLOB_RECURSE FREETYPE_SOURCES
  src/*.c
)

# Build the library
add_library(freetype STATIC
  ${PUBLIC_HEADERS}
  ${PUBLIC_CONFIG_HEADERS}
  ${PRIVATE_HEADERS}
  ${FREETYPE_SOURCES}
)

# Ensure PIC for static builds
set_target_properties(freetype PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Install headers
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
  DESTINATION include/freetype2
  PATTERN "internal" EXCLUDE
)

# Install library
install(TARGETS freetype
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
