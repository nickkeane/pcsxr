message(STATUS "* Configuring DFSound")
add_library(DFSound MODULE)
add_library(pcsxr::plugins::DFSound ALIAS DFSound)
target_link_libraries(DFSound PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(DFSound PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
    LOCALE_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/locale/"
    PSEMU_DATA_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/psemu"
    DEF_PLUGIN_DIR="${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu"    
)
if (BUILD_SIO1)
    target_compile_definitions(DFSound PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgDFSound)
target_link_libraries(cfgDFSound PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgDFSound PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
    LOCALE_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/locale/"
    PSEMU_DATA_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/psemu"
    DEF_PLUGIN_DIR="${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu"    
)
if (BUILD_SIO1)
    target_compile_definitions(cfgDFSound PUBLIC ENABLE_SIO1API)
endif()

#sources
set(PLUGIN_SRCS
    adsr.c
    cfg.c
    dma.c
    externals.c
    freeze.c
    registers.c
    reverb.c
    spu.c
    xa.c
)

# Target Selected Sound Backend
if (${SND_BACKEND} STREQUAL "oss")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} oss.c)
    target_include_directories(DFSound PUBLIC ${OSS_INCLUDE_DIR})
    target_include_directories(cfgDFSound PRIVATE ${OSS_INCLUDE_DIR})
    target_compile_definitions(DFSound PUBLIC USEOSS=1)
    target_compile_definitions(cfgDFSound PRIVATE USEOSS=1)
elseif (${SND_BACKEND} STREQUAL "alsa")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} alsa.c)
    target_include_directories(DFSound PUBLIC ${ALSA_INCLUDE_DIRS})
    target_include_directories(cfgDFSound PRIVATE ${ALSA_INCLUDE_DIRS})
    target_compile_definitions(DFSound PUBLIC USEALSA=1)
    target_compile_definitions(cfgDFSound PRIVATE USEALSA=1)    
    target_link_libraries(DFSound ${ALSA_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
elseif (${SND_BACKEND} STREQUAL "pulse")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} pulseaudio.c)
    target_include_directories(DFSound PUBLIC ${PULSEAUDIO_INCLUDE_DIR})
    target_include_directories(cfgDFSound PRIVATE ${PULSEAUDIO_INCLUDE_DIR})
    target_compile_definitions(DFSound PUBLIC USEPULSEAUDIO=1)
    target_compile_definitions(cfgDFSound PRIVATE USEPULSEAUDIO=1)    
    target_link_libraries(DFSound ${PULSEAUDIO_LIBRARY} ${CMAKE_THREAD_LIBS_INIT})
elseif (${SND_BACKEND} STREQUAL "sdl")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} sdl.c)
    if (SDL_USE_V2)
        target_include_directories(DFSound PUBLIC ${SDL2_INCLUDE_DIR})    
        target_include_directories(cfgDFInput PRIVATE ${SDL2_INCLUDE_DIR})
    else()
        target_include_directories(DFSound PUBLIC ${SDL_INCLUDE_DIR})    
        target_include_directories(cfgDFInput PRIVATE ${SDL_INCLUDE_DIR})
    endif()
    target_compile_definitions(DFSound PUBLIC USESDL=1)
    target_compile_definitions(cfgDFSound PRIVATE USESDL=1)
    target_link_libraries(DFSound ${SDL_LIBRARY} ${SDL2_LIBRARY} ${CMAKE_THREAD_LIBS_INIT})
elseif (${SND_BACKEND} STREQUAL "openal")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} openal.c)
    target_include_directories(DFSound PUBLIC ${OPENAL_INCLUDE_DIR})
    target_include_directories(cfgDFSound PRIVATE ${OPENAL_INCLUDE_DIR})
    target_compile_definitions(DFSound PUBLIC USEOPENAL=1)
    target_compile_definitions(cfgDFSound PRIVATE USEOPENAL=1)    
    target_link_libraries(DFSound ${OPENAL_LIBRARY} ${CMAKE_THREAD_LIBS_INIT})
elseif (${SND_BACKEND} STREQUAL "null")
    set(PLUGIN_SRCS ${PLUGIN_SRCS} nullsnd.c)
    target_compile_definitions(DFSound PUBLIC USENULL=1)
    target_compile_definitions(cfgDFSound PRIVATE USENULL=1)
endif()

set(GUI_SRCS
    spucfg-0.1df/main.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/spucfg-0.1df/dfsound.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/dfsound
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/spucfg-0.1df
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(dfsound_resource DEPENDS ${RESOURCE_FILE})

target_sources(DFSound PUBLIC ${PLUGIN_SRCS})
target_include_directories(DFSound PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(DFSound PUBLIC ${GTK_LIBRARIES})

target_sources(cfgDFSound PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
add_dependencies(cfgDFSound dfsound_resource)
target_include_directories(cfgDFSound PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(cfgDFSound PRIVATE ${GTK_LIBRARIES})

install(TARGETS DFSound LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgDFSound RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)