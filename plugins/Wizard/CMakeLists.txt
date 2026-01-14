cmake_minimum_required(VERSION 3.17)
project(Wizard-qml LANGUAGES C CXX)

set(CMAKE_AUTOMOC ON)

add_definitions(-DQT_NO_KEYWORDS)

find_package(PkgConfig REQUIRED)
pkg_search_module(GD3 REQUIRED gnome-desktop-3.0)
pkg_check_modules(GEONAMES REQUIRED geonames)

find_package(Qt6DBus REQUIRED)
find_package(Qt6Qml REQUIRED)

include_directories(SYSTEM ${GD3_INCLUDE_DIRS} ${GLIB_INCLUDE_DIRS} ${GEONAMES_INCLUDE_DIRS})
include_directories(${CMAKE_SOURCE_DIR}/../../include)

qt_add_qml_module(Wizard-qml
    URI Wizard
    VERSION 0.1
    SOURCES
    PageList.h
    PageList.cpp
    System.h
    System.cpp
    LocalePlugin.h
    LocalePlugin.cpp
    timezonemodel.h
    timezonemodel.cpp
    Status.h
    Status.cpp
    keyboardLayoutsModel.h
    keyboardLayoutsModel.cpp
    Changelog.h
    Changelog.cpp)

target_link_libraries(Wizard-qml
    PRIVATE
    Qt6::DBus Qt6::Qml
    ${GD3_LDFLAGS} ${GLIB_LDFLAGS} ${GEONAMES_LDFLAGS}
)

if (NOT "${DISPLAYED_DISTRO_NAME}" STREQUAL "")
    target_compile_definitions(Wizard-qml PRIVATE
        -DLOMIRI_DISPLAYED_DISTRO_NAME="${DISPLAYED_DISTRO_NAME}")
endif ()


set(POLKIT_LIB_DIR "${CMAKE_INSTALL_LOCALSTATEDIR}/lib/polkit-1")
install(FILES 50-com.lomiri.wizard.pkla DESTINATION ${POLKIT_LIB_DIR}/localauthority/10-vendor.d)
