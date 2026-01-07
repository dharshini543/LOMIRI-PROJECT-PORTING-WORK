cmake_minimum_required(VERSION 3.17)
project(IndicatorsQml LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)

add_definitions(-DQT_NO_KEYWORDS)

find_package(PkgConfig REQUIRED)
pkg_check_modules(GLIB REQUIRED glib-2.0)
pkg_check_modules(GIO REQUIRED gio-2.0)
pkg_check_modules(QMENUMODEL REQUIRED qmenumodel)

find_package(Qt6Core REQUIRED)
find_package(Qt6Qml REQUIRED)
find_package(Qt6Quick REQUIRED)
find_package(Qt6DBus REQUIRED)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)

include_directories(
    SYSTEM
    ${GLIB_INCLUDE_DIRS}
)
include_directories(
    SYSTEM
    ${GIO_INCLUDE_DIRS}
    ${QMENUMODEL_INCLUDE_DIRS}
)

qt_add_qml_module(IndicatorsQml
    URI Lomiri.Indicators
    VERSION 0.1
    SOURCES
    actionrootstate.h
    actionrootstate.cpp
    indicator.h
    indicator.cpp
    indicators.h
    indicatorsmanager.h
    indicatorsmanager.cpp
    indicatorsmodel.h
    indicatorsmodel.cpp
    menucontentactivator.h
    menucontentactivator.cpp
    modelactionrootstate.h
    modelactionrootstate.cpp
    modelprinter.h
    modelprinter.cpp
    plugin.h
    plugin.cpp
    rootstateparser.h
    rootstateparser.cpp
    sharedlomirimenumodel.h
    sharedlomirimenumodel.cpp
    lomirimenumodelcache.h
    lomirimenumodelcache.cpp
    lomirimenumodelstack.h
    lomirimenumodelstack.cpp
)

add_definitions(-DLOMIRIINDICATORS_LIBRARY)

target_link_libraries(IndicatorsQml
    PRIVATE
    ${GLIB_LIBRARIES}
    ${GIO_LIBRARIES}
    ${QMENUMODEL_LDFLAGS}
    Qt6::Core Qt6::Qml Qt6::Quick Qt6::DBus
)

