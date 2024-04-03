cmake_minimum_required(VERSION 3.18)
project(imgui)

if(WIN32) 
    set(CMAKE_DEBUG_POSTFIX d)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(IMGUI_GLFW_SUPPORT      ON CACHE BOOL "Enable glfw")
set(IMGUI_OPENGL3_SUPPORT   ON CACHE BOOL "Enable opengl3")
set(IMGUI_VULKAN_SUPPORT    OFF CACHE BOOL "Enable vulkan")
if(WIN32)
    set(IMGUI_DIRECTX12_SUPPORT OFF CACHE BOOL "Enable directx12")
endif()

find_package(lux-cmake-toolset	REQUIRED)

set(IMGUI_CORE_SRCS
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui_demo.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui_draw.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui_tables.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui_widgets.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui.cpp
)

set(IMGUI_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/imconfig.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui_internal.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imstb_rectpack.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imstb_textedit.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imstb_truetype.h
)

add_component(
    COMPONENT_NAME  core
    STATIC
    NAMESPACE       imgui
    OUTPUT_NAME     imgui_core
	SOURCE_FILES	${IMGUI_CORE_SRCS}
)

component_include_directories(
    core 
    BUILD_TIME_SHARED 
        ${CMAKE_CURRENT_SOURCE_DIR} 
        ${CMAKE_CURRENT_SOURCE_DIR}/backends
    INSTALL_TIME
        include
)

set(IMGUI_BACKENDS_HEADERS)
set(IMGUI_COMPONENTS core)

if(IMGUI_OPENGL3_SUPPORT)
    add_component(
        COMPONENT_NAME  opengl3
        STATIC
        NAMESPACE       imgui
        OUTPUT_NAME     imgui_opengl3
    	SOURCE_FILES	${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
    )
    component_add_internal_dependencies(opengl3 core)
    list(APPEND IMGUI_BACKENDS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_opengl3.h)
    list(APPEND IMGUI_BACKENDS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_opengl3_loader.h)
    list(APPEND IMGUI_COMPONENTS opengl3)
endif()

if(IMGUI_GLFW_SUPPORT)
    find_package(glfw3 CONFIG REQUIRED)
    add_component(
        COMPONENT_NAME  glfw3
        STATIC
        NAMESPACE       imgui
        OUTPUT_NAME     imgui_glfw3
    	SOURCE_FILES	${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    )
    target_link_libraries(glfw3 PRIVATE glfw)
    component_add_internal_dependencies(glfw3 core)
    list(APPEND IMGUI_BACKENDS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_glfw.h)
    list(APPEND IMGUI_COMPONENTS glfw3)
endif()

if(IMGUI_VULKAN_SUPPORT)
    find_package(Vulkan REQUIRED)
    add_component(
        COMPONENT_NAME  vulkan
        STATIC
        NAMESPACE       imgui
        OUTPUT_NAME     imgui_vulkan
    	SOURCE_FILES	${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp
    )
    target_link_libraries(vulkan PRIVATE Vulkan::Vulkan)
    component_add_internal_dependencies(vulkan core)
    list(APPEND IMGUI_BACKENDS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_vulkan.h)
    list(APPEND IMGUI_COMPONENTS vulkan)
endif()

if(IMGUI_DIRECTX12_SUPPORT AND WIN32)
    add_component(
        COMPONENT_NAME  directx12
        STATIC
        NAMESPACE       imgui
        OUTPUT_NAME     imgui_directx12
    	SOURCE_FILES	${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_directx12.cpp
    )
    component_add_internal_dependencies(directx12 core)
    list(APPEND IMGUI_BACKENDS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_dx12.h)
    list(APPEND IMGUI_COMPONENTS directx12)
endif()

install_components(
	PROJECT_NAME	imgui
	VERSION			0.0.0
	NAMESPACE		imgui
	COMPONENTS		${IMGUI_COMPONENTS}
)

install(
    FILES       ${IMGUI_HEADERS} ${IMGUI_BACKENDS_HEADERS}
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include
)
