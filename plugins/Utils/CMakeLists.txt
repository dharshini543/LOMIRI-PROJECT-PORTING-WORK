cmake_minimum_required(VERSION 3.24)
project(Utils-qml LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Qml Quick DBus Network Gui Sql Concurrent REQUIRED)

find_package(PkgConfig REQUIRED)
pkg_check_modules(DEVICEINFO REQUIRED deviceinfo)

set(APPLICATION_API_INCLUDEDIR "/usr/include")
set(LAUNCHER_API_INCLUDEDIR "/usr/include")

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)

include_directories(
    SYSTEM
    ${Qt6Gui_PRIVATE_INCLUDE_DIRS}
    ${Qt6Quick_PRIVATE_INCLUDE_DIRS}
    ${DEVICEINFO_INCLUDE_DIRS}
)

file(GLOB UTILS_FILES "*.h" "*.cpp")

qt_add_qml_module(Utils-qml
    URI Utils
    VERSION 0.1
    SOURCES
    ${UTILS_FILES}
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/ApplicationManagerInterface.h
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/ApplicationInfoInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/AppDrawerModelInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherItemInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherModelInterface.h
)

target_link_libraries(Utils-qml
    PRIVATE
    Qt6::Qml Qt6::Quick Qt6::DBus Qt6::Network Qt6::Gui Qt6::Sql Qt6::Concurrent
    ${DEVICEINFO_LDFLAGS}
)

# Because this is an internal support library, we want
# to expose all symbols in it. Consider changing this
# either to a static library or just using the
# files directly in targets.
set_target_properties(Utils-qml PROPERTIES COMPILE_FLAGS -fvisibility=default)
