cmake_minimum_required(VERSION 3.0)
project(jpeg-compressor VERSION 1.0.4)

option(BUILD_SHARED_LIBS "Build shared libraries" FALSE)
option(BUILD_EXE "build exe" FALSE)

if(MSVC AND BUILD_SHARED_LIBS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

# jgpd lib

set(JPGD_SRC_LIST jpgd.cpp)
set(JPGD_HDR_LIST jpgd.h jpgd_idct.h)
if(BUILD_SHARED_LIBS)
  add_library(jpgd SHARED ${JPGD_SRC_LIST} ${JPGD_HDR_LIST})
else()
  add_library(jpgd STATIC ${JPGD_SRC_LIST} ${JPGD_HDR_LIST})
endif()
target_include_directories(jpgd PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>")

# jpge lib

set(JPGE_SRC_LIST jpge.cpp)
set(JPGE_HDR_LIST jpge.h jpge.h)
if(BUILD_SHARED_LIBS)
  add_library(jpge SHARED ${JPGE_SRC_LIST} ${JPGE_HDR_LIST})
else()
  add_library(jpge STATIC ${JPGE_SRC_LIST} ${JPGE_HDR_LIST})
endif()
target_include_directories(jpge PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>")

# jgpe executable

if(BUILD_EXE)
  set(STB_SRC_LIST stb_image.h stb_image_write.h)
  set(JPGEEXE_SRC_LIST tga2jpg.cpp timer.h timer.cpp)

  add_executable(jpge_exe ${JPGEEXE_SRC_LIST} ${STB_SRC_LIST})
  set_target_properties(jpge_exe PROPERTIES OUTPUT_NAME "jpge")
  target_include_directories(jpge_exe PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
  target_link_libraries(jpge_exe jpgd jpge)

  install(TARGETS jpge_exe DESTINATION bin)
endif()

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")

set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY ExactVersion
)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
    INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    TARGETS jpgd jpge
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

install(
    FILES ${JPGD_HDR_LIST} ${JPGE_HDR_LIST}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

install(
    FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)
