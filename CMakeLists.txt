
# Mandatory line, sets the minimum version of CMake that should be used with this repository.
# I specified 3.22 because I trust it. However, currently I have 3.26 installed on my machine.
# To verify your version run
# $ cmake --version
cmake_minimum_required(VERSION 3.22)

# Sets a few variables, like PROJECT_NAME
project(PS1-Reverb)

# Always use the newest C++ standard on green-field projects if possible.
set(CMAKE_CXX_STANDARD 23)

# I like to download the dependencies to the same folder as the project.
# If you want to install them system wide, set CPM_SOURCE_CACHE with the path to the dependencies
# either as an environment variable or pass it to the cmake script with -DCPM_SOURCE_CACHE=<path>.
set(LIB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libs)

# Downloads CPM if not already downloaded. CPM is an easy-to-use package manager nicely integrated with CMake.
include(cmake/cpm.cmake)

# This commands downloads AND configures JUCE. It sets up some variables, like JUCE_SOURCE_DIR.
CPMAddPackage(
    NAME JUCE
    GIT_TAG 7.0.9
    VERSION 7.0.9
    GITHUB_REPOSITORY juce-framework/JUCE
    SOURCE_DIR ${LIB_DIR}/juce
)

# Adds googletest.
CPMAddPackage(
    NAME GOOGLETEST
    GITHUB_REPOSITORY google/googletest
    GIT_TAG v1.14.0
    VERSION 1.14.0
    SOURCE_DIR ${LIB_DIR}/googletest
    OPTIONS
        "INSTALL_GTEST OFF"
        "gtest_force_shared_crt ON"
)

# This command allows running tests from the "build" folder (the one where CMake generates the project to).
enable_testing()

# Enables all warnings and treats warnings as errors.
# This needs to be added AFTER all the third-party dependencies.
if(MSVC)
    add_compile_options(/Wall /WX)
else()
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Adds all the targets configured in the "plugin" folder.
add_subdirectory(plugin)

# Adds all the targets configured in the "test" folder.
add_subdirectory(test)


# IR files configuration
set(IR_DIR "${CMAKE_CURRENT_SOURCE_DIR}/plugin/source/IR")
set(IR_FILES
    "${IR_DIR}/IRChurch.wav"
    "${IR_DIR}/IRDome.wav"
    "${IR_DIR}/IRHall.wav"
    "${IR_DIR}/IRStudio.wav"
)

# Copy IR files to build directory
foreach(IR_FILE ${IR_FILES})
    get_filename_component(IR_FILE_NAME ${IR_FILE} NAME)
    configure_file(${IR_FILE} ${CMAKE_BINARY_DIR}/IR/${IR_FILE_NAME} COPYONLY)
endforeach()
