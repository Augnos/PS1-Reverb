cmake_minimum_required(VERSION 3.22)

# Version is needed by JUCE.
project(AudioPlugin VERSION 0.1.0)

# Adding a directory with the library/application name as a subfolder of the
# include folder is a good practice. It helps avoid name clashes later on.
set(INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include/PS1-Reverb")

# IR files configuration
set(IR_DIR "${CMAKE_CURRENT_SOURCE_DIR}/source/IR")
set(IR_FILES
    "${IR_DIR}/IRChurch.aiff"
    "${IR_DIR}/IRDome.aiff"
    "${IR_DIR}/IRHall.aiff"
    "${IR_DIR}/IRStudio.aiff"
)

# Adds a plugin target (that's basically what the Projucer does).
juce_add_plugin(${PROJECT_NAME}
    COMPANY_NAME AugnosMedia
    IS_SYNTH FALSE
    NEEDS_MIDI_INPUT FALSE
    NEEDS_MIDI_OUTPUT FALSE
    PLUGIN_MANUFACTURER_CODE AUGN
    PLUGIN_CODE PS1R
    FORMATS VST3 Standalone
    PRODUCT_NAME "PS1 Reverb"
)

# Sets the source files of the plugin project.
target_sources(${PROJECT_NAME}
    PRIVATE
        source/PluginEditor.cpp
        source/PluginProcessor.cpp
        ${INCLUDE_DIR}/PluginEditor.h
        ${INCLUDE_DIR}/PluginProcessor.h
        ${IR_FILES} # Add the IR files to the project
)

# Sets the include directories of the plugin project.
target_include_directories(${PROJECT_NAME}
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${IR_DIR} # Include the IR Directory
)

# Links to all necessary dependencies. The present ones are recommended by JUCE.
# If you use one of the additional modules, like the DSP module, you need to specify it here.
target_link_libraries(${PROJECT_NAME}
    PRIVATE
        juce::juce_audio_utils
        juce::juce_dsp # Add the JUCE DSP module for convolution and other DSP tasks
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags
)

# These definitions are recommended by JUCE.
target_compile_definitions(${PROJECT_NAME}
    PUBLIC
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0
        JucePlugin_Name="${PROJECT_NAME}"
)

# Silences some deprecation warnings that (hopefully) will be fixed in a future release of JUCE.
if (MSVC)
target_compile_definitions(${PROJECT_NAME}
    PRIVATE
        _SILENCE_CXX23_ALIGNED_STORAGE_DEPRECATION_WARNING)
endif()

# In Visual Studio this command provides a nice grouping of source files in "filters".
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/..)

# Copy IR files to build directory
foreach(IR_FILE ${IR_FILES})
    get_filename_component(IR_FILE_NAME ${IR_FILE} NAME)
    configure_file(${IR_FILE} ${CMAKE_BINARY_DIR}/IR/${IR_FILE_NAME} COPYONLY)
endforeach()