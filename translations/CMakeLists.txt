# PianoBooster Translations - Modern CMake with Qt6 support
#
# This file handles translation file processing for PianoBooster.
# It uses modern CMake with Qt6 compatibility while maintaining Qt5 support.

set(TRANSLATIONS_FILES
    pianobooster_ca.ts
    pianobooster_cs.ts
    pianobooster_da.ts
    pianobooster_de.ts
    pianobooster_es.ts
    pianobooster_fr.ts
    pianobooster_gl.ts
    pianobooster_is.ts
    pianobooster_it.ts
    pianobooster_ja.ts
    pianobooster_nb.ts
    pianobooster_nl.ts
    pianobooster_pl.ts
    pianobooster_ru.ts
    pianobooster_sk.ts
    pianobooster_sv.ts
    pianobooster_uk.ts
    pianobooster_zh.ts
)

# Optionally include all translation files if requested
if(INSTALL_ALL_LANGS)
    file(GLOB TRANSLATIONS_FILES "*.ts")
endif()

# Remove the blank template from the list
get_filename_component(BLANK_TS ${CMAKE_CURRENT_SOURCE_DIR}/pianobooster_blank.ts ABSOLUTE)
list(REMOVE_ITEM TRANSLATIONS_FILES ${BLANK_TS})

# Find Qt translation tools
# Use the same Qt version as the main project
find_package(${QT_PACKAGE_NAME} REQUIRED COMPONENTS LinguistTools)

# Create translation files with Qt6/Qt5 compatibility
if(COMMAND Qt6_create_translation)
    Qt6_create_translation(QM_FILES ${CMAKE_CURRENT_SOURCE_DIR} ${TRANSLATIONS_FILES})
elseif(COMMAND Qt5_create_translation)
    Qt5_create_translation(QM_FILES ${CMAKE_CURRENT_SOURCE_DIR} ${TRANSLATIONS_FILES})
elseif(COMMAND QT_ADD_TRANSLATION)
    QT_ADD_TRANSLATION(QM_FILES ${TRANSLATIONS_FILES})
else()
    message(WARNING "Qt translation tools not found - translations will not be built")
endif()

# Create custom target for translations
if(DEFINED QM_FILES)
    add_custom_target(translations ALL DEPENDS ${QM_FILES})
endif()

# Installation
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    if(DEFINED QM_FILES)
        install(FILES ${QM_FILES} DESTINATION ${DATA_DIR}/translations)
    endif()
    install(FILES langs.json DESTINATION ${DATA_DIR}/translations)
elseif(WIN32)
    if(DEFINED QM_FILES)
        install(FILES ${QM_FILES} DESTINATION .)
    endif()
    install(FILES langs.json DESTINATION .)
endif()

# Copy language configuration to build directory
configure_file(langs.json ${CMAKE_CURRENT_BINARY_DIR}/langs.json COPYONLY)
