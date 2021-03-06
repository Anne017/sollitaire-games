project(solitaire-games C CXX)
cmake_minimum_required(VERSION 3.0.0)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

find_package(Qt5Core)
find_package(Qt5Qml)
find_package(Qt5Quick)

# Automatically create moc files
set(CMAKE_AUTOMOC ON)

# Components PATH
execute_process(
    COMMAND dpkg-architecture -qDEB_HOST_MULTIARCH
    OUTPUT_VARIABLE ARCH_TRIPLET
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

set(PROJECT_NAME "solitaire-games")
set(FULL_PROJECT_NAME "solitaire-games.mateo-salta")
set(CMAKE_INSTALL_PREFIX /)
set(DATA_DIR /)
set(DESKTOP_FILE_NAME ${PROJECT_NAME}.desktop)
set(MAIN_QML ${PROJECT_NAME}.qml)

# This command figures out the target architecture for use in the manifest file
execute_process(
    COMMAND dpkg-architecture -qDEB_HOST_ARCH
    OUTPUT_VARIABLE CLICK_ARCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

configure_file(manifest.json.in ${CMAKE_CURRENT_BINARY_DIR}/manifest.json)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/manifest.json DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES ${PROJECT_NAME}.apparmor DESTINATION ${DATA_DIR})
install(FILES ${PROJECT_NAME}.qml DESTINATION ${DATA_DIR})
install(FILES ${PROJECT_NAME}.png DESTINATION ${DATA_DIR})
install(DIRECTORY components DESTINATION ${DATA_DIR})
install(DIRECTORY data DESTINATION ${DATA_DIR})
install(DIRECTORY games DESTINATION ${DATA_DIR})
install(DIRECTORY layout DESTINATION ${DATA_DIR})
install(DIRECTORY svg DESTINATION ${DATA_DIR})

# Translations
file(GLOB_RECURSE I18N_SRC_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/po components/*.qml "solitaire-games.qml" games/*.qml layout/*.qml)
list(APPEND I18N_SRC_FILES ${DESKTOP_FILE_NAME}.in.h)
FILE(GLOB XML_SRC_FILES data/games.xml)
FILE(GLOB PREVIOUS_POT po/${FULL_PROJECT_NAME}.pot)

find_program(INTLTOOL_MERGE intltool-merge)
if(NOT INTLTOOL_MERGE)
    message(FATAL_ERROR "Could not find intltool-merge, please install the intltool package")
endif()
find_program(INTLTOOL_EXTRACT intltool-extract)
if(NOT INTLTOOL_EXTRACT)
    message(FATAL_ERROR "Could not find intltool-extract, please install the intltool package")
endif()
find_program(ITSTOOL itstool)
if(NOT ITSTOOL)
    message(FATAL_ERROR "Could not find itstool, please install the itstool package")
endif()

add_custom_target(${DESKTOP_FILE_NAME} ALL
    COMMENT "Merging translations into ${DESKTOP_FILE_NAME}..."
    COMMAND LC_ALL=C ${INTLTOOL_MERGE} -d -u ${CMAKE_SOURCE_DIR}/po ${CMAKE_SOURCE_DIR}/${DESKTOP_FILE_NAME}.in ${DESKTOP_FILE_NAME}
    COMMAND sed -i 's/${PROJECT_NAME}-//g' ${CMAKE_CURRENT_BINARY_DIR}/${DESKTOP_FILE_NAME}
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${DESKTOP_FILE_NAME} DESTINATION ${DATA_DIR})

add_subdirectory(po)

# # Make source files visible in qtcreator
# # We don't need to add plugin sources here as they get exposed
# # via the library target.
# file(GLOB_RECURSE PROJECT_SRC_FILES
#     RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
#     components/*.qml
#     components/*.js
#     *.json
#     *.apparmor
#     *.desktop
# )

# add_custom_target(${PROJECT_NAME}_FILES ALL SOURCES ${PROJECT_SRC_FILES})
