cmake_minimum_required(VERSION 3.14)
project(LightSimulator CXX C)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_CURRENT_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_CURRENT_SOURCE_DIR}")

# ?? OpenGL (system) ????????????????????????????????????????????????????????????
find_package(OpenGL REQUIRED)

# ?? GLAD (vendored source) ????????????????????????????????????????????????????
add_library(glad STATIC
    glad/src/glad.c
)
target_include_directories(glad PUBLIC
    glad/include
)
target_link_libraries(glad PUBLIC OpenGL::GL)

# ?? GLFW (downloaded and built from source via FetchContent) ??????????????????
# The vendored copy only contains a 32-bit pre-built lib with no source,
# so we fetch the matching 3.2.1 source and let CMake build it for any target.
include(FetchContent)
set(GLFW_BUILD_DOCS     OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS    OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
FetchContent_Declare(
    glfw
    URL https://github.com/glfw/glfw/releases/download/3.4/glfw-3.4.zip
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)
FetchContent_MakeAvailable(glfw)
unset(CMAKE_POLICY_VERSION_MINIMUM)

# ?? SOIL (vendored source) ????????????????????????????????????????????????????
set(SOIL_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/soil/Simple OpenGL Image Library/src")
add_library(soil STATIC
    "${SOIL_SRC_DIR}/SOIL.c"
    "${SOIL_SRC_DIR}/image_helper.c"
    "${SOIL_SRC_DIR}/image_DXT.c"
    "${SOIL_SRC_DIR}/stb_image_aug.c"
)
target_include_directories(soil PUBLIC "${SOIL_SRC_DIR}")
target_link_libraries(soil PUBLIC OpenGL::GL)

# ?? ImGui (vendored source) ???????????????????????????????????????????????????
# imgui_impl_opengl3 uses <glad/glad.h> when IMGUI_IMPL_OPENGL_LOADER_GLAD is defined.
# imgui_impl_glfw    uses <glfw3.h> and <glfw3native.h>.
add_library(imgui STATIC
    OpenGL/imgui/imgui.cpp
    OpenGL/imgui/imgui_draw.cpp
    OpenGL/imgui/imgui_widgets.cpp
    OpenGL/imgui/imgui_impl_glfw.cpp
    OpenGL/imgui/imgui_impl_opengl3.cpp
)
target_compile_definitions(imgui PUBLIC IMGUI_IMPL_OPENGL_LOADER_GLAD)
target_include_directories(imgui PUBLIC
    OpenGL/imgui
    ${glfw_SOURCE_DIR}/include/GLFW  # imgui uses <glfw3.h> (no GLFW/ prefix)
    glad/include                     # <glad/glad.h>
)
target_link_libraries(imgui PUBLIC glfw glad OpenGL::GL)

# ?? LightSimulator executable ?????????????????????????????????????????????????
add_executable(LightSimulator
    main/main.cpp
    program/program.cpp
    resourceManager/resourceManager.cpp
    resourceManager/texture/texture.cpp
    shader/shader.cpp
    objectManager/objectManager.cpp
    objectManager/object/object.cpp
    objectManager/object/objectTemplate.cpp
    objectManager/object/angleIndicator/angleIndicator.cpp
    objectManager/object/lensObject/lensObject.cpp
    objectManager/object/lightObject/lightObject.cpp
    renderers/spriteRenderer/SpriteRenderer.cpp
    renderers/rayRenderer/RayRenderer.cpp
    guiManager/guiManager.cpp
    databaseManager/dbManager.cpp
)

target_include_directories(LightSimulator PRIVATE
    # Project root: resolves quoted includes like "program/program.h",
    # "OpenGL/imgui/imgui.h", "OpenGL/glad/include/glad/glad.h",
    # "OpenGL/glfw-3.2.1/include/glfw3.h", etc.
    ${CMAKE_CURRENT_SOURCE_DIR}
    # Resolves angle-bracket includes like <glm/glm.hpp>
    ${CMAKE_CURRENT_SOURCE_DIR}/OpenGL
    # GLFW headers from fetched source (used by main.cpp via OpenGL/glfw-3.2.1/include path)
    ${glfw_SOURCE_DIR}/include
    # Resolves <SOIL.h>
    "${SOIL_SRC_DIR}"
)

target_link_libraries(LightSimulator PRIVATE
    glad
    glfw
    soil
    imgui
    OpenGL::GL
)

# On Windows, also link against gdi32 and user32 which GLFW depends on
if(WIN32)
    target_link_libraries(LightSimulator PRIVATE gdi32 user32 shell32)
endif()
