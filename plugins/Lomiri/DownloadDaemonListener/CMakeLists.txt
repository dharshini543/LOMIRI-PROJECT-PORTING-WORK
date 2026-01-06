cmake_minimum_required(VERSION 3.17)
project(DownloadDaemonListener LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)

find_package(PkgConfig REQUIRED)
pkg_check_modules(DDLADAPTER REQUIRED ldm-common-qt6)
find_package(Qt6Qml REQUIRED)
find_package(Qt6Quick REQUIRED)
find_package(Qt6DBus REQUIRED)
find_package(Qt6Core REQUIRED)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)

qt_add_qml_module(DownloadDaemonListener
    URI Lomiri.DownloadDaemonListener
    VERSION 0.1
    SOURCES
    plugin.h
    plugin.cpp
    DownloadTracker.h
    DownloadTracker.cpp
    interface/downloadtrackeradaptor.h
    interface/downloadtrackeradaptor.cpp)

add_definitions(${DDLADAPTER_CFLAGS} ${DDLADAPTER_CFLAGS_OTHER})

target_link_libraries(DownloadDaemonListener
    PRIVATE
    Qt6::Qml Qt6::Quick Qt6::DBus Qt6::Core
    ${DDLADAPTER_LIBRARIES}
)
