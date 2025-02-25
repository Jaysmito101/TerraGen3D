cmake_minimum_required(VERSION 3.12)


set_property(GLOBAL PROPERTY USE_FOLDERS ON)

project(TerraForge3D C CXX)

if(APPLE)
    set(EXECUTABLE_NAME "tf3d")
else()
    set(EXECUTABLE_NAME "terraforge3d")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REUIRED True)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REUIRED True)

find_package( OpenGL REQUIRED )
set(OPENSSL_USE_STATIC_LIBS TRUE)
find_package(OpenSSL REQUIRED)

include_directories( 
    ./TerraForge3D/include
    ./TerraForge3D/include/Base
    ./TerraForge3D/vendor
    ./TerraForge3D/vendor/tinyexr
    ./TerraForge3D/vendor/avir
    ./TerraForge3D/vendor/delaunator/include
    ./TerraForge3D/vendor/glad/include
    ./TerraForge3D/vendor/glfw/include
    ./TerraForge3D/vendor/fmt/include
    ./TerraForge3D/vendor/glm
    ./TerraForge3D/vendor/httplib
    ./TerraForge3D/vendor/imgui
    ./TerraForge3D/vendor/imgui-node-editor
    ./TerraForge3D/vendor/json/single_include/nlohmann
    ./TerraForge3D/vendor/stb
    ./TerraForge3D/vendor/libpng
    ${OPENSSL_INCLUDE_DIR}
)

set( GLFW_BUILD_DOCS OFF CACHE BOOL  "GLFW lib only" )
set( GLFW_INSTALL OFF CACHE BOOL  "GLFW lib only" )

add_subdirectory(./TerraForge3D/vendor/glfw)
add_subdirectory(./TerraForge3D/vendor/glad)
add_subdirectory(./TerraForge3D/vendor/imgui-builder)
add_subdirectory(./TerraForge3D/vendor/libwebp)
add_subdirectory(./TerraForge3D/vendor/fmt)

if( MSVC )
    SET( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /ENTRY:mainCRTStartup" )
endif()

file(GLOB_RECURSE SOURCES
     ./TerraForge3D/src/**.cpp
     ./TerraForge3D/src/TerraForge3D.rc
)

# list(APPEND SOURCES ./TerraForge3D/vendor/imgui/backends/imgui_impl_glfw.cpp)
# list(APPEND SOURCES ./TerraForge3D/vendor/imgui/backends/imgui_impl_opengl3.cpp)

add_executable(${EXECUTABLE_NAME} ${SOURCES})

target_link_libraries(${EXECUTABLE_NAME} 
    glfw
    ${GLFW_LIBRARIES}
    ${OPENGL_LIBRARIES}
    Glad
    ImGui
)

if(WIN32)
    target_compile_definitions(${EXECUTABLE_NAME} 
        PUBLIC _CRT_SECURE_NO_WARNINGS
        PUBLIC TERR3D_WIN32
    )
endif()

if(APPLE)
    target_link_libraries(${EXECUTABLE_NAME} "-framework Security")
endif()

target_link_libraries(${EXECUTABLE_NAME} OpenSSL::SSL)
target_link_libraries(${EXECUTABLE_NAME} OpenSSL::Crypto)
target_link_libraries(${EXECUTABLE_NAME} fmt::fmt)
target_link_libraries(${EXECUTABLE_NAME} webp)

if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19.12.25835)
    set(CMAKE_CXX20_STANDARD_COMPILE_OPTION "-std:c++latest")
    set(CMAKE_CXX20_EXTENSION_COMPILE_OPTION "-std:c++latest")
endif()

if( MSVC )
    if(${CMAKE_VERSION} VERSION_LESS "3.6.0") 
        message( "\n\t[ WARNING ]\n\n\tCMake version lower than 3.6.\n\n\t - Please update CMake and rerun; OR\n\t - Manually set 'GLFW-CMake-starter' as StartUp Project in Visual Studio.\n" )
    else()
        set_property( DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT terraforge3d )
    endif()
endif()
