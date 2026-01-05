cmake_minimum_required(VERSION 3.17)
project(LomiriLauncher-qml LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)

find_package(PkgConfig REQUIRED)
pkg_check_modules(GSETTINGS_QT REQUIRED gsettings-qt)
pkg_check_modules(UAL   REQUIRED lomiri-app-launch-0)

find_package(Qt6DBus REQUIRED)
find_package(Qt6Qml REQUIRED)
find_package(Qt6Gui REQUIRED)
find_package(Qt6Concurrent REQUIRED)

set(APPLICATION_API_INCLUDEDIR "/usr/include")
set(LAUNCHER_API_INCLUDEDIR    "/usr/include")

add_definitions(-DSM_BUSNAME=systemBus)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../../../src/liblomiri-private liblomiri-private)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../../AccountsService
    ${liblomiri-private_SOURCE_DIR}
)

include_directories(
    SYSTEM
    ${GSETTINGS_QT_INCLUDE_DIRS}
    ${GLIB_INCLUDE_DIRS}
    ${UAL_INCLUDE_DIRS}
    )

qt_add_qml_module(LomiriLauncher-qml
    URI Lomiri.Launcher
    VERSION 0.1
    SOURCES
    plugin.h
    plugin.cpp
    launchermodel.h
    launchermodel.cpp
    launcheritem.h
    launcheritem.cpp
    quicklistmodel.h
    quicklistmodel.cpp
    quicklistentry.h
    quicklistentry.cpp
    dbusinterface.h
    dbusinterface.cpp
    gsettings.h
    gsettings.cpp
    asadapter.h
    asadapter.cpp
    appdrawermodel.h
    appdrawermodel.cpp
    ualwrapper.h
    ualwrapper.cpp
    xdgwatcher.h
    xdgwatcher.cpp
    ${CMAKE_SOURCE_DIR}/../../AccountsService/AccountsServiceDBusAdaptor.cpp
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/ApplicationManagerInterface.h
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/ApplicationInfoInterface.h
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/MirSurfaceListInterface.h
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/MirSurfaceInterface.h
    ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/Mir.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherItemInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherModelInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/QuickListModelInterface.h
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/AppDrawerModelInterface.h)

target_link_libraries(LomiriLauncher-qml
    PRIVATE
    lomiri-private
    ${GSETTINGS_QT_LDFLAGS}
    ${GLIB_LIBRARIES}
    ${UAL_LIBRARIES}
    Qt6::DBus Qt6::Qml Qt6::Gui Qt6::Concurrent
    )

