cmake_minimum_required (VERSION 3.1)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_BUILD_TYPE Debug)


project(TessellationDemo)

macro(copy_files_once)
    set(FILES_LIST "${ARGN}")
    foreach(SOME_FILE ${FILES_LIST})
        message(STATUS "Copying ${SOME_FILE} to build folder")
        file(COPY ${SOME_FILE} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
    endforeach()
endmacro()

message(STATUS "**** Tessellation Demo, Loading Configuration ****")

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    set(CMAKE_CXX_FLAGS "-w" CACHE STRING "COMPILER OPTIONS" FORCE)
else()
    set(CMAKE_CXX_FLAGS "-Wall -Wno-unused-variable -Wno-unused-function -Wno-unused-parameter" CACHE STRING "COMPILER OPTIONS" FORCE)
endif()

# disable GLFW docs, examples and tests
# see http://www.glfw.org/docs/latest/build_guide.html
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)


# set path to dependencies
set(SUBMOD_DIR common/submodules)

add_subdirectory(${SUBMOD_DIR}/glfw)
include_directories(${SUBMOD_DIR}/glfw/include)
include_directories(${SUBMOD_DIR}/glm)
include_directories(${SUBMOD_DIR}/imgui)
include_directories(${SUBMOD_DIR}/stb)
include_directories(${SUBMOD_DIR}/dj_opengl)
include_directories(${SUBMOD_DIR}/tinyobjloader)


# imgui implementation (compiled as a library)
aux_source_directory(${SUBMOD_DIR}/imgui IMGUI_SRC_FILES)
set(IMGUI_INCLUDE_DIR  common/imgui_impl ${SUBMOD_DIR}/imgui)
add_library(imgui STATIC common/imgui_impl/imgui_impl.cpp ${IMGUI_SRC_FILES})
include_directories(${IMGUI_INCLUDE_DIR})

# glad implementation (compiled as a library)
add_library(glad STATIC common/glad/glad/glad.c)
include_directories(common/glad)


# OBj files
file(GLOB_RECURSE OBJ_FILES *.obj)
copy_files_once(${OBJ_FILES})


# **************** Project DEMO **************** #

 message(STATUS "**** Compute Tessellation DEMO ****")

 set(SRC_DIR ComputeTess_demo/)

 file(GLOB_RECURSE SHADERS ${SRC_DIR}/*.glsl)
 file(GLOB_RECURSE HEADERS ${SRC_DIR}/*.h)

 include_directories(${SRC_DIR})
 aux_source_directory(${SRC_DIR} SRC_FILES)

 add_executable(demo ${SRC_FILES} ${SHADERS} ${HEADERS})

 target_link_libraries(demo glad glfw imgui)

 unset(SRC_FILES)
 unset(SHADERS)
 unset(HEADERS)


# **************** Project BENCHMARK **************** #

message(STATUS "**** Compute Tessellation BENCHMARK ****")

set(SRC_DIR ComputeTess_bench/)

file(GLOB_RECURSE SHADERS ${SRC_DIR}/*.glsl)
file(GLOB_RECURSE HEADERS ${SRC_DIR}/*.h)

include_directories(${SRC_DIR})
aux_source_directory(${SRC_DIR} SRC_FILES)

add_executable(bench ${SRC_FILES} ${SHADERS} ${HEADERS})

target_link_libraries(bench glad glfw imgui)

unset(SRC_FILES)
unset(SHADERS)
unset(HEADERS)