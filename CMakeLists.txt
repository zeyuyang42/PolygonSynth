cmake_minimum_required(VERSION 3.15)

# Setup basic information
set(COMPANY_NAME NyLP)
set(PROJECT_NAME PolygonSynth)
set(PRODUCT_NAME Polygon)


# Project info from the JUCER file
project(Polygon VERSION 1.0.0)

# Basic build setups
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_XCODE_GENERATE_SCHEME OFF)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# We're going to use CPM as our package manager to bring in JUCE
# Check to see if we have CPM installed already.  Bring it in if we don't.
set(CPM_DOWNLOAD_VERSION 0.34.0)
set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")

if (NOT EXISTS ${CPM_DOWNLOAD_LOCATION})
    message(STATUS "Downloading CPM.cmake to ${CPM_DOWNLOAD_LOCATION}")
    file(DOWNLOAD https://github.com/cpm-cmake/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake ${CPM_DOWNLOAD_LOCATION})
endif ()

include(${CPM_DOWNLOAD_LOCATION})

# Bring in JUCE locally
CPMAddPackage(
    NAME juce
    GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
    GIT_TAG origin/master
)

# Create a list of source files
set(SourceFiles
    Source/DcBlocker.cpp
    Source/DcBlocker.h
    Source/Oscilloscope2D.h
    Source/PluginEditor.cpp
    Source/PluginEditor.h
    Source/PluginProcessor.cpp
    Source/PluginProcessor.h
    Source/PolygonAlgorithm.cpp
    Source/PolygonAlgorithm.h
    Source/PolygonAudioSource.cpp
    Source/PolygonAudioSource.h
    Source/PolygonDisplay.cpp
    Source/PolygonDisplay.h
    Source/PolygonSound.h
    Source/PolygonVoice.cpp
    Source/PolygonVoice.h
    Source/RingBuffer.h
    Source/TuningEditor.cpp
    Source/TuningEditor.h
    
    # HIIR files
    Source/hiir/PolyphaseIir2Designer.cpp
    Source/hiir/PolyphaseIir2Designer.h
    Source/hiir/def.h
    Source/hiir/Downsampler2xFpu.h
    Source/hiir/Downsampler2xFpu.hpp
    Source/hiir/Downsampler2xSse.h
    Source/hiir/Downsampler2xSse.hpp
    Source/hiir/fnc.h
    Source/hiir/fnc.hpp
    Source/hiir/StageProcFpu.h
    Source/hiir/StageProcFpu.hpp
)


# Plugin preferences
juce_add_plugin(${PROJECT_NAME}
        COMPANY_NAME                 ${COMPANY_NAME}
        IS_SYNTH                     TRUE
        NEEDS_MIDI_INPUT             TRUE
        NEEDS_MIDI_OUTPUT            FALSE
        IS_MIDI_EFFECT               FALSE
        EDITOR_WANTS_KEYBOARD_FOCUS  FALSE
        JUCE_VST3_CAN_REPLACE_VST2   FALSE
        COPY_PLUGIN_AFTER_BUILD      TRUE
        PLUGIN_MANUFACTURER_CODE     Nylp
        PLUGIN_CODE                  Aaab
        FORMATS                      VST3 AU Standalone
        PRODUCT_NAME                 ${PRODUCT_NAME}
)

# `juce_generate_juce_header` will create a JuceHeader.h for a given target, which will be generated
# into your build tree. This should be included with `#include <JuceHeader.h>`. The include path for
# this header will be automatically added to the target. The main function of the JuceHeader is to
# include all your JUCE module headers; if you're happy to include module headers directly, you
# probably don't need to call this.
juce_generate_juce_header(${PROJECT_NAME})


# How we want our SourceFiles to appear in our IDE
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${SourceFiles})


# `target_sources` adds source files to a target. We pass the target that needs the sources as the
# first argument, then a visibility parameter for the sources which should normally be PRIVATE.
# Finally, we supply a list of source files that will be built into the target. This is a standard
# CMake command.
# Make the SourceFiles buildable
target_sources(${PROJECT_NAME} PRIVATE ${SourceFiles})


# `target_compile_definitions` adds some preprocessor definitions to our target. In a Projucer
# project, these might be passed in the 'Preprocessor Definitions' field. JUCE modules also make use
# of compile definitions to switch certain features on/off, so if there's a particular feature you
# need that's not on by default, check the module header for the correct flag to set here. These
# definitions will be visible both to your code, and also the JUCE module code, so for new
# definitions, pick unique names that are unlikely to collide! This is a standard CMake command.

target_compile_definitions(${PROJECT_NAME}
    PRIVATE
        JUCE_WEB_BROWSER=0  
        JUCE_USE_CURL=0     
        JUCE_APPLICATION_NAME_STRING="$<TARGET_PROPERTY:${PROJECT_NAME},JUCE_PRODUCT_NAME>"
        JUCE_APPLICATION_VERSION_STRING="$<TARGET_PROPERTY:${PROJECT_NAME},JUCE_VERSION>")


target_link_libraries(${PROJECT_NAME}
    PRIVATE
        juce::juce_gui_extra
        juce::juce_audio_formats
        juce::juce_audio_plugin_client
        juce::juce_cryptography
        juce::juce_events
        juce::juce_analytics
        juce::juce_audio_basics
        juce::juce_audio_devices
        juce::juce_core
        juce::juce_data_structures
        juce::juce_graphics
        juce::juce_gui_basics
        juce::juce_audio_utils
        juce::juce_dsp
        juce::juce_opengl
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags
)