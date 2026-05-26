cmake_minimum_required(VERSION 3.16)

project(kiero VERSION 2.0.0 LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(KIERO_BUILD_D3D9 "Build the Direct3D 9 backend" ${WIN32})
option(KIERO_BUILD_D3D10 "Build the Direct3D 10 backend" ${WIN32})
option(KIERO_BUILD_D3D11 "Build the Direct3D 11 backend" ${WIN32})
option(KIERO_BUILD_D3D12 "Build the Direct3D 12 backend" ${WIN32})
option(KIERO_BUILD_OPENGL "Build the OpenGL backend" ON)
option(KIERO_BUILD_VULKAN "Build the Vulkan backend" ${WIN32})
option(KIERO_INSTALL "Install kiero targets and CMake package files" ${PROJECT_IS_TOP_LEVEL})

if(NOT WIN32)
  foreach(kiero_windows_backend IN ITEMS
    KIERO_BUILD_D3D9
    KIERO_BUILD_D3D10
    KIERO_BUILD_D3D11
    KIERO_BUILD_D3D12
    KIERO_BUILD_VULKAN
  )
    if(${kiero_windows_backend})
      message(FATAL_ERROR "${kiero_windows_backend} is only supported on Windows")
    endif()
  endforeach()
endif()

set(kiero_public_headers
  kiero.hpp
  kiero_intern.hpp
  kiero_d3d9.hpp
  kiero_d3d10.hpp
  kiero_d3d11.hpp
  kiero_d3d12.hpp
  kiero_opengl.hpp
  kiero_vulkan.hpp
)

set(kiero_sources
  kiero_intern.cpp
)

if(KIERO_BUILD_D3D9)
  list(APPEND kiero_sources kiero_d3d9.cpp)
endif()

if(KIERO_BUILD_D3D10)
  list(APPEND kiero_sources kiero_d3d10.cpp)
endif()

if(KIERO_BUILD_D3D11)
  list(APPEND kiero_sources kiero_d3d11.cpp)
endif()

if(KIERO_BUILD_D3D12)
  list(APPEND kiero_sources kiero_d3d12.cpp)
endif()

if(KIERO_BUILD_OPENGL)
  list(APPEND kiero_sources kiero_opengl.cpp)
endif()

if(KIERO_BUILD_VULKAN)
  list(APPEND kiero_sources kiero_vulkan.cpp)
endif()

add_library(kiero ${kiero_sources} ${kiero_public_headers})
add_library(kiero::kiero ALIAS kiero)

target_compile_features(kiero PUBLIC cxx_std_17)

set_target_properties(kiero PROPERTIES
  CXX_EXTENSIONS OFF
  POSITION_INDEPENDENT_CODE ON
)

target_include_directories(kiero
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(WIN32)
  target_link_libraries(kiero PUBLIC user32)
elseif(UNIX AND NOT APPLE)
  if(KIERO_BUILD_OPENGL)
    target_link_libraries(kiero PUBLIC ${CMAKE_DL_LIBS})
  endif()
endif()

target_compile_definitions(kiero
  PUBLIC
    $<$<BOOL:${KIERO_BUILD_D3D9}>:KIERO_BUILD_D3D9=1>
    $<$<BOOL:${KIERO_BUILD_D3D10}>:KIERO_BUILD_D3D10=1>
    $<$<BOOL:${KIERO_BUILD_D3D11}>:KIERO_BUILD_D3D11=1>
    $<$<BOOL:${KIERO_BUILD_D3D12}>:KIERO_BUILD_D3D12=1>
    $<$<BOOL:${KIERO_BUILD_OPENGL}>:KIERO_BUILD_OPENGL=1>
    $<$<BOOL:${KIERO_BUILD_VULKAN}>:KIERO_BUILD_VULKAN=1>
)

if(KIERO_INSTALL)
  install(
    TARGETS kiero
    EXPORT kieroTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

  install(
    FILES ${kiero_public_headers}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

  install(
    EXPORT kieroTargets
    NAMESPACE kiero::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/kiero
  )

  write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/kieroConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
  )

  configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/kieroConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/kieroConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/kiero"
  )

  install(
    FILES
      "${CMAKE_CURRENT_BINARY_DIR}/kieroConfig.cmake"
      "${CMAKE_CURRENT_BINARY_DIR}/kieroConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/kiero"
  )
endif()
