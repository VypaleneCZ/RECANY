cmake_minimum_required(VERSION 3.16)
project(KronikarLabe)

# C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Debug/Release configuration
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -DDEBUG")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -DNDEBUG")
endif()

# Find packages
find_package(OpenGL REQUIRED)
find_package(glfw3 REQUIRED)
find_package(GLEW REQUIRED)
find_package(OpenAL REQUIRED)
find_package(Bullet REQUIRED)

# Include directories
include_directories(${CMAKE_SOURCE_DIR}/src)
include_directories(${CMAKE_SOURCE_DIR}/shaders)
include_directories(${CMAKE_SOURCE_DIR}/assets)

# Source files
set(SOURCES
    src/main.cpp
    src/GameEngine.cpp
    src/Renderer.cpp
    src/Shader.cpp
    src/Camera.cpp
    src/Light.cpp
    src/Mesh.cpp
    src/RenderSystem.cpp
    src/InputSystem.cpp
    src/AudioSystem.cpp
    src/UISystem.cpp
    src/SaveLoadSystem.cpp
    src/AISystem.cpp
    src/QuestSystem.cpp
    src/PhysicsEngine.cpp
    src/Player.cpp
    src/Inventory.cpp
    src/Skills.cpp
    src/Equipment.cpp
    src/Matrix3.cpp
)

# Header files
set(HEADERS
    src/ISystem.h
    src/GameEngine.h
    src/Renderer.h
    src/Shader.h
    src/Camera.h
    src/Light.h
    src/Mesh.h
    src/RenderSystem.h
    src/Vector3.h
    src/Vector2.h
    src/Vector4.h
    src/Matrix4.h
    src/Matrix3.h
    src/InputSystem.h
    src/AudioSystem.h
    src/UISystem.h
    src/SaveLoadSystem.h
    src/AISystem.h
    src/QuestSystem.h
    src/PhysicsEngine.h
    src/Player.h
    src/Inventory.h
    src/Skills.h
    src/Equipment.h
)

# Shader files
file(GLOB SHADERS "shaders/*.vert" "shaders/*.frag")

# Create executable
add_executable(KronikarLabe ${SOURCES} ${HEADERS} ${SHADERS})

# Link libraries
target_link_libraries(KronikarLabe 
    ${OPENGL_LIBRARIES}
    glfw
    ${GLEW_LIBRARIES}
    ${OPENAL_LIBRARY}
    Bullet::Bullet3Common
    Bullet::LinearMath
    Bullet::BulletCollision
    Bullet::BulletDynamics
)

# Platform-specific libraries
if(WIN32)
    target_link_libraries(KronikarLabe opengl32)
elseif(APPLE)
    find_library(COCOA_LIBRARY Cocoa)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(COREVIDEO_LIBRARY CoreVideo)
    target_link_libraries(KronikarLabe ${COCOA_LIBRARY} ${IOKIT_LIBRARY} ${COREVIDEO_LIBRARY})
elseif(UNIX)
    target_link_libraries(KronikarLabe X11)
endif()

# Compiler-specific options
if(MSVC)
    target_compile_options(KronikarLabe PRIVATE /W4)
    target_compile_definitions(KronikarLabe PRIVATE _CRT_SECURE_NO_WARNINGS)
else()
    target_compile_options(KronikarLabe PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Copy shaders to build directory
file(COPY ${SHADERS} DESTINATION ${CMAKE_BINARY_DIR}/shaders)

# Copy assets to build directory
file(COPY ${CMAKE_SOURCE_DIR}/assets DESTINATION ${CMAKE_BINARY_DIR})

# Set working directory
set_target_properties(KronikarLabe PROPERTIES
    VS_DEBUGGER_WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    VS_DEBUGGER_COMMAND_ARGUMENTS ""
)

# Install targets
install(TARGETS KronikarLabe DESTINATION bin)
install(DIRECTORY assets DESTINATION share/KronikarLabe)
install(DIRECTORY shaders DESTINATION share/KronikarLabe)

# Package configuration
set(CPACK_PACKAGE_NAME "KronikarLabe")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Medieval RPG hra Kronikář Labe: Rok 1142")
set(CPACK_PACKAGE_VENDOR "Independent Developer")
set(CPACK_PACKAGE_CONTACT "developer@example.com")

if(WIN32)
    set(CPACK_GENERATOR "NSIS")
    set(CPACK_NSIS_DISPLAY_NAME "Kronikář Labe: Rok 1142")
    set(CPACK_NSIS_PACKAGE_NAME "KronikarLabe")
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop")
else()
    set(CPACK_GENERATOR "DEB;RPM")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "developer@example.com")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libglfw3, libglew2.1, libopenal1, libbullet-dev")
endif()

include(CPack)

# Custom targets for development
add_custom_target(run
    COMMAND KronikarLabe
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    DEPENDS KronikarLabe
    COMMENT "Running the game"
)

add_custom_target(debug
    COMMAND gdb ./KronikarLabe
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    DEPENDS KronikarLabe
    COMMENT "Running the game with debugger"
)

# Testing
enable_testing()
add_subdirectory(tests)

# Documentation
find_package(Doxygen)
if(DOXYGEN_FOUND)
    set(DOXYGEN_INPUT_DIR ${CMAKE_SOURCE_DIR}/src)
    set(DOXYGEN_OUTPUT_DIR ${CMAKE_BINARY_DIR}/docs)
    
    configure_file(${CMAKE_SOURCE_DIR}/Doxyfile.in ${CMAKE_BINARY_DIR}/Doxyfile @ONLY)
    
    add_custom_target(docs
        ${DOXYGEN_EXECUTABLE} ${CMAKE_BINARY_DIR}/Doxyfile
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen"
    )
endif()

# Profiling
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_custom_target(profile
        COMMAND valgrind --tool=callgrind ./KronikarLabe
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        DEPENDS KronikarLabe
        COMMENT "Running the game with Valgrind profiler"
    )
endif()

# Memory checking
add_custom_target(memcheck
    COMMAND valgrind --leak-check=full --show-leak-kinds=all ./KronikarLabe
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    DEPENDS KronikarLabe
    COMMENT "Running the game with Valgrind memory checker"
)

add_test(NAME BasicTest COMMAND KronikarLabe --test)
