cmake_minimum_required(VERSION 3.22)

project(parallel-rdp LANGUAGES CXX C)

set(OpenGL_GL_PREFERENCE GLVND)

set(NAME_PLUGIN_M64P "mupen64plus-video-parallel")
include_directories(../mupen64plus-core/src/api)
add_definitions(-DM64P_PLUGIN_API)

include(CheckIPOSupported)
check_ipo_supported(RESULT ENABLE_IPO)

if(ENABLE_IPO AND (NOT CMAKE_BUILD_TYPE STREQUAL "Debug"))
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
    message("Interprocedural optimizations enabled")
endif()

find_package(OpenGL REQUIRED)
if(WIN32)
    add_definitions(-DVK_USE_PLATFORM_WIN32_KHR)
endif()

FILE(GLOB RDPSources parallel-rdp/*.cpp)

add_library(${NAME_PLUGIN_M64P} SHARED
    ${RDPSources}
    vulkan/buffer.cpp
    vulkan/buffer_pool.cpp
    vulkan/command_buffer.cpp
    vulkan/command_pool.cpp
    vulkan/context.cpp
    vulkan/cookie.cpp
    vulkan/descriptor_set.cpp
    vulkan/device.cpp
    vulkan/event_manager.cpp
    vulkan/fence.cpp
    vulkan/fence_manager.cpp
    vulkan/image.cpp
    vulkan/memory_allocator.cpp
    vulkan/pipeline_event.cpp
    vulkan/query_pool.cpp
    vulkan/render_pass.cpp
    vulkan/sampler.cpp
    vulkan/semaphore.cpp
    vulkan/semaphore_manager.cpp
    vulkan/shader.cpp
    vulkan/texture_format.cpp
    vulkan/wsi.cpp
    util/arena_allocator.cpp
    util/logging.cpp
    util/thread_id.cpp
    util/aligned_alloc.cpp
    util/timer.cpp
    util/timeline_trace_file.cpp
    util/thread_name.cpp
    volk/volk.c
    gfx_m64p.c
    glguts.c
    parallel_imp.cpp)

target_include_directories(${NAME_PLUGIN_M64P} PUBLIC
    parallel-rdp
    volk
    vulkan
    vulkan-headers/include
    util)

if(WIN32)
    target_link_libraries(${NAME_PLUGIN_M64P} ${OPENGL_LIBRARIES} -lwinmm)
else(WIN32)
    target_link_libraries(${NAME_PLUGIN_M64P} ${OPENGL_LIBRARIES})
endif(WIN32)

set_property(TARGET ${NAME_PLUGIN_M64P} PROPERTY C_VISIBILITY_PRESET hidden)
set_property(TARGET ${NAME_PLUGIN_M64P} PROPERTY CXX_VISIBILITY_PRESET hidden)
set_property(TARGET ${NAME_PLUGIN_M64P} PROPERTY VISIBILITY_INLINES_HIDDEN ON)
if(NOT APPLE)
    target_link_options(${NAME_PLUGIN_M64P} PRIVATE -Wl,-Bsymbolic -Wl,-Bsymbolic-functions)
endif(NOT APPLE)
set_target_properties(${NAME_PLUGIN_M64P} PROPERTIES PREFIX "")
