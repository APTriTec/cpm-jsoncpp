cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME jsoncpp)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# JsonCpp 
#------------------------------------------------------------------------------
# JsonCpp as a subdirectory so that it can capture your compiler flags
set(JSONCPP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/jsoncpp)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${JSONCPP_DIR}
  GIT_REPOSITORY "https://github.com/open-source-parsers/jsoncpp.git"
  GIT_TAG 1.6.0
  USE_CACHING TRUE
  )

set(ModuleHeaders
  ${JSONCPP_DIR}/include/json/config.h
  ${JSONCPP_DIR}/include/json/forwards.h
  ${JSONCPP_DIR}/include/json/features.h
  ${JSONCPP_DIR}/include/json/value.h
  ${JSONCPP_DIR}/include/json/reader.h
  ${JSONCPP_DIR}/include/json/writer.h
  ${JSONCPP_DIR}/include/json/assertions.h
  ${JSONCPP_DIR}/include/json/version.h
  )

set(ModuleSources
  "${JSONCPP_DIR}/src/lib_json/json_reader.cpp"
  "${JSONCPP_DIR}/src/lib_json/json_value.cpp"
  "${JSONCPP_DIR}/src/lib_json/json_writer.cpp"
  "${JSONCPP_DIR}/src/lib_json/version.h.in"
  "${JSONCPP_DIR}/src/lib_json/json_tool.h"
  "${JSONCPP_DIR}/src/lib_json/json_valueiterator.inl")

include_directories(${JSONCPP_DIR}/include)
add_library(${CPM_LIB_TARGET_NAME} STATIC ${ModuleHeaders} ${ModuleSources})

CPM_ExportAdditionalIncludeDir("${JSONCPP_DIR}/include")

