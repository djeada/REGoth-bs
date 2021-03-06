cmake_minimum_required (VERSION 3.13.0)
project (REGoth)

option(REGOTH_USE_SYSTEM_BSF "Whether to use the system installed bsf via find_package." OFF)

if (NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(FATAL_ERROR "REGoth does not support to be built on architectures other than 64 bit.")
endif()

###############################################################################
#                                Configuration                                #
###############################################################################

# Make sure to use the C++14 standard
set(CMAKE_CXX_STANDARD 14)

# Set up binary output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Add compiler flags required by bsf (See: https://www.bsframework.io/docs/build.html)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR 
   CMAKE_CXX_COMPILER_ID MATCHES "AppleClang" OR
   CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    add_compile_options(-fno-rtti)

    # So that REGoth-Symbols will show up in the bs:f-crash handler
    add_link_options(-rdynamic)
endif()

###############################################################################
#                            Add external libraries                           #
###############################################################################

# Let CMake know where to find the Findbsf.cmake file
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

if (REGOTH_USE_SYSTEM_BSF)
  find_package(bsf REQUIRED)
else()
  # Need to reset RPATH-settings to default again because bsf has messed with them.
  # See https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling
  # use, i.e. don't skip the full RPATH for the build tree
  set(CMAKE_SKIP_BUILD_RPATH FALSE)

  # when building, don't use the install RPATH already
  # (but later on when installing)
  set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

  # Enable exceptions in bsf as some dependencies (like cxxopts) rely on them.
  set(BSF_ENABLE_EXCEPTIONS ON CACHE BOOL "" FORCE)

  add_subdirectory(lib/bsf)

endif()

# BsZenLib will try to find bs:f for itself as well. Since we already have a bsf-target, skip that.
set(BSZENLIB_SKIP_FIND_BSF ON CACHE BOOL "" FORCE)
add_subdirectory(lib/BsZenLib)

# cxxopts as library to parse command line options.
set(CXXOPTS_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE) # Disable unnecessary targets for examples.
set(CXXOPTS_BUILD_TESTS OFF CACHE BOOL "" FORCE) # Disable unnecessary targets for unit tests.
add_definitions(-DCXXOPTS_NO_RTTI) # Disable RTTI dependent instructions.
add_subdirectory(lib/cxxopts)

# Assertions.
if(CMAKE_BUILD_TYPE MATCHES Debug OR CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
  add_definitions(-DREGOTH_ENABLE_ASSERTIONS)
endif()

###############################################################################
#                               Add Samples, etc                              #
###############################################################################
add_subdirectory(docs-source)
add_subdirectory(src)
