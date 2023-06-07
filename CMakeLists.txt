cmake_minimum_required(VERSION 3.15)

project(minisat)

#--------------------------------------------------------------------------------------------------
# Configurable options:

option(USE_SORELEASE   "Use SORELEASE in shared library filename." ON)

#--------------------------------------------------------------------------------------------------
# Library version:

set(MINISAT_SOMAJOR   2)
set(MINISAT_SOMINOR   1)
set(MINISAT_SORELEASE 0)

# Compute VERSION and SOVERSION:
if (USE_SORELEASE)
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR}.${MINISAT_SORELEASE})
else()
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR})
endif()
set(MINISAT_SOVERSION ${MINISAT_SOMAJOR})

# Reference specific library paths used during linking for install
if (POLICY CMP0042)
  # Enable `MACOSX_RPATH` by default.
  cmake_policy(SET CMP0042 NEW)
endif()
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

#--------------------------------------------------------------------------------------------------
# Dependencies:

find_package(ZLIB)
find_package(Threads)
include_directories(${ZLIB_INCLUDE_DIR})
include_directories(${minisat_SOURCE_DIR})
include (GenerateExportHeader)

#--------------------------------------------------------------------------------------------------
# Compile flags:

add_definitions(-D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS)

set(MINISAT_EXPORT_NAME "minisatTargets")

# -----------------------------------------------------------------------------
# Add uninstall target for makefiles
# -----------------------------------------------------------------------------
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY
)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)


#--------------------------------------------------------------------------------------------------
# Build Targets:

set(MINISAT_LIB_SOURCES
    minisat/utils/Options.cc
    minisat/utils/System.cc
    minisat/core/Solver.cc
    minisat/simp/SimpSolver.cc)

add_library(minisat ${MINISAT_LIB_SOURCES})
target_link_libraries(minisat PUBLIC ZLIB::ZLIB Threads::Threads)

add_executable(minisat_core minisat/core/Main.cc)
add_executable(minisat_simp minisat/simp/Main.cc)


target_link_libraries(minisat_core minisat)
target_link_libraries(minisat_simp minisat)

set_target_properties(minisat
  PROPERTIES
    OUTPUT_NAME "minisat"
    VERSION ${MINISAT_VERSION}
    SOVERSION ${MINISAT_SOVERSION})

set_target_properties(minisat_simp       PROPERTIES OUTPUT_NAME "minisat")

#--------------------------------------------------------------------------------------------------
# Installation targets:

install(TARGETS minisat minisat_core minisat_simp
        EXPORT ${MINISAT_EXPORT_NAME}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY minisat/mtl minisat/utils minisat/core minisat/simp
        DESTINATION include/minisat
        FILES_MATCHING PATTERN "*.h")

# Setup for export
set(MINISAT_TARGETS_FILENAME "minisatTargets.cmake")
set(MINISAT_CONFIG_FILENAME "minisatConfig.cmake")

# Export targets
export(
    TARGETS minisat
    FILE "${CMAKE_CURRENT_BINARY_DIR}/${MINISAT_TARGETS_FILENAME}"
)

# Create minisatConfig file
set(EXPORT_TYPE "Build-tree")
set(CONF_INCLUDE_DIRS "${minisat_SOURCE_DIR}/")
configure_file(minisatConfig.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${MINISAT_CONFIG_FILENAME}" @ONLY
)

# Export this package to the CMake user package registry
# Now the user can just use find_package(minisat) on their system
export(PACKAGE minisat)

set(DEF_INSTALL_CMAKE_DIR lib/cmake/minisat)
set(MINISAT_INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
    "Installation directory for minisat CMake files")

# Create minisatConfig file
set(EXPORT_TYPE "installed")
configure_file(minisatConfig.cmake.in
   "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/${MINISAT_CONFIG_FILENAME}" @ONLY
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/${MINISAT_CONFIG_FILENAME}"
    DESTINATION "${MINISAT_INSTALL_CMAKE_DIR}"
)

# Install the export set for use with the install-tree
install(
    EXPORT ${MINISAT_EXPORT_NAME}
    DESTINATION "${MINISAT_INSTALL_CMAKE_DIR}"
)
