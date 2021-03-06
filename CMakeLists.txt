cmake_minimum_required (VERSION 2.8)

Project(smart_city NONE)

if (NOT DEFINED REGISTRY)
    set(REGISTRY "")
elseif (NOT ${REGISTRY} MATCHES "/$")
    set(REGISTRY "${REGISTRY}/")
endif()
if (NOT DEFINED PLATFORM)
    set(PLATFORM "Xeon")
endif()
if (NOT DEFINED FRAMEWORK)
    set(FRAMEWORK "gst")
endif()
if (NOT DEFINED SCENARIO)
    set(SCENARIO "traffic")
endif()
if (NOT DEFINED NOFFICES)
    set(NOFFICES "1")
endif()
if (NOT DEFINED NCAMERAS)
    set(NCAMERAS "5")
endif()
if (NOT DEFINED NANALYTICS)
    if(SCENARIO MATCHES "stadium")
        set(NANALYTICS "1,0,1")
    else()
        set(NANALYTICS "1")
    endif()
endif()
if (NOT DEFINED NETWORK)
    if(PLATFORM STREQUAL "Xeon")
        set(NETWORK "INT8,FP32")
    endif()
    if(PLATFORM STREQUAL "VCAC-A")
        set(NETWORK "FP16")
    endif()
endif()

if(PLATFORM STREQUAL "Xeon")
    if((NOT (NETWORK MATCHES "FP32")) AND (NOT (NETWORK MATCHES "INT8")))
        if(NOT (NETWORK MATCHES "FP32"))
            set(NETWORK "FP32,${NETWORK}")
        endif()
        if(NOT (NETWORK MATCHES "INT8"))
            set(NETWORK "INT8,${NETWORK}")
        endif()
    endif()
    if(NETWORK MATCHES "FP16")
        string(REGEX REPLACE "FP16,*" "" NETWORK ${NETWORK})
    endif()
endif()
if(PLATFORM STREQUAL "VCAC-A")
    if(NOT (NETWORK MATCHES "FP16"))
        set(NETWORK "FP16,${NETWORK}")
    endif()
    if(NETWORK MATCHES "FP32")
        string(REGEX REPLACE "FP32,*" "" NETWORK ${NETWORK})
    endif()
    if(NETWORK MATCHES "INT8")
        string(REGEX REPLACE "INT8,*" "" NETWORK ${NETWORK})
    endif()
endif()
string(REGEX REPLACE ",$" "" NETWORK ${NETWORK})

if(NOT REGISTRY)
    add_custom_target(update ${CMAKE_HOME_DIRECTORY}/script/update-image.sh)
endif()

file(GLOB dirs "deployment" "*")
list(REMOVE_DUPLICATES dirs)
foreach (dir ${dirs})
    if(EXISTS ${dir}/CMakeLists.txt)
        add_subdirectory(${dir})
    endif()
endforeach()

# legal message
execute_process(COMMAND printf "\n\nThis script will build third party components licensed under various open source licenses into your container images. The terms under which those components may be used and distributed can be found with the license document that is provided with those components. Please familiarize yourself with those terms to ensure your distribution of those components complies with the terms of those licenses.\n\n")


execute_process(COMMAND printf "\n-- Setting: PLATFORM=${PLATFORM}, SCENARIO=${SCENARIO}, NOFFICES=${NOFFICES}, NCAMERAS=${NCAMERAS}, NANALYTICS=${NANALYTICS}, FRAMEWORK=${FRAMEWORK}, NETWORK=${NETWORK}\n")
execute_process(COMMAND printf "-- Setting: REGISTRY=${REGISTRY}\n")
