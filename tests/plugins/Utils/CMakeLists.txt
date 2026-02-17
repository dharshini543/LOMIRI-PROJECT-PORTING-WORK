cmake_minimum_required(VERSION 3.16)

project(UtilsTests)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Core Test Qml Quick Sql DBus REQUIRED)

include_directories(
    ${UtilsQmlPlugin_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils
    ${CMAKE_CURRENT_BINARY_DIR}
    )

add_definitions(-DCONFIG_TESTING)

execute_process(COMMAND mktemp -d OUTPUT_VARIABLE TMPDIR OUTPUT_STRIP_TRAILING_WHITESPACE)

# Note: WindowStateStorage test is skipped because it depends on Qt5-only Mir library
# which has Qt5/Qt6 header conflicts. It will work when built from root project
# after Mir library is ported to Qt6.

foreach(util_test
    QLimitProxyModel
    LomiriSortFilterProxyModel
    WindowInputMonitor
)
    set(TEST_SOURCES ${util_test}Test.cpp ModelTest.cpp)
    
    # Add source files for specific tests
    if(util_test STREQUAL "QLimitProxyModel")
        list(APPEND TEST_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils/qlimitproxymodelqml.cpp)
    elseif(util_test STREQUAL "LomiriSortFilterProxyModel")
        list(APPEND TEST_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils/lomirisortfilterproxymodelqml.cpp)
    elseif(util_test STREQUAL "WindowInputMonitor")
        list(APPEND TEST_SOURCES 
            ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils/WindowInputMonitor.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils/Timer.cpp
        )
    endif()
    
    add_executable(${util_test}TestExec ${TEST_SOURCES})
    target_link_libraries(${util_test}TestExec Qt6::Test Qt6::Core Qt6::Qml Qt6::Quick Qt6::Sql)
    install(TARGETS ${util_test}TestExec
        DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Utils"
    )
    
    # Use add_lomiri_unittest when building from root project
    # Otherwise fallback to add_test for standalone builds
    if(COMMAND add_lomiri_unittest)
        add_lomiri_unittest(${util_test} ${util_test}TestExec ADD_TEST
            ENVIRONMENT LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}/plugins/Utils HOME=${TMPDIR}
        )
    else()
        enable_testing()
        add_test(NAME ${util_test}Test
            COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${util_test}TestExec
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        )
        set_tests_properties(${util_test}Test PROPERTIES
            ENVIRONMENT "LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}/plugins/Utils;HOME=${TMPDIR}"
        )
    endif()

endforeach()

# plain qml test
if(COMMAND add_lomiri_qmlunittest)
    add_lomiri_qmlunittest(. UtilsStyle)
endif()

# URLDispatcher test needs dbus-test-runner because it claims a dbus name
# Note: Requires Qt6::DBus
add_executable(URLDispatcherTestExec 
    URLDispatcherTest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugins/Utils/URLDispatcher.cpp
)
target_link_libraries(URLDispatcherTestExec Qt6::Test Qt6::Core Qt6::Qml Qt6::Quick Qt6::Sql Qt6::DBus)
install(TARGETS URLDispatcherTestExec
    DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Utils"
)

if(COMMAND add_lomiri_unittest)
    add_lomiri_unittest(URLDispatcher dbus-test-runner
        ARG_PREFIX "--parameter"
        ARGS --task $<TARGET_FILE:URLDispatcherTestExec>
        ENVIRONMENT LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}/plugins/Utils
    )
else()
    enable_testing()
    add_test(NAME URLDispatcherTest
        COMMAND ${CMAKE_CURRENT_BINARY_DIR}/URLDispatcherTestExec
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
    set_tests_properties(URLDispatcherTest PROPERTIES
        ENVIRONMENT "LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}/plugins/Utils"
    )
endif()
