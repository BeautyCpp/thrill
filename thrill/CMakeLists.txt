################################################################################
# thrill/CMakeLists.txt
#
# CMake file for Thrill library
#
# Part of Project Thrill - http://project-thrill.org
#
# Copyright (C) 2015 Timo Bingmann <tb@panthema.net>
#
# All rights reserved. Published under the BSD-2 license in the LICENSE file.
################################################################################

# glob general sources
file(GLOB THRILL_SRCS
  RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/api/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/common/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/core/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/data/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/io/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/mem/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/net/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/net/mock/*.[ch]pp
  ${CMAKE_CURRENT_SOURCE_DIR}/vfs/*.[ch]pp
  )

# add net/tcp on all platforms except Windows
if(NOT MSVC)
  file(GLOB THRILL_NET_TCP_SRCS
    RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/net/tcp/*.[ch]pp)

  list(APPEND THRILL_SRCS ${THRILL_NET_TCP_SRCS})
endif()

# add net/mpi if MPI is wanted
if(MPI_FOUND)
  file(GLOB THRILL_NET_MPI_SRCS
    RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/net/mpi/*.[ch]pp)

  list(APPEND THRILL_SRCS ${THRILL_NET_MPI_SRCS})
endif()

add_library(thrill STATIC ${THRILL_SRCS})
target_compile_definitions(thrill PUBLIC ${THRILL_DEFINITIONS})
#target_include_directories(thrill PUBLIC ${PROJECT_SOURCE_DIR})
target_include_directories(thrill PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}> $<INSTALL_INTERFACE:include>)
target_include_directories(thrill SYSTEM PUBLIC ${THRILL_INCLUDE_DIRS})
target_link_libraries(thrill ${THRILL_LINK_LIBRARIES})

################################################################################
####
# Installation (https://github.com/forexample/package-example)

# Layout. This works for all platforms:
#   * <prefix/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: PROJECT_VERSION is used as a VERSION
# write_basic_package_version_file(
#     "${version_config}" COMPATIBILITY SameMajorVersion
# )

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "../cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

# Install the library and produce a CMake export script
install(TARGETS thrill
	EXPORT "${targets_export_name}"
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	INCLUDES DESTINATION include
	FRAMEWORK DESTINATION Frameworks
)
if (APPLE AND BUILD_FRAMEWORK)
	set_target_properties(thrill PROPERTIES OUTPUT_NAME thrill FRAMEWORK ON)
else()
	set_target_properties(thrill PROPERTIES OUTPUT_NAME thrill)

endif()

install(
		DIRECTORY . 
		DESTINATION include/thrill
    FILES_MATCHING PATTERN "*.hpp"
	)
# Config
#   * <prefix>/lib/cmake/Async++/Async++Config.cmake
#   * <prefix>/lib/cmake/Async++/Async++ConfigVersion.cmake
install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}"
)

# Config
#   * <prefix>/cmake/Async++/FooTargets.cmake
install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

SET(CPACK_GENERATOR "DEB")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "none") #required

INCLUDE(CPack)
