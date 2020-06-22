message(STATUS "* Configuring DFXVideo")
add_library(DFXVideo MODULE)
add_library(pcsxr::plugins::DFXVideo ALIAS DFXVideo)
target_link_libraries(DFXVideo PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(DFXVideo PUBLIC 
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
    target_compile_definitions(DFXVideo PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgDFXVideo)
target_link_libraries(cfgDFXVideo PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgDFXVideo PUBLIC 
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
    target_compile_definitions(cfgDFXVideo PUBLIC ENABLE_SIO1API)
endif()

# Glib GTK3 X11

#sources
set(PLUGIN_SRCS
    cfg.c
    draw.c
    fps.c
    gpu.c
    key.c
    menu.c
    prim.c
    soft.c
    zn.c
)

set(GUI_SRCS
    gpucfg-0.1df/main.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/gpucfg-0.1df/dfxvideo.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/dfxvideo
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/gpucfg-0.1df
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(dfxvideo_resource DEPENDS ${RESOURCE_FILE})

target_sources(DFXVideo PUBLIC ${PLUGIN_SRCS})
if (SDL_USE_V2)
    target_include_directories(DFXVideo PUBLIC ${SDL2_INCLUDE_DIR})
else()
    target_include_directories(DFXVideo PUBLIC ${SDL_INCLUDE_DIR})
endif()
target_include_directories(DFXVideo PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH} ${X11_Xv_INCLUDE_PATH} ${X11_XShm_INCLUDE_PATH})
target_link_libraries(DFXVideo PUBLIC ${GTK_LIBRARIES} ${X11_LIBRARIES} ${X11_Xv_LIB} ${X11_Xext_LIB} -lm)

target_sources(cfgDFXVideo PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
if (SDL_USE_V2)
    target_include_directories(cfgDFXVideo PRIVATE ${SDL2_INCLUDE_DIR})
else()
    target_include_directories(cfgDFXVideo PRIVATE ${SDL_INCLUDE_DIR})
endif()
add_dependencies(cfgDFXVideo dfxvideo_resource)
target_include_directories(cfgDFXVideo PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH} ${X11_Xv_INCLUDE_PATH} ${X11_XShm_INCLUDE_PATH})
target_link_libraries(cfgDFXVideo PRIVATE ${GTK_LIBRARIES})

install(TARGETS DFXVideo LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgDFXVideo RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)