cmake_minimum_required(VERSION 3.16)

project(liborl VERSION 0.4.3)
include(CTest)

add_compile_options(-fPIC -Wall -Wextra)

find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED CONFIG)
find_package(Franka REQUIRED)
message(STATUS "libfranka version: ${Franka_VERSION}")

add_library(orl SHARED
    3rdparty/libfranka/examples_common.cpp
    src/geometry_math.cpp
    src/Payload.cpp
    src/Pose.cpp
    src/PoseGenerator.cpp
    src/Robot.cpp
    src/SCurve.cpp
    src/SpeedProfile.cpp
    src/StopCondition.cpp
    include/liborl/enums.h
    include/liborl/Frame.h
    include/liborl/geometry_math.h
    include/liborl/SCurve.h
    include/liborl/SpeedProfile.h
    include/liborl/StopCondition.h
)
set_target_properties(orl PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    OUTPUT_NAME "orl"
    INTERFACE_INCLUDE_DIRECTORIES "${Boost_INCLUDE_DIRS};${EIGEN3_INCLUDE_DIR}"
)
target_include_directories(orl PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libfranka/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${Boost_INCLUDE_DIRS}>
    $<BUILD_INTERFACE:${EIGEN3_INCLUDE_DIR}>
)
target_link_libraries(orl PUBLIC Franka::Franka)
include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/orl)
install(DIRECTORY include/liborl/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/liborl
    FILES_MATCHING PATTERN "*.h*"
)

install(TARGETS orl
    EXPORT orlTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    orlConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
)
configure_package_config_file(cmake/orlConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/orlConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/orlConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/orlConfigVersion.cmake"
    DESTINATION ${INSTALL_CONFIGDIR}
)
install(EXPORT orlTargets
    FILE orlTargets.cmake
    NAMESPACE orl::
    DESTINATION ${INSTALL_CONFIGDIR}
)

option(BUILD_TESTS "Build tests" OFF)
if (BUILD_TESTS)
    add_subdirectory(examples)
    enable_testing()
    add_subdirectory(test)
endif ()
