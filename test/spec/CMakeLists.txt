# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2019-2022 Second State INC

include(FetchContent)

message(STATUS "Downloading the WASM spec test suite")
FetchContent_Declare(
  wasmedge_unit_test
  GIT_REPOSITORY https://github.com/second-state/WasmEdge-unittest
  GIT_TAG        wasm-dev-0.13.0
)
FetchContent_MakeAvailable(wasmedge_unit_test)
message(STATUS "Downloading the WASM spec test suite -- done")

message(STATUS "Downloading SIMDJSON")
include(FetchContent)
FetchContent_Declare(
  simdjson
  GIT_REPOSITORY https://github.com/simdjson/simdjson.git
  GIT_TAG  tags/v3.2.0
  GIT_SHALLOW TRUE)

  get_property(
    compile_options
    DIRECTORY
    PROPERTY COMPILE_OPTIONS
    )
  set_property(
    DIRECTORY
    APPEND
    PROPERTY COMPILE_OPTIONS
    -Wno-conditional-uninitialized
    -Wno-implicit-int-conversion
    -Wno-zero-as-null-pointer-constant
    
    )
  unset(compile_options)
  set(RAPIDJSON_BUILD_DOC OFF CACHE BOOL "Build SIMDJSON documentation." FORCE)
  set(RAPIDJSON_BUILD_CXX11 OFF CACHE BOOL "Build RapidJSON with C++11" FORCE)
  set(RAPIDJSON_BUILD_CXX17 ON CACHE BOOL "Build RapidJSON with C++17" FORCE)
FetchContent_MakeAvailable(simdjson)
message(STATUS "Downloading SIMDJSON -- done")

function(wasmedge_copy_spec_testsuite proposal)
  message(STATUS "Copying test suite to ${CMAKE_CURRENT_BINARY_DIR}/testSuites/${proposal}")
  file(COPY
    ${wasmedge_unit_test_SOURCE_DIR}/${proposal}
    DESTINATION
    ${CMAKE_CURRENT_BINARY_DIR}/testSuites
  )
  message(STATUS "Copying test suite to ${CMAKE_CURRENT_BINARY_DIR}/testSuites/${proposal} -- done")
endfunction()

foreach(PROPOSAL core multi-memory tail-call extended-const threads)
  wasmedge_copy_spec_testsuite(${PROPOSAL})
endforeach()

wasmedge_add_library(wasmedgeTestSpec
  spectest.cpp
)

target_include_directories(wasmedgeTestSpec SYSTEM PUBLIC simdjson)

target_link_libraries(wasmedgeTestSpec
  PUBLIC
  simdjson::simdjson
  std::filesystem
  wasmedgeCommon
  ${GTEST_BOTH_LIBRARIES}
)