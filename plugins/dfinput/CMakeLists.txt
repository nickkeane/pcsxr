message(STATUS "* Configuring DFInput")
add_library(DFInput MODULE)
add_library(pcsxr::plugins::DFInput ALIAS DFInput)
target_link_libraries(DFInput PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(DFInput PUBLIC 
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
    target_compile_definitions(DFInput PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgDFInput)
target_link_libraries(cfgDFInput PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgDFInput PUBLIC 
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
    target_compile_definitions(cfgDFInput PUBLIC ENABLE_SIO1API)
endif()

#sources
set(PLUGIN_SRCS
    analog.c
    cfg.c
    pad.c
    sdljoy.c
    util.c
    xkb.c
)

set(GUI_SRCS
    cfg-gtk.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/dfinput.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/dfinput
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(dfinput_resource DEPENDS ${RESOURCE_FILE})

target_sources(DFInput PUBLIC ${PLUGIN_SRCS})
if (SDL_USE_V2)
    target_include_directories(DFInput PUBLIC ${SDL2_INCLUDE_DIR})
else()
    target_include_directories(DFInput PUBLIC ${SDL_INCLUDE_DIR})
endif()
target_include_directories(DFInput PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH})
target_link_libraries(DFInput PUBLIC ${GTK_LIBRARIES} ${SDL_LIBRARY} ${SDL2_LIBRARY} ${X11_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})

target_sources(cfgDFInput PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
if (SDL_USE_V2)
    target_include_directories(cfgDFInput PRIVATE ${SDL2_INCLUDE_DIR})
else()
    target_include_directories(cfgDFInput PRIVATE ${SDL_INCLUDE_DIR})
endif()
add_dependencies(cfgDFInput dfinput_resource)
target_include_directories(cfgDFInput PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH})
target_link_libraries(cfgDFInput PRIVATE ${GTK_LIBRARIES} ${SDL_LIBRARY} ${SDL2_LIBRARY} ${X11_LIBRARIES})

install(TARGETS DFInput LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgDFInput RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
