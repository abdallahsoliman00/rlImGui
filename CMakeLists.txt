cmake_minimum_required(VERSION 3.15)
project(rlImGui)

# Options
set(GRAPHICS_OPTIONS "opengl11" "opengl21" "opengl33" "opengl43")
set(GRAPHICS "opengl33" CACHE STRING "Version of OpenGL to build raylib against")
set_property(CACHE GRAPHICS PROPERTY STRINGS ${GRAPHICS_OPTIONS})

# Set C and C++ standards
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Configuration types
set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

# Platform definitions
if(UNIX AND NOT APPLE)
    add_definitions(-D_GLFW_X11 -D_GNU_SOURCE)
endif()

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE})

# Function to check and download raylib
function(check_raylib)
    if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raylib" AND NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master")
        if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master.zip")
            message(STATUS "Raylib not found, downloading from github")
            file(DOWNLOAD
                "https://github.com/raysan5/raylib/archive/refs/heads/master.zip"
                "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master.zip"
                SHOW_PROGRESS
            )
        endif()
        message(STATUS "Extracting raylib to ${CMAKE_CURRENT_SOURCE_DIR}")
        file(ARCHIVE_EXTRACT INPUT "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master.zip"
             DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}")
        file(REMOVE "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master.zip")
    endif()
endfunction()

# Function to check and download imgui
function(check_imgui)
    if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/imgui" AND NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master")
        if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master.zip")
            message(STATUS "imgui not found, downloading from github")
            file(DOWNLOAD
                "https://github.com/ocornut/imgui/archive/refs/heads/master.zip"
                "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master.zip"
                SHOW_PROGRESS
            )
        endif()
        message(STATUS "Extracting imgui to ${CMAKE_CURRENT_SOURCE_DIR}")
        file(ARCHIVE_EXTRACT INPUT "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master.zip"
             DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}")
        file(REMOVE "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master.zip")
    endif()
endfunction()

# Download dependencies
check_raylib()
check_imgui()

# Add raylib subdirectory
# This assumes raylib-master contains a CMakeLists.txt
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master")
    add_subdirectory(raylib-master)
    set(RAYLIB_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/raylib-master/src")
elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/raylib")
    add_subdirectory(raylib)
    set(RAYLIB_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/raylib/src")
endif()

# Determine imgui directory
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master")
    set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/imgui-master")
elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/imgui")
    set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/imgui")
endif()

# rlImGui static library
file(GLOB IMGUI_SOURCES
    "${IMGUI_DIR}/*.cpp"
    "${IMGUI_DIR}/*.h"
)

file(GLOB RLIMGUI_SOURCES
    "*.cpp"
    "*.h"
    "extras/**.h"
)

add_library(rlImGui STATIC
    ${IMGUI_SOURCES}
    ${RLIMGUI_SOURCES}
)

target_include_directories(rlImGui PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/rlImGui
    ${IMGUI_DIR}
    ${RAYLIB_INCLUDE_DIRS}
)

target_compile_definitions(rlImGui PUBLIC
    IMGUI_DISABLE_OBSOLETE_FUNCTIONS
    IMGUI_DISABLE_OBSOLETE_KEYIO
)

target_link_libraries(rlImGui PUBLIC raylib)

# Examples
add_executable(simple examples/simple.cpp)
target_link_libraries(simple PRIVATE rlImGui raylib)
target_include_directories(simple PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${IMGUI_DIR}
)

add_executable(editor examples/editor.cpp)
target_link_libraries(editor PRIVATE rlImGui raylib)
target_include_directories(editor PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${IMGUI_DIR}
)

add_executable(imgui_style_example examples/imgui_style_example.cpp)
target_link_libraries(imgui_style_example PRIVATE rlImGui raylib)
target_include_directories(imgui_style_example PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${IMGUI_DIR}
)

add_executable(docking_example examples/docking_example.cpp)
target_link_libraries(docking_example PRIVATE rlImGui raylib)
target_include_directories(docking_example PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${IMGUI_DIR}
)

file(GLOB ASSET_BROWSER_SOURCES
    "examples/asset_browser/*.cpp"
    "examples/asset_browser/*.h"
)

add_executable(asset_browser ${ASSET_BROWSER_SOURCES})
target_link_libraries(asset_browser PRIVATE rlImGui raylib)
target_include_directories(asset_browser PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/examples/asset_browser
    ${IMGUI_DIR}
)

# Set working directory for Visual Studio
if(MSVC)
    set_target_properties(simple editor imgui_style_example docking_example asset_browser
        PROPERTIES
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
    )
endif()