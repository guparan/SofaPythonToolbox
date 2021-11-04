cmake_minimum_required(VERSION 3.12)

project(SofaPythonToolbox VERSION 4.0)

include(cmake/environment.cmake)

find_package(SofaFramework REQUIRED)
sofa_find_package(SofaPython QUIET)
sofa_find_package(SofaPython3 QUIET)

if(SofaPython3_FOUND)
    message("-- Found SofaPython3")
    SP3_add_python_package(
        SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/python3/splib
        TARGET_DIRECTORY splib
    )
    SP3_add_python_package(
        SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/python3/splib3
        TARGET_DIRECTORY splib3
    )
    SP3_add_python_package(
        SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/python3/stlib
        TARGET_DIRECTORY stlib
    )
    SP3_add_python_package(
        SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/python3/stlib3
        TARGET_DIRECTORY stlib3
    )
endif()

if(SofaPython_FOUND)
    message("-- Found SofaPython")
    sofa_install_pythonscripts(PLUGIN_NAME ${PROJECT_NAME} PYTHONSCRIPTS_SOURCE_DIR "python" PYTHONSCRIPTS_INSTALL_DIR "python")
endif()

if(NOT SofaPython_FOUND AND NOT SofaPython3_FOUND)
    message(SEND_ERROR "Missing a python implementation (SofaPython or SofaPython3).")
endif()

set(HEADER_FILES src/SofaPythonToolbox/initPlugin.h)
set(SOURCE_FILES src/SofaPythonToolbox/initPlugin.cpp)

file(GLOB_RECURSE RESOURCE_FILES  "*.md" "*.psl" "*.py" "*.pyscn" "*.scn" "*.rst")

add_library(${PROJECT_NAME} SHARED ${HEADER_FILES} ${SOURCE_FILES} ${RESOURCE_FILES})
target_link_libraries(${PROJECT_NAME} SofaCore)

## Install rules for the library and headers; CMake package configurations files
sofa_create_package_with_targets(
    PACKAGE_NAME ${PROJECT_NAME}
    PACKAGE_VERSION ${PROJECT_VERSION}
    TARGETS ${PROJECT_NAME} AUTO_SET_TARGET_PROPERTIES
    INCLUDE_SOURCE_DIR src
    INCLUDE_INSTALL_DIR ${PROJECT_NAME}
    RELOCATABLE "plugins"
    )

install(DIRECTORY docs DESTINATION share/sofa/${PROJECT_NAME})
install(FILES "README.md" "LICENSE" DESTINATION .)

# Tests
# If SOFA_BUILD_TESTS exists and is OFF, then these tests will be auto-disabled
cmake_dependent_option(SOFAPYTHONTOOLBOX_BUILD_SPLIB_TESTS "Compile the tests" ON "SOFA_BUILD_TESTS OR NOT DEFINED SOFA_BUILD_TESTS" OFF)
if(SOFAPYTHONTOOLBOX_BUILD_SPLIB_TESTS)
    enable_testing()
    add_subdirectory(SPLIB_test)
endif()

include(cmake/packaging.cmake)
