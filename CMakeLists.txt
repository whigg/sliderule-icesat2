# ICESat-2 SlideRule Plugin top-level CMake build script

project (ICESAT2 LANGUAGES CXX)

#################
# CMake Options #
#################

set (CMAKE_LEGACY_CYGWIN_WIN32 0) # Squelch a warning when building on Win32/Cygwin
cmake_minimum_required (VERSION 3.13.0) # The minimum CMake version is chosen to enable policy CMP0079
cmake_policy(SET CMP0079 NEW) # add link library to target which is not built in this directory
cmake_policy(SET CMP0053 NEW) # simplified variable escape processing (recommended by cmake)

###################
# ICESat-2 Plugin #
###################

add_library (icesat2 MODULE "")

set_target_properties (icesat2 PROPERTIES OUTPUT_NAME icesat2)
set_target_properties (icesat2 PROPERTIES PREFIX "")

# Prerequisites #
find_package (Lua "5.3")

# Version Information #
file(STRINGS ${PROJECT_SOURCE_DIR}/version.txt TGTVER)
execute_process (COMMAND git --git-dir ${PROJECT_SOURCE_DIR}/.git --work-tree ${PROJECT_SOURCE_DIR} describe --abbrev --dirty --always --tags --long OUTPUT_VARIABLE BUILDINFO)
string(REGEX REPLACE "\n$" "" BUILDINFO "${BUILDINFO}")
target_compile_definitions (icesat2 PUBLIC BUILDINFO="${BUILDINFO}")

# Set Environment Variables
set (INSTALLDIR /usr/local CACHE STRING "Installation directory for library and executables")
set (CONFDIR ${INSTALLDIR}/etc/sliderule)

# Compile Definitions #
target_compile_definitions (icesat2 PUBLIC BINID="${TGTVER}")

# Source Files #
target_sources(icesat2
    PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}/plugin/icesat2.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/lua_parms.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/Atl03Reader.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/Atl03Indexer.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/Atl06Dispatch.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/CumulusIODriver.cpp
        ${CMAKE_CURRENT_LIST_DIR}/plugin/UT_Atl06Dispatch.cpp
)

# Include Directories #
target_include_directories (icesat2
    PUBLIC
        ${INSTALLDIR}/include/sliderule
        ${LUA_INCLUDE_DIR}
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>/plugin
)

# Script Installation #
install (
    FILES
        ${CMAKE_CURRENT_LIST_DIR}/endpoints/atl06.lua
        ${CMAKE_CURRENT_LIST_DIR}/endpoints/atl03s.lua
        ${CMAKE_CURRENT_LIST_DIR}/endpoints/indexer.lua
    DESTINATION
        ${CONFDIR}
)

# Plugin Installation #
install (TARGETS icesat2 LIBRARY DESTINATION ${CONFDIR})
