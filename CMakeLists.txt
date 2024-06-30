cmake_minimum_required(VERSION 3.28)
project(project-von-neumann)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
file(GLOB_RECURSE PROJECT_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_LIST_DIR}/sources/*.cpp")

# Generate compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Dependencies

# RAYLIB #
find_package(raylib QUIET)
if (NOT raylib_FOUND)
    include(FetchContent)
    FetchContent_Declare(
            raylib
            GIT_REPOSITORY https://github.com/raysan5/raylib.git
            GIT_TAG 5.0
            GIT_SHALLOW 1
    )
    FetchContent_MakeAvailable(raylib)
endif()

# RAYLIB-CPP #
find_package(raylib_cpp QUIET)
if (NOT raylib_cpp_FOUND)
    if (NOT DEFINED RAYLIB_CPP_VERSION)
        set(RAYLIB_CPP_VERSION v5.0.2)
    endif()
    include(FetchContent)
    FetchContent_Declare(
            raylib_cpp
            GIT_REPOSITORY https://github.com/RobLoach/raylib-cpp.git
            GIT_TAG ${RAYLIB_CPP_VERSION}
    )
    FetchContent_MakeAvailable(raylib_cpp)
endif()

# ENTT #
set(ENTT_VERSION 3.13.2)
find_package(EnTT::EnTT ${ENTT_VERSION} QUIET) # QUIET or REQUIRED
if (NOT entt_FOUND) # If there's none, fetch and build entt
    include(FetchContent)
    FetchContent_Declare(
            entt
            DOWNLOAD_EXTRACT_TIMESTAMP OFF
            URL https://github.com/skypjack/entt/archive/refs/tags/v${ENTT_VERSION}.tar.gz
    )
    FetchContent_GetProperties(EnTT::EnTT)
    if (NOT entt_POPULATED) # Have we downloaded entt yet?
        set(FETCHCONTENT_QUIET NO)
        FetchContent_Populate(entt)
        add_subdirectory(${entt_SOURCE_DIR} ${entt_BINARY_DIR})
        list(APPEND CMAKE_PREFIX_PATH ${entt_BINARY_DIR}) # endy wont install without this
    endif()
endif()

# Our Project

add_executable(${PROJECT_NAME} main.cpp
)
#set(raylib_VERBOSE 1)
target_link_libraries(${PROJECT_NAME} raylib)
target_link_libraries(${PROJECT_NAME} raylib_cpp)
target_link_libraries(${PROJECT_NAME} EnTT::EnTT)

# Web Configurations
if (${PLATFORM} STREQUAL "Web")
    # Tell Emscripten to build an example.html file.
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".html")
endif()

# Checks if OSX and links appropriate frameworks (Only required on MacOS)
if (APPLE)
    target_link_libraries(${PROJECT_NAME} "-framework IOKit")
    target_link_libraries(${PROJECT_NAME} "-framework Cocoa")
    target_link_libraries(${PROJECT_NAME} "-framework OpenGL")
endif()