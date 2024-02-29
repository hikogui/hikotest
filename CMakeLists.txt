# Copyright Take Vos 2024
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at https://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.25)
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

set(HIKOTEST_VERSION "1.0.0")
string(REGEX MATCH "[0-9]+" HIKOTEST_MAJOR_VERSION "${HIKOTEST_VERSION}")
project(hikotest VERSION "${HIKOTEST_VERSION}" LANGUAGES "CXX")

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(hikotest)
target_sources(hikotest PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/src/hikotest/hikotest.cpp")
target_sources(hikotest PUBLIC
    FILE_SET hikotest_include_files TYPE HEADERS BASE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/src/" FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/hikotest/hikotest.hpp")

target_compile_features(hikotest PUBLIC cxx_std_23)
set_target_properties(hikotest PROPERTIES DEBUG_POSTFIX "d")
set_target_properties(hikotest PROPERTIES RELWITHDEBINFO_POSTFIX "i")
set_target_properties(hikotest PROPERTIES VERSION "${HIKOTEST_VERSION}")
set_target_properties(hikotest PROPERTIES SOVERSION "${HIKOTEST_MAJOR_VERSION}")
set_target_properties(hikotest PROPERTIES FRAMEWORK TRUE)
set_target_properties(hikotest PROPERTIES MACOSX_FRAMEWORK_IDENTIFIER "org.hikogui.hikotest")

target_include_directories(hikotest PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
    "$<INSTALL_INTERFACE:include>")

set(INSTALL_CMAKEDIR "${CMAKE_INSTALL_DATADIR}/hikotest/CMake")
set(INSTALL_MODULEDIR "module")

install(TARGETS hikotest EXPORT hikotest
    FILE_SET hikotest_include_files
    RUNTIME
    ARCHIVE
    LIBRARY
    RESOURCE DESTINATION "${INSTALL_RESOURCEDIR}"
)

install(
    EXPORT hikotest
    DESTINATION "${INSTALL_CMAKEDIR}"
    NAMESPACE hikotest::
    FILE "hikotestTargets.cmake"
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/CMake/packaging/hikotestConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/hikotestConfig.cmake"
    INSTALL_DESTINATION "${INSTALL_CMAKEDIR}"
)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/hikotestConfigVersion.cmake"
    COMPATIBILITY SameMajorVersion
)
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/hikotestConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/hikotestConfigVersion.cmake"
    DESTINATION "${INSTALL_CMAKEDIR}"
)

