cmake_minimum_required(VERSION 3.17)
project(WizardUtils-qml LANGUAGES C CXX)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 REQUIRED COMPONENTS Core DBus Qml Quick)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${Qt6Gui_PRIVATE_INCLUDE_DIRS}
)

set(QMLPLUGIN_SRC
    qsortfilterproxymodelqml.h
    qsortfilterproxymodelqml.cpp
    # plugin.cpp
    ../System.cpp
)

add_library(WizardUtils-qml SHARED
    ${QMLPLUGIN_SRC}
)

# Because this is an internal support library, we want
# to expose all symbols in it. Consider changing this
# either to a static library or just using the
# files directly in targets.
set_target_properties(WizardUtils-qml PROPERTIES COMPILE_FLAGS -fvisibility=default)

target_link_libraries(WizardUtils-qml Qt6::DBus Qt6::Qml Qt6::Quick)

set(PLUG_DIR ${PLUGIN_PRIVATE_MODULE_DIR}/Lomiri/SystemSettings/Wizard/Utils)
install(FILES qmldir DESTINATION ${PLUG_DIR})
install(TARGETS WizardUtils-qml DESTINATION ${PLUG_DIR})
