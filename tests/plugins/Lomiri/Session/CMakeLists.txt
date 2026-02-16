cmake_minimum_required(VERSION 3.16)
project(SessionBroadcastTests)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Core DBus Test Qml REQUIRED)
include(FindPkgConfig)
pkg_check_modules(GLIB REQUIRED glib-2.0)

find_program(QDBUSXML2CPP_EXECUTABLE qdbusxml2cpp REQUIRED)

link_libraries(
    ${GLIB_LIBRARIES}
)

macro(make_dbus_class NAME INTERFACE)
    if(${CMAKE_CURRENT_SOURCE_DIR}/interfaces.xml IS_NEWER_THAN ${CMAKE_CURRENT_BINARY_DIR}/${NAME}Adaptor.h)
        execute_process(COMMAND ${QDBUSXML2CPP_EXECUTABLE} -c ${NAME}Adaptor -a ${CMAKE_CURRENT_BINARY_DIR}/${NAME}Adaptor ${CMAKE_CURRENT_SOURCE_DIR}/interfaces.xml ${INTERFACE}
            OUTPUT_QUIET ERROR_QUIET)
    endif()
endmacro(make_dbus_class)

make_dbus_class(LightDMSession org.freedesktop.DisplayManager.Session)
make_dbus_class(LogindManager org.freedesktop.login1.Manager)
make_dbus_class(LogindSession org.freedesktop.login1.Session)

include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../plugins/Lomiri/Session
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../src/liblomiri-private
    ${liblomiri-private_SOURCE_DIR}
    ${GLIB_INCLUDE_DIRS}
)

add_definitions(-DSM_BUSNAME=sessionBus)
add_definitions(-DSRCDIR="${CMAKE_CURRENT_SOURCE_DIR}")
add_definitions(-DSESSION_TESTING)

### LightDMServer
add_executable(LightDMServer
    ${CMAKE_CURRENT_BINARY_DIR}/LightDMSessionAdaptor.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/LogindManagerAdaptor.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/LogindSessionAdaptor.cpp
    LightDMServer.cpp
    LogindServer.cpp
    server.cpp
)
target_link_libraries(LightDMServer Qt6::Core Qt6::DBus)
install(TARGETS LightDMServer
    DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Lomiri/Session"
    )

### Preload
add_library(SessionBackendTestPreload SHARED
    preload.c
)
target_link_libraries(SessionBackendTestPreload
    ${GLIB_LIBRARIES}
    )
install(TARGETS SessionBackendTestPreload
    DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Lomiri/Session"
    )

### SessionBackendTest
add_executable(sessionbackendtestExec
    sessionbackendtest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../plugins/Lomiri/Session/dbuslomirisessionservice.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../src/liblomiri-private/lomiridbusobject.cpp
)
target_link_libraries(sessionbackendtestExec
    lomiri-private
    ${GLIB_LIBRARIES}
    )
target_link_libraries(sessionbackendtestExec Qt6::Test Qt6::Core Qt6::Qml Qt6::DBus)
install(TARGETS sessionbackendtestExec
    DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Lomiri/Session"
)

# Use add_lomiri_unittest when building from root project
# Otherwise fallback to add_test for standalone builds
if(COMMAND add_lomiri_unittest)
    add_lomiri_unittest(SessionBackend dbus-test-runner
        ENVIRONMENT LD_PRELOAD=$<TARGET_FILE:SessionBackendTestPreload>
                    XDG_SESSION_PATH=/session XDG_SESSION_ID=2
        ARG_PREFIX "--parameter"
        ARGS
            --task $<TARGET_FILE:LightDMServer>
            --task-name server
            --ignore-return
            --task $<TARGET_FILE:sessionbackendtestExec>
            --task-name client
            --wait-for org.freedesktop.DisplayManager
    )
else()
    set(TEST_ENV "QML2_IMPORT_PATH=${CMAKE_CURRENT_SOURCE_DIR}/../../../../qml:${CMAKE_CURRENT_SOURCE_DIR}/../../../../plugins")
    set(TEST_ENV "${TEST_ENV} LOMIRI_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/../../../../")
    set(TEST_ENV "${TEST_ENV} LD_PRELOAD=$<TARGET_FILE:SessionBackendTestPreload>")
    set(TEST_ENV "${TEST_ENV} XDG_SESSION_PATH=/session XDG_SESSION_ID=2")

    enable_testing()
    add_test(NAME SessionBackendTest
        COMMAND bash -c "${TEST_ENV} $<TARGET_FILE:LightDMServer> & sleep 1 && ${TEST_ENV} $<TARGET_FILE:sessionbackendtestExec> ; pkill -f LightDMServer"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
endif()
