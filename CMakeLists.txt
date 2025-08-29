cmake_minimum_required(VERSION 3.16)

project(liborl VERSION 0.4.0)
include(CTest)

add_compile_options(-fPIC)

add_compile_options(-Wall -Wextra)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED CONFIG)
find_package(Franka REQUIRED)
message(STATUS "libfranka version: ${Franka_VERSION}")

include_directories(${Boost_INCLUDE_DIRS})
include_directories(${EIGEN3_INCLUDE_DIR})

add_library(orl SHARED
        src/Robot.cpp
        3rdparty/libfranka/examples_common.cpp
        src/Pose.cpp src/geometry_math.cpp include/liborl/geometry_math.h src/PoseGenerator.cpp src/Payload.cpp src/StopCondition.cpp include/liborl/StopCondition.h src/SpeedProfile.cpp include/liborl/SpeedProfile.h include/liborl/Frame.h src/SCurve.cpp include/liborl/SCurve.h include/liborl/enums.h)
set_target_properties(orl PROPERTIES VERSION ${PROJECT_VERSION}
        )
target_include_directories(orl
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libfranka/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        )

target_link_libraries(orl PUBLIC
        Franka::Franka
        )
include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/orl)
install(DIRECTORY include/liborl/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/liborl
        FILES_MATCHING PATTERN "*.h*"
        )

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(TARGETS orl
        EXPORT orlTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include
        )
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        orlConfigVersion.cmake
        VERSION ${PACKAGE_VERSION}
        COMPATIBILITY AnyNewerVersion
)
configure_package_config_file(cmake/orlConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/orlConfig.cmake
        INSTALL_DESTINATION ${INSTALL_CONFIGDIR})
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/orlConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/orlConfigVersion.cmake"
        DESTINATION lib/cmake/orl
        )
install(EXPORT orlTargets
        FILE orlTargets.cmake
        NAMESPACE orl::
        DESTINATION lib/cmake/orl
        )

option(BUILD_TESTS "Build tests" OFF)
if (BUILD_TESTS)
    add_subdirectory(examples)
    enable_testing()
    add_subdirectory(test)
endif ()

## Packaging
set(CPACK_PACKAGE_VENDOR "FZJ GmbH")
set(CPACK_GENERATOR "DEB;TGZ")
set(CPACK_PACKAGE_VERSION ${liborl_VERSION})
set(CPACK_SYSTEM_NAME ${CMAKE_HOST_SYSTEM_PROCESSOR})
# Debian versions require a dash
set(CPACK_DEBIAN_PACKAGE_VERSION ${CPACK_PACKAGE_VERSION}-1)
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "FZJ GmbH")
include(CPack)
