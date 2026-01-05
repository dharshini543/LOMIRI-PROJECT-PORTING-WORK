cmake_minimum_required(VERSION 3.17)
project(UInput-qml LANGUAGES CXX)

set(CMAKE_AUTOMOC ON)

find_package(Qt6Qml REQUIRED)

qt_add_qml_module(UInput-qml
    URI UInput
    VERSION 0.1
    SOURCES
    plugin.h
    plugin.cpp
    uinput.h
    uinput.cpp)

target_link_libraries(UInput-qml PRIVATE Qt6::Qml)
