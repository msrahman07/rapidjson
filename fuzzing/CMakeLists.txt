if(FALSE)
set(PERFTEST_SOURCES
    misctest.cpp
    perftest.cpp
    platformtest.cpp
    rapidjsontest.cpp
    schematest.cpp)

add_executable(perftest ${PERFTEST_SOURCES})
target_link_libraries(perftest ${TEST_LIBRARIES})

add_dependencies(tests perftest)

find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Qunused-arguments -fcolor-diagnostics")
    endif()
endif(CCACHE_FOUND)

set_property(DIRECTORY PROPERTY COMPILE_OPTIONS ${EXTRA_CXX_FLAGS})

IF(NOT (CMAKE_BUILD_TYPE STREQUAL "Debug"))
add_test(NAME perftest
    COMMAND ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/perftest
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
ENDIF()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=fuzzer,address")

set_target_properties(libfuzzer_fuzzer
	PROPERTIES LINK_FLAGS "-g -fsanitize=fuzzer,address"
)

endif()


if(FALSE)

cmake_minimum_required(VERSION 2.8)

if(POLICY CMP0054)
  cmake_policy(SET CMP0054 NEW)
endif()

set(EXAMPLES
    capitalize
    condense
    filterkey
    filterkeydom
    jsonx
    lookaheadparser
    messagereader
    parsebyparts
    pretty
    prettyauto
    schemavalidator
    serialize
    simpledom
    simplereader
    simplepullreader
    simplewriter
    sortkeys
    tutorial)
    
include_directories("../include/")

add_definitions(-D__STDC_FORMAT_MACROS)
set_property(DIRECTORY PROPERTY COMPILE_OPTIONS ${EXTRA_CXX_FLAGS})

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
endif()

add_executable(archivertest archiver/archiver.cpp archiver/archivertest.cpp)

foreach (example ${EXAMPLES})
    add_executable(${example} ${example}/${example}.cpp)
endforeach()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_link_libraries(parsebyparts pthread)
endif()

add_custom_target(examples ALL DEPENDS ${EXAMPLES})


set(LIBFUZZER_FLAGS_BASE "${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS "-g -fsanitize=fuzzer,address")

include_directories("../include/")

endif()

if(FALSE)

add_executable(libfuzzer_fuzzer "")
target_sources(libfuzzer_fuzzer
	PRIVATE "${PROJECT_SOURCE_DIR}/fuzzing/libfuzzer_fuzztarget_01.cpp"
)
target_link_libraries(libfuzzer_fuzzer "")
set_target_properties(libfuzzer_fuzzer
	PROPERTIES LINK_FLAGS "-g -fsanitize=address,fuzzer"
)

endif()

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
if(POLICY CMP0025)
  # detect Apple's Clang
  cmake_policy(SET CMP0025 NEW)
endif()
if(POLICY CMP0054)
  cmake_policy(SET CMP0054 NEW)
endif()

option(RAPIDJSON_BUILD_CXX11 "Build rapidjson with C++11 (gcc/clang)" ON)
if(RAPIDJSON_BUILD_CXX11)
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
endif()

set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang)
project (libfuzzer_fuzzer)

include_directories("../include/")

add_executable(libfuzzer_fuzzer libfuzzer_fuzztarget_01.cpp)
target_compile_options(libfuzzer_fuzzer
            PRIVATE $<$<CXX_COMPILER_ID:Clang>:-g -fsanitize=fuzzer,address>
            )

target_link_libraries(libfuzzer_fuzzer
            PRIVATE $<$<CXX_COMPILER_ID:Clang>:-fsanitize=fuzzer,address>
            )