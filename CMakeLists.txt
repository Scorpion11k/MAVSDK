cmake_minimum_required(VERSION 3.1)

# Extract command line args into ${CL_ARGS} in order to propagate them to the superbuild dependencies 
get_cmake_property(vars CACHE_VARIABLES)
foreach(var ${vars})
  get_property(currentHelpString CACHE "${var}" PROPERTY HELPSTRING)
    if("${currentHelpString}" MATCHES "No help, variable specified on the command line." OR "${currentHelpString}" STREQUAL "")
        #message("${var} = [${${var}}]  --  ${currentHelpString}") # uncomment to see the variables being processed
        list(APPEND CL_ARGS "-D${var}=${${var}}")
    endif()
endforeach()

project(mavsdk_superbuild)

option(SUPERBUILD "Build dependencies" ON)
option(BUILD_BACKEND "Build gRPC backend server" OFF)
option(BUILD_SHARED_LIBS "Build core as shared libraries instead of static ones" ON)

set(DEPS_BUILD_PATH "${PROJECT_BINARY_DIR}/third_party" CACHE PATH "Install path for the dependencies. Ignored if SUPERBUILD=OFF.")
set(DEPS_INSTALL_PATH "${DEPS_BUILD_PATH}/install" CACHE PATH "Install path for the dependencies. Ignored if SUPERBUILD=OFF.")

include(GNUInstallDirs)

if(SUPERBUILD)
    add_subdirectory(third_party)
endif()

add_subdirectory(src)

install(EXPORT mavsdk-targets
    FILE MAVSDKTargets.cmake
    NAMESPACE MAVSDK::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MAVSDK
    )

# For the build tree
configure_file(MAVSDKConfig.cmake.in
    "${PROJECT_BINARY_DIR}/MAVSDKConfig.cmake" @ONLY)
# For the install tree
configure_file(MAVSDKConfig.cmake.in
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/MAVSDKConfig.cmake" @ONLY)

install(FILES
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/MAVSDKConfig.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MAVSDK)
