cmake_minimum_required(VERSION 3.20)

# Set C++ standard before project() declaration
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(pianobooster 
    VERSION 0.1.0
    DESCRIPTION "A MIDI file player/game that displays musical notes and teaches you how to play the piano"
    HOMEPAGE_URL "https://www.pianobooster.org/"
    LANGUAGES CXX
)

# Modern CMake policies
if(POLICY CMP0020)
    cmake_policy(SET CMP0020 NEW)
endif()
if(POLICY CMP0043)
    cmake_policy(SET CMP0043 NEW)
endif()
if(POLICY CMP0074)
    cmake_policy(SET CMP0074 NEW)
endif()
if(POLICY CMP0077)
    cmake_policy(SET CMP0077 NEW)
endif()

# Project configuration
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Use colored output for build tools
if(NOT WIN32 AND NOT APPLE)
    add_compile_options(-fdiagnostics-color=always)
endif()

# Add position-independent code
add_compile_options(-fPIC)

# Enable basic warnings
add_compile_options(-Wall -Wextra)

# ============================================================================
# Build Options
# ============================================================================
option(WITH_INTERNAL_FLUIDSYNTH "Build with internal FluidSynth sound generator" ON)
option(USE_FTGL "Build with FTGL for notes localization" ON)
option(USE_SYSTEM_FONT "Build with system font instead of bundled" OFF)
option(USE_JACK "Build with JACK (required for BSD Unix)" OFF)
option(ENABLE_WARNINGS "Enable additional compiler warnings" OFF)
option(TREAT_WARNINGS_AS_ERRORS "Treat compiler warnings as errors" OFF)

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    option(USE_BUNDLED_RTMIDI "Use bundled rtmidi (for older distributions)" OFF)
else()
    option(USE_BUNDLED_RTMIDI "Use bundled rtmidi" ON)
endif()

# Installation directories
if(NOT CMAKE_INSTALL_BINDIR)
    set(CMAKE_INSTALL_BINDIR "bin")
endif()
if(NOT DATA_DIR)
    set(DATA_DIR "share/games/pianobooster")
endif()

message(STATUS "PianoBooster ${PROJECT_VERSION}")
message(STATUS "  CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  CMAKE_INSTALL_BINDIR: ${CMAKE_INSTALL_BINDIR}")
message(STATUS "  DATA_DIR: ${DATA_DIR}")

# ============================================================================
# Dependencies
# ============================================================================

# Qt6
set(QT_PACKAGE_NAME "Qt6" CACHE STRING "Qt package name (Qt5 or Qt6)")
set(QT_COMPONENTS Core Gui Widgets LinguistTools OpenGL Xml)

if(${QT_PACKAGE_NAME} STREQUAL "Qt6")
    list(APPEND QT_COMPONENTS OpenGLWidgets Core5Compat)
endif()

find_package(${QT_PACKAGE_NAME} REQUIRED COMPONENTS ${QT_COMPONENTS})

# OpenGL
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

# Platform-specific setup and dependencies
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(ALSA REQUIRED alsa)
    
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
    # macOS frameworks
    find_library(COREAUDIO CoreAudio REQUIRED)
    find_library(COREMIDI CoreMidi REQUIRED)
    find_library(COREFOUNDATION CoreFoundation REQUIRED)
    
elseif(WIN32)
    # Windows multimedia (linked via target_link_libraries)
endif()

# FTGL (optional, for text rendering)
if(USE_FTGL)
    if(NOT WIN32)
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(FTGL REQUIRED ftgl)
    else()
        find_package(ftgl REQUIRED)
    endif()
else()
    set(FTGL_FOUND FALSE)
endif()

# FluidSynth (optional, for internal sound synthesis)
if(WITH_INTERNAL_FLUIDSYNTH)
    if(NOT WIN32)
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(FLUIDSYNTH REQUIRED fluidsynth)
    else()
        find_package(fluidsynth REQUIRED)
    endif()
    set(HAVE_FLUIDSYNTH TRUE)
else()
    set(HAVE_FLUIDSYNTH FALSE)
endif()

# JACK (optional, for BSD Unix)
if(USE_JACK)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(JACK REQUIRED jack)
    set(HAVE_JACK TRUE)
else()
    set(HAVE_JACK FALSE)
endif()

# RtMidi (for MIDI input)
if(USE_BUNDLED_RTMIDI)
    message(STATUS "Using bundled RtMidi")
    set(HAVE_RTMIDI TRUE)
else()
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(RTMIDI REQUIRED rtmidi)
    set(HAVE_RTMIDI TRUE)
endif()

# ============================================================================
# Subdirectories
# ============================================================================

add_subdirectory(src)
add_subdirectory(translations)
