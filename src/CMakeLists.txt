# PianoBooster Main Source CMakeLists.txt
#
# This file defines the main PianoBooster executable and its dependencies.
# Modern CMake practices are used throughout:
# - Use of target_* commands instead of global settings
# - Private/Public visibility properly specified
# - GENERATOR_EXPRESSION for platform-specific logic

# ============================================================================
# Source Files
# ============================================================================

set(PB_CORE_SOURCES
    MidiFile.cpp
    MidiTrack.cpp
    Song.cpp
    Conductor.cpp
    Util.cpp
    Chord.cpp
    Tempo.cpp
    MidiDevice.cpp
    MidiDeviceRt.cpp
    StavePosition.cpp
    Score.cpp
    Cfg.cpp
    Piano.cpp
    Draw.cpp
    Scroll.cpp
    Notation.cpp
    TrackList.cpp
    Rating.cpp
    Bar.cpp
    Settings.cpp
    Merge.cpp
)

set(PB_UI_SOURCES
    QtMain.cpp
    QtWindow.cpp
    GuiTopBar.cpp
    GuiSidePanel.cpp
    GuiMidiSetupDialog.cpp
    GuiKeyboardSetupDialog.cpp
    GuiPreferencesDialog.cpp
    GuiSongDetailsDialog.cpp
    GuiLoopingPopup.cpp
    GlView.cpp
)

set(PB_SOURCES
    ${PB_CORE_SOURCES}
    ${PB_UI_SOURCES}
)

# Add FluidSynth source if enabled
if(HAVE_FLUIDSYNTH)
    list(APPEND PB_SOURCES MidiDeviceFluidSynth.cpp)
endif()

# Add bundled RtMidi if needed
if(USE_BUNDLED_RTMIDI)
    list(APPEND PB_SOURCES 3rdparty/rtmidi/RtMidi.cpp)
endif()

set(PB_HEADERS
    MidiFile.h
    MidiTrack.h
    Song.h
    Conductor.h
    Rating.h
    Util.h
    Chord.h
    Tempo.h
    MidiDevice.h
)

set(PB_MOC_HEADERS
    QtWindow.h
    GlView.h
    GuiTopBar.h
    GuiSidePanel.h
    GuiMidiSetupDialog.h
    GuiKeyboardSetupDialog.h
    GuiPreferencesDialog.h
    GuiSongDetailsDialog.h
    GuiLoopingPopup.h
)

set(PB_UI_FILES
    GuiTopBar.ui
    GuiSidePanel.ui
    GuiMidiSetupDialog.ui
    GuiKeyboardSetupDialog.ui
    GuiPreferencesDialog.ui
    GuiSongDetailsDialog.ui
    GuiLoopingPopup.ui
)

set(PB_RESOURCES
    application.qrc
)

# Platform-specific resources
if(APPLE)
    list(APPEND PB_SOURCES ../icons/MacOS/pianobooster.icns)
endif()
if(WIN32)
    list(APPEND PB_SOURCES pianobooster.rc)
    list(APPEND PB_SOURCES images/pianobooster.ico)
endif()

# ============================================================================
# Create executable
# ============================================================================

if(APPLE)
    set(MACOSX_BUNDLE_ICON_FILE pianobooster.icns)
    
    add_executable(pianobooster MACOSX_BUNDLE
        ${PB_SOURCES}
        ${PB_MOC_HEADERS}
        ${PB_UI_FILES}
        ${PB_RESOURCES}
    )
    
    set_target_properties(pianobooster PROPERTIES
        MACOSX_BUNDLE TRUE
        MACOSX_BUNDLE_ICON_FILE pianobooster.icns
        MACOSX_BUNDLE_INFO_STRING "PianoBooster - A MIDI file player and music teacher"
        MACOSX_BUNDLE_GUI_IDENTIFIER "org.pianobooster.app"
    )
else()
    add_executable(pianobooster
        ${PB_SOURCES}
        ${PB_MOC_HEADERS}
        ${PB_UI_FILES}
        ${PB_RESOURCES}
    )
endif()

if(WIN32)
    set_target_properties(pianobooster PROPERTIES
        WIN32_EXECUTABLE ON
        LINK_FLAGS "-mwindows"
    )
endif()

# ============================================================================
# Compile Definitions (Modern approach)
# ============================================================================

target_compile_definitions(pianobooster PRIVATE
    PREFIX="${CMAKE_INSTALL_PREFIX}"
    DATA_DIR="${DATA_DIR}"
)

# Platform-specific definitions
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    target_compile_definitions(pianobooster PRIVATE __LINUX_ALSA__)
elseif(WIN32)
    target_compile_definitions(pianobooster PRIVATE __WINDOWS_MM__ _WIN32)
elseif(APPLE)
    target_compile_definitions(pianobooster PRIVATE __MACOSX_CORE__ GL_SILENCE_DEPRECATION)
endif()

# Optional feature definitions
if(HAVE_FLUIDSYNTH)
    target_compile_definitions(pianobooster PRIVATE WITH_INTERNAL_FLUIDSYNTH)
endif()

if(NOT USE_FTGL)
    target_compile_definitions(pianobooster PRIVATE NO_USE_FTGL)
endif()

if(USE_JACK AND HAVE_JACK)
    target_compile_definitions(pianobooster PRIVATE __UNIX_JACK__)
endif()

if(USE_BUNDLED_RTMIDI)
    target_compile_definitions(pianobooster PRIVATE USE_BUNDLED_RTMIDI)
endif()

# ============================================================================
# Include Directories
# ============================================================================

target_include_directories(pianobooster PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${OPENGL_INCLUDE_DIR}
)

if(USE_FTGL AND FTGL_FOUND)
    target_include_directories(pianobooster PRIVATE ${FTGL_INCLUDE_DIRS})
endif()

if(HAVE_FLUIDSYNTH)
    target_include_directories(pianobooster PRIVATE ${FLUIDSYNTH_INCLUDE_DIRS})
endif()

if(USE_JACK AND HAVE_JACK)
    target_include_directories(pianobooster PRIVATE ${JACK_INCLUDE_DIRS})
endif()

if(USE_BUNDLED_RTMIDI)
    target_include_directories(pianobooster PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty)
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    target_include_directories(pianobooster PRIVATE ${ALSA_INCLUDE_DIRS})
endif()

if(NOT WIN32)
    if(HAVE_RTMIDI)
        target_include_directories(pianobooster PRIVATE ${RTMIDI_INCLUDE_DIRS})
    endif()
endif()

# ============================================================================
# Link Libraries (Modern approach)
# ============================================================================

# Qt libraries
target_link_libraries(pianobooster PRIVATE
    ${QT_PACKAGE_NAME}::Core
    ${QT_PACKAGE_NAME}::Gui
    ${QT_PACKAGE_NAME}::Widgets
    ${QT_PACKAGE_NAME}::Xml
    ${QT_PACKAGE_NAME}::OpenGL
)

if(${QT_PACKAGE_NAME} STREQUAL "Qt6")
    target_link_libraries(pianobooster PRIVATE
        ${QT_PACKAGE_NAME}::OpenGLWidgets
        ${QT_PACKAGE_NAME}::Core5Compat
    )
endif()

# OpenGL
target_link_libraries(pianobooster PRIVATE ${OPENGL_LIBRARIES})

# Platform-specific libraries
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    target_link_libraries(pianobooster PRIVATE
        ${ALSA_LIBRARIES}
        pthread
        GL
    )
elseif(WIN32)
    target_link_libraries(pianobooster PRIVATE
        winmm
        opengl32
    )
elseif(APPLE)
    target_link_libraries(pianobooster PRIVATE
        ${COREAUDIO}
        ${COREMIDI}
        ${COREFOUNDATION}
    )
endif()

# Optional libraries
if(USE_FTGL AND FTGL_FOUND)
    target_link_libraries(pianobooster PRIVATE ${FTGL_LIBRARIES})
    target_link_directories(pianobooster PRIVATE ${FTGL_LIBRARY_DIRS})
endif()

if(HAVE_FLUIDSYNTH)
    target_link_libraries(pianobooster PRIVATE ${FLUIDSYNTH_LIBRARIES})
    target_link_directories(pianobooster PRIVATE ${FLUIDSYNTH_LIBRARY_DIRS})
endif()

if(USE_JACK AND HAVE_JACK)
    target_link_libraries(pianobooster PRIVATE ${JACK_LIBRARIES})
    target_link_directories(pianobooster PRIVATE ${JACK_LIBRARY_DIRS})
endif()

if(NOT USE_BUNDLED_RTMIDI AND HAVE_RTMIDI)
    target_link_libraries(pianobooster PRIVATE ${RTMIDI_LIBRARIES})
    target_link_directories(pianobooster PRIVATE ${RTMIDI_LIBRARY_DIRS})
endif()

# ============================================================================
# Compiler Warnings
# ============================================================================

if(ENABLE_WARNINGS)
    set(CLANG_WARNINGS
        -Wall
        -Wextra
        -Wshadow=local
        -Wnon-virtual-dtor
        -Wold-style-cast
        -Wcast-align
        -Wunused
        -Woverloaded-virtual
        -Wconversion
        -Wsign-conversion
        -Wnull-dereference
        -Wdouble-promotion
        -Wformat=2
        -Wno-pedantic
        -Wno-missing-field-initializers
        -Wno-useless-cast
        -Wno-unused-const-variable
        -Wno-unknown-warning-option
    )
    
    set(GCC_WARNINGS
        ${CLANG_WARNINGS}
        -Wmisleading-indentation
        -Wduplicated-cond
        -Wduplicated-branches
        -Wlogical-op
    )
    
    if(CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")
        target_compile_options(pianobooster PRIVATE ${CLANG_WARNINGS})
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(pianobooster PRIVATE ${GCC_WARNINGS})
    endif()
endif()

if(TREAT_WARNINGS_AS_ERRORS)
    if(CMAKE_CXX_COMPILER_ID MATCHES ".*Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(pianobooster PRIVATE -Werror)
    endif()
endif()

# ============================================================================
# Installation
# ============================================================================

# Application binary
if(APPLE)
    install(TARGETS pianobooster BUNDLE DESTINATION .)
else()
    install(TARGETS pianobooster RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

# Desktop entry (Linux only)
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    install(FILES ../pianobooster.desktop DESTINATION share/applications)
endif()

# Icons (Linux only)
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    install(FILES ../icons/hicolor/32x32/pianobooster.png DESTINATION share/icons/hicolor/32x32/apps)
    install(FILES ../icons/hicolor/48x48/pianobooster.png DESTINATION share/icons/hicolor/48x48/apps)
    install(FILES ../icons/hicolor/64x64/pianobooster.png DESTINATION share/icons/hicolor/64x64/apps)
endif()

# Documentation
if(NOT NO_DOCS)
    install(FILES ../README.md DESTINATION share/doc/pianobooster/)
    install(FILES ../doc/faq.md DESTINATION share/doc/pianobooster/)
endif()

# License
if(NOT NO_LICENSE)
    install(FILES ../license.txt DESTINATION share/licenses/pianobooster/)
endif()

# Changelog
if(NOT NO_CHANGELOG)
    install(FILES ../Changelog.txt DESTINATION share/doc/pianobooster/)
endif()

# Man page
if(WITH_MAN)
    install(FILES ../pianobooster.6 DESTINATION share/man/man6/)
endif()

# Music data
configure_file(../music/BoosterMusicBooks.zip ${CMAKE_CURRENT_BINARY_DIR}/BoosterMusicBooks.zip COPYONLY)

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    install(FILES ../music/BoosterMusicBooks.zip DESTINATION ${DATA_DIR}/music)
elseif(WIN32)
    install(FILES ../music/BoosterMusicBooks.zip DESTINATION .)
endif()

# Fonts
if(NOT USE_SYSTEM_FONT)
    if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
        install(FILES fonts/DejaVuSans.ttf DESTINATION ${DATA_DIR}/fonts/)
    endif()
    configure_file(fonts/DejaVuSans.ttf ${CMAKE_CURRENT_BINARY_DIR}/fonts/DejaVuSans.ttf COPYONLY)
endif()
