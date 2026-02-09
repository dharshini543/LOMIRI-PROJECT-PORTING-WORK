cmake_minimum_required(VERSION 3.16)
project(IndicatorsTests)

set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Core DBus Test Qml REQUIRED)
include(FindPkgConfig)
pkg_check_modules(GLIB REQUIRED glib-2.0)
pkg_check_modules(GIO REQUIRED gio-2.0)
# pkg_check_modules(QMENUMODEL REQUIRED ayatana-menumodel)
 pkg_check_modules(QMENUMODEL REQUIRED libayatana-ido3-0.4)

set(INDICATORS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../../../plugins/Lomiri/Indicators)
set(TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../..)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../../include/paths.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/paths.h
    @ONLY
)
add_definitions(-DQT_NO_KEYWORDS)

include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${INDICATORS_DIR}
    ${TEST_DIR}/mocks/QMenuModel.1
    ${GLIB_INCLUDE_DIRS}
    ${GIO_INCLUDE_DIRS}
    ${QMENUMODEL_INCLUDE_DIRS}
    )

macro(indicator_test TEST)
    set(multi_value_keywords ADDITIONAL_CPPS ADDITIONAL_LIBS)
    cmake_parse_arguments(test "" "" "${multi_value_keywords}" ${ARGN})

    add_executable(${TEST}Exec
            ${TEST}Test.cpp
            ${test_ADDITIONAL_CPPS}
        )
    target_link_libraries(${TEST}Exec
        Qt6::Test Qt6::Core Qt6::Qml Qt6::DBus
        ${test_ADDITIONAL_LIBS}
        ${GLIB_LIBRARIES}
        ${GIO_LIBRARIES}
        ${QMENUMODEL_LIBRARIES}
        )
    install(TARGETS ${TEST}Exec
        DESTINATION "${SHELL_PRIVATE_LIBEXECDIR}/tests/plugins/Lomiri/Indicators"
        )

    set(ldpath "")
    foreach(lib ${test_ADDITIONAL_LIBS})
        set(ldpath "$<TARGET_FILE_DIR:${lib}>:")
    endforeach(lib)

    # Use add_lomiri_unittest when building from root project
    # Otherwise fallback to add_test for standalone builds
    if(COMMAND add_lomiri_unittest)
        add_lomiri_unittest(${TEST} ${TEST}Exec
            ENVIRONMENT "LD_LIBRARY_PATH=${ldpath}"
            )
    else()
        set(TEST_ENV "QML2_IMPORT_PATH=${TEST_DIR}/qml:${TEST_DIR}/plugins")
        set(TEST_ENV "${TEST_ENV} LOMIRI_SOURCE_DIR=${TEST_DIR}")

        enable_testing()
        add_test(NAME ${TEST}
            COMMAND ${TEST}Exec
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            )
        set_tests_properties(${TEST} PROPERTIES ENVIRONMENT "${TEST_ENV}")
    endif()
endmacro(indicator_test)

indicator_test(IndicatorsManager ADDITIONAL_LIBS IndicatorsQml)
indicator_test(IndicatorsModel ADDITIONAL_LIBS IndicatorsQml)
indicator_test(MenuContentActivator ADDITIONAL_CPPS ${INDICATORS_DIR}/menucontentactivator.cpp)
indicator_test(LomiriMenuModelStack ADDITIONAL_CPPS ${TEST_DIR}/mocks/QMenuModel.1/ayatanamenumodel.cpp ${INDICATORS_DIR}/lomirimenumodelstack.cpp)
indicator_test(RootActionState ADDITIONAL_LIBS IndicatorsQml)
indicator_test(SharedLomiriMenuModel ADDITIONAL_LIBS IndicatorsQml)
