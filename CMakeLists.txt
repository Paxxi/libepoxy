cmake_minimum_required(VERSION 3.0)

project(epoxy VERSION 1.3.1 LANGUAGES C)

find_package(PythonInterp REQUIRED)

option(BUILD_EGL "Build EGL interface" OFF)
option(BUILD_GL "Build OpenGL interface" OFF)
option(BUILD_GLX "Build GLX interface" OFF)
option(BUILD_WGL "Build WGL interface" ON)

set(SRC ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(REGISTRY ${CMAKE_CURRENT_SOURCE_DIR}/registry)
set(PY_GEN ${SRC}/gen_dispatch.py)

add_library(epoxy
  ${SRC}/dispatch_common.c
  ${SRC}/dispatch_common.h
)
add_custom_command(
  OUTPUT
  ${CMAKE_CURRENT_BINARY_DIR}/include/epoxy/gl_generated.h
  ${CMAKE_CURRENT_BINARY_DIR}/src/gl_generated_dispatch.c
  COMMAND ${PYTHON_EXECUTABLE} ${PY_GEN} --dir ${CMAKE_CURRENT_BINARY_DIR} ${REGISTRY}/gl.xml
)
if(BUILD_WGL)
  add_custom_command(
    OUTPUT
    ${CMAKE_CURRENT_BINARY_DIR}/include/epoxy/wgl_generated.h
    ${CMAKE_CURRENT_BINARY_DIR}/src/wgl_generated_dispatch.c
    COMMAND ${PYTHON_EXECUTABLE} ${PY_GEN} --dir ${CMAKE_CURRENT_BINARY_DIR} ${REGISTRY}/wgl.xml
  )
  target_sources(epoxy
    PRIVATE
    ${SRC}/dispatch_wgl.c
    ${CMAKE_CURRENT_SOURCE_DIR}/include/epoxy/wgl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/epoxy/gl.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/epoxy/wgl_generated.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/epoxy/gl_generated.h
    ${CMAKE_CURRENT_BINARY_DIR}/src/wgl_generated_dispatch.c
    ${CMAKE_CURRENT_BINARY_DIR}/src/gl_generated_dispatch.c
  )
  target_include_directories(epoxy
    PRIVATE
    $<BUILD_INTERFACE:include/;${CMAKE_CURRENT_BINARY_DIR}/include/;src/>
  )
endif()