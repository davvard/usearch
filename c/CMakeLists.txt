

set(USEARCH_PUNNED_INCLUDE_DIRS
    "${CMAKE_CURRENT_SOURCE_DIR}/../include"
    "${CMAKE_CURRENT_SOURCE_DIR}/../fp16/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/../simsimd/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/"
)

# This article discusses a better way to allow building either static or shared libraries
# https://alexreinking.com/blog/building-a-dual-shared-and-static-library-with-cmake.html
if (${USEARCH_BUILD_STATIC})
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -static-libstdc++")
    elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -static")
    endif()

    add_library(usearch_c STATIC "${CMAKE_CURRENT_SOURCE_DIR}/lib.cpp")
else()
    add_library(usearch_c SHARED "${CMAKE_CURRENT_SOURCE_DIR}/lib.cpp")
endif()

set_target_properties(usearch_c PROPERTIES POSITION_INDEPENDENT_CODE ON)
set_target_properties(usearch_c PROPERTIES CXX_STANDARD 11)
set_target_properties(usearch_c PROPERTIES C_STANDARD 99)

target_include_directories(usearch_c PRIVATE ${USEARCH_PUNNED_INCLUDE_DIRS})
set_target_properties(usearch_c PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set_target_properties(usearch_c PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set_target_properties(usearch_c PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})


target_include_directories(usearch_c PRIVATE ${USEARCH_PUNNED_INCLUDE_DIRS})

if(${USEARCH_DEBUG_BUILD_ASAN})
    # For ASAN in debug builds
    target_link_libraries(usearch_c PRIVATE gcov)
endif()

if(${USEARCH_USE_SIMSIMD})
    target_compile_definitions(usearch_c PRIVATE USEARCH_USE_SIMSIMD=1)
endif()
if (${USEARCH_LOOKUP_LABEL})
    target_compile_definitions(usearch_c PRIVATE USEARCH_LOOKUP_LABEL=1)
endif()

if(${USEARCH_BUILD_TEST})
    add_executable(test_c "${CMAKE_CURRENT_SOURCE_DIR}/test.c")
    target_link_libraries(test_c usearch_c)
    set_target_properties(test_c PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
endif()
