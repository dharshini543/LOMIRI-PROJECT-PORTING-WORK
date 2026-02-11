cmake_minimum_required(VERSION 3.16)
project(LauncherModelASTest)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Core DBus Gui Qml Test REQUIRED)
find_package(PkgConfig REQUIRED)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/AccountsService
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/Greeter/Lomiri/Launcher
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../include
    )

add_definitions(-DSM_BUSNAME=sessionBus)
add_definitions(-DTEST_DIR="plugins/Greeter/Lomiri/Launcher")

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../include/paths.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/paths.h
    @ONLY
)
set(APPLICATION_API_INCLUDEDIR "/usr/include")
set(LAUNCHER_API_INCLUDEDIR "/usr/include")

### LauncherModelASTest
add_executable(launchermodelastestExec
    launchermodelastest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/AccountsService/AccountsServiceDBusAdaptor.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/Greeter/Lomiri/Launcher/launchermodelas.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/Greeter/Lomiri/Launcher/launcheritem.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/Greeter/Lomiri/Launcher/quicklistmodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../plugins/Greeter/Lomiri/Launcher/quicklistentry.cpp
    ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherItemInterface.h
        ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/LauncherModelInterface.h
        ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/launcher/QuickListModelInterface.h
        ${APPLICATION_API_INCLUDEDIR}/lomiri/shell/application/ApplicationManagerInterface.h
        ${LAUNCHER_API_INCLUDEDIR}/lomiri/shell/application/ApplicationInfoInterface.h
    )
target_link_libraries(launchermodelastestExec
    Qt6::Test Qt6::Core Qt6::DBus Qt6::Gui Qt6::Qml
    )

enable_testing()

if(COMMAND add_lomiri_unittest)
    add_lomiri_unittest(LauncherModelAS dbus-test-runner
        ARG_PREFIX "--parameter"
        ARGS
            --task $<TARGET_FILE:mock-server>
            --ignore-return
            --task $<TARGET_FILE:launchermodelastestExec>
            --wait-for org.freedesktop.Accounts
    )
else()
    set(TEST_PROJECT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../")
    set(TEST_ENV "QML2_IMPORT_PATH=${TEST_PROJECT_ROOT}qml:${TEST_PROJECT_ROOT}plugins")
    set(TEST_ENV "${TEST_ENV} LOMIRI_SOURCE_DIR=${TEST_PROJECT_ROOT}")

    add_test(NAME LauncherModelASTest
        COMMAND bash -c "${TEST_ENV} $<TARGET_FILE_DIR:launchermodelastestExec>/../../../../../../tests/plugins/AccountsService/build/mock-server & sleep 1 && ${TEST_ENV} $<TARGET_FILE:launchermodelastestExec> ; pkill -f mock-server"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
endif()
