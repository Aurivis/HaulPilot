# SPDX-License-Identifier: GPL-3.0-or-later WITH AdditionRef-HaulPilot-plugin-exception
# Copyright (c) 2026 HaulPilot Authors
# See LICENSE and LICENSE-PLUGIN-EXCEPTION in the repository root.

# HaulPilot - lane assist for Euro Truck Simulator 2 and American Truck Simulator.
#
# Build rules for the whole repository:
#   - C++23 with MSVC, x64 only (both games are 64-bit only).
#   - No third-party libraries. Allowed are the C++ standard library, the
#     system APIs of the Windows SDK and the official SCS SDK headers. Nothing
#     is pulled in through find_package() or FetchContent.
#   - Static C runtime, so neither the app nor the game plugin needs the
#     Visual C++ Redistributable on the user's machine.
#
# Subprojects are added below as they are written, bottom-up.

cmake_minimum_required(VERSION 3.25)

project(HaulPilot
    VERSION 0.1.0
    DESCRIPTION "Lane assist for Euro Truck Simulator 2 and American Truck Simulator"
    LANGUAGES CXX)

if(NOT MSVC)
    message(FATAL_ERROR "HaulPilot is built with MSVC (Visual Studio 2022 or newer).")
endif()

if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "HaulPilot targets x64 only, like ETS2 and ATS.")
endif()

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# /MT in Release, /MTd in Debug.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# One output folder for executables and DLLs, so tests and the app run in place.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")

option(HAULPILOT_BUILD_TESTS "Build the unit tests" ON)
option(HAULPILOT_WARNINGS_AS_ERRORS "Treat compiler warnings as errors" ON)

# Compiler settings shared by every HaulPilot target. Each target links
# haulpilot::options instead of setting its own flags.
add_library(haulpilot_options INTERFACE)
add_library(haulpilot::options ALIAS haulpilot_options)

target_compile_options(haulpilot_options INTERFACE
    /W4
    /permissive-
    /utf-8
    /Zc:__cplusplus
    /Zc:preprocessor
    $<$<BOOL:${HAULPILOT_WARNINGS_AS_ERRORS}>:/WX>)

target_compile_definitions(haulpilot_options INTERFACE
    UNICODE
    _UNICODE
    NOMINMAX
    WIN32_LEAN_AND_MEAN
    _WIN32_WINNT=0x0A00   # Windows 10 or newer
    HAULPILOT_VERSION_MAJOR=${PROJECT_VERSION_MAJOR}
    HAULPILOT_VERSION_MINOR=${PROJECT_VERSION_MINOR}
    HAULPILOT_VERSION_PATCH=${PROJECT_VERSION_PATCH}
    HAULPILOT_VERSION_STRING="${PROJECT_VERSION}")

if(HAULPILOT_BUILD_TESTS)
    enable_testing()
endif()